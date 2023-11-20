# Домашнее задание к занятию 5. «Оркестрация кластером Docker контейнеров на примере Docker Swarm» - Леонид Хорошев
---

## Задача 1

Дайте письменые ответы на вопросы:

- В чём отличие режимов работы сервисов в Docker Swarm-кластере: replication и global?
- Какой алгоритм выбора лидера используется в Docker Swarm-кластере?
- Что такое Overlay Network?

## Задача 2

#### Создайте ваш первый Docker Swarm-кластер в Яндекс Облаке.

1. Развернем в Яндекс облаке 6 виртуальных машин (Centos7) по 3 управляющих (manager) и управляемых (worker). Для оптимизации процесса воспользуемся terraform:
```
nano main.tf

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  token = "y0_AgAA--------------0K1VqSs"
  cloud_id  = "b1g3ks25rm2qagep03qb"
  folder_id = "b1gadttfn3t0cohh2hk2"
}

resource "yandex_compute_instance" "manager" {
  count                     = 3
  name                      = format("manager-%02d", count.index + 1)
  zone                      = "ru-central1-b"
  hostname                  = format("manager-%02d", count.index + 1)
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 2
    core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id    = "fd8vm24pae6k98274k7o"
      type        = "network-ssd"
      size        = "15"
    }
  }

  network_interface {
    subnet_id = "e2laj8khjjcf3lfs0l3p"
    nat       = true
  }

  metadata = {
    user-data = "${file("/root/terraform/meta.yml")}"
  }
  scheduling_policy {
    preemptible = true

resource "yandex_compute_instance" "worker" {
  count                     = 3
  name                      = format("worker-%02d", count.index + 1)
  zone                      = "ru-central1-b"
  hostname                  = format("worker-%02d", count.index + 1)
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 2
    core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id    = "fd8vm24pae6k98274k7o"
      type        = "network-ssd"
      size        = "15"
    }
  }

  network_interface {
    subnet_id = "e2laj8khjjcf3lfs0l3p"
    nat       = true
  }

  metadata = {
    user-data = "${file("/root/terraform/meta.yml")}"
  }
  scheduling_policy {
    preemptible = true
  }
}
```

```
terraform apply
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-05-docker-swarm/swarm/Swarm1.png)

2. C помощью Ansible установим требуемое программное обеспечение (Docker):

Создадим Inventory файл
```
nano /etc/ansible/inventory

[nodes:children]
manager
worker

[manager]
158.160.18.199 ansible_user=leo
158.160.65.20  ansible_user=leo
158.160.79.173 ansible_user=leo

[worker]
158.160.69.55 ansible_user=leo
158.160.75.3  ansible_user=leo
84.201.136.153 ansible_user=leo
```

Проверяем:
```
ansible all --list-host
  hosts (6):
    158.160.18.199
    158.160.65.20
    158.160.79.173
    158.160.69.55
    158.160.75.3
    84.201.136.153
```
Проверяем соединение:
```
ansible all -m ping -u leo
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-05-docker-swarm/swarm/Swarm2.png)

Неудача, причина - отсутствие phyton на управляемых хостах. Воспользуемся модулем [Ansible raw](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/raw_module.html) для устранения данной проблемы.
Дополняем Inventory файл строками следующего содержания:

```
[children:vars]
ansible_python=/usr/bin/python
```
Далее пишем короткий плейбук для установки phyton на наши виртуальные хосты:
```
nano playbook.yml

- hosts: nodes
  gather_facts: false
  become: true
  become_user: root
  remote_user: leo
  tasks:
  - name: install python
    raw: test -e /usr/bin/python || ( yum update && yum install python -y )
```
Устанавливаем python:

```
ansible-playbook playbook.yml
```

![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-05-docker-swarm/swarm/Swarm3.png)

Проверяем:
```
ansible all -m ping -u leo
```

![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-05-docker-swarm/swarm/Swarm4.png)

Готовим playbook для установки docker на всех хостах:
```
nano playbook.yml

- hosts: nodes
  become: yes
  become_user: root
  remote_user: leo

  tasks:
    - name: Create directory for ssh-keys
      file: state=directory mode=0700 dest=/root/.ssh/

    - name: Adding rsa-key in /root/.ssh/authorized_keys
      copy: src=~/.ssh/id_rsa.pub dest=/root/.ssh/authorized_keys owner=root mode=0600
      ignore_errors: yes

    - name: Installing tools
      yum:
        name={{ item }}
        update_cache=yes
        state=present
      with_items:
        - git
        - curl
        - ca-certificates
        - gnupg

    - name: Add docker repository
      command: yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

    - name: Installing docker package
      yum:
        name={{ item }}
        state=present
        update_cache=yes
      with_items:
        - docker-ce-cli
        - containerd.io
        - docker-ce
        - docker-buildx-plugin
        - docker-compose-plugin

    - name: Enable docker daemon
      systemd:
        name: docker
        state: started
        enabled: yes
```
Запускаем процесс установки:

```
ansible-playbook provision.yml
```

![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-05-docker-swarm/swarm/Swarm5.png)

3. Переходим на ноду master-01 и инициализируем docker swarm кластер:

```
ssh leo@158.160.18.199
docker swarm init --advertise-addr 10.129.0.23
```

![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-05-docker-swarm/swarm/Swarm6.png)

4. Получаем команды для подключения нод:


Manager:
```
docker swarm join-token -q manager
SWMTKN-1-1l8ur1uj54p63oc0gelp7zut99kbdn9p8vssxoasqhprbzwqbw-b2ic09xpwq5xhyp8um6ml3ex2
```

Worker:
```
docker swarm join-token -q worker
SWMTKN-1-1l8ur1uj54p63oc0gelp7zut99kbdn9p8vssxoasqhprbzwqbw-2uu082ss5rx14i7pugmo0vq72
```


5. Соединяем ноды в кластер:

manager-02
```
ssh leo@158.160.65.20
sudo su
docker swarm join --token SWMTKN-1-1l8ur1uj54p63oc0gelp7zut99kbdn9p8vssxoasqhprbzwqbw-b2ic09xpwq5xhyp8um6ml3ex2 10.129.0.23
```
manager-03 - аналогичная последовательность (меняется только ip адрес при подключении по протоколу ssh).

worker-01:
```
ssh leo@158.160.69.55
sudo su
docker swarm join --token SWMTKN-1-1l8ur1uj54p63oc0gelp7zut99kbdn9p8vssxoasqhprbzwqbw-2uu082ss5rx14i7pugmo0vq72 10.129.0.23
exit
```
worker-02 и worker-03 - аналогичная последовательность (меняется только ip адрес при подключении по протоколу ssh).

Получившийся кластер:
```
docker node ls
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-05-docker-swarm/swarm/Swarm7.png)


## Задача 3

Создайте ваш первый, готовый к боевой эксплуатации кластер мониторинга, состоящий из стека микросервисов.

Чтобы получить зачёт, предоставьте скриншот из терминала (консоли), с выводом команды:
```
docker service ls
```


## Задача 4 (*)

Выполните на лидере Docker Swarm-кластера команду, указанную ниже, и дайте письменное описание её функционала — что она делает и зачем нужна:
```
# см.документацию: https://docs.docker.com/engine/swarm/swarm_manager_locking/
docker swarm update --autolock=true
```


