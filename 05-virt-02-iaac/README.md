
# Домашнее задание к занятию 2. «Применение принципов IaaC в работе с виртуальными машинами» - Леонид Хорошев

---

## Задача 1

#### Опишите основные преимущества применения на практике IaaC-паттернов:
- скорость и возможности масштабирования, IaaC позволяет конфигурировать инфраструктуру одинаково быстро вне зависимости от того, сколько виртуальных машин необходимо развернуть или на скольких машинах необходимо установить и настроить программное обеспечение, так как все операции выполняются в соответствии с четкими инструкциями, прописанными в конфигурационных файлах .tf (Terraform) или .yml файлах (Ansible), однажды написанный скрипт может быстро разворачивать инфраструктуру и вносить изменения в ужк существующие системы;
- стандартизация, вытекает из первого буллита, так как в конфигурационных файлах/скриптах можно прописать требуемое состояние среды, тем самым при развертывании инфраструктуры обеспечить корректную настройку программного обеспечения и исключить "нестыковки" конфигурации, отсутствие необходимых зависимостей и так далее;
- восстановление в аварийных ситуациях происходит также "из кода", что позволяет существенно сократить простой сервисов;
- сокращение затрат на обслуживание инфраструктуры - преимущество вытекает из вышеперечисленных буллитов за счет раста скорости развертывания ресурсов и снижения простоя сервисов.

#### Какой из принципов IaaC является основополагающим?

Основопологающий принцип IaaC - идемпотентность (свойство объекта или операции при повторном применении операции к объекту давать тот же результат, что и при первом). В нашем случае это значит, что один и тотже конфигурационный файл (например .tf, развертывающий 5 виртуальных машин в Яндекс облаке под управлением OC Debian11) каждый раз при его запуске командoй ` terraform apply` будет один и тот же результат, будут подняты все те же 5 виртуальных машин с Debian11. 

## Задача 2

#### Чем Ansible выгодно отличается от других систем управление конфигурациями?

Для управления узлами Ansible использует протокол SSH, это значит, что для подключения к виртуальной машине, на ней не должны быть устоновлены специальные клиенты, необходима только работающая служба sshd и библиотеки phyton (которые присутствуют в большинстве дистрибутивов Linux "из коробки"). В случае если библиотеки phнton на клиентских ВМ отсутсвуют, проблема решается через  модуль raw:
```
---
- name: raw module usage
  hosts: all
  become: yes
  tasks:
  - name: Bootstrap a host without python3 installed
    raw: dnf install -y python3 python3-dnf libselinux-python
```

Из этого следует второе преимущество - у Ansible есть много полезных [модулей](https://habr.com/ru/companies/slurm/articles/707130/), они позволяют проверять соединение (ping), устанавливать пакеты (apt, yum) и так далее.

Также для Ansible есть портал [Ansible Galaxy](https://galaxy.ansible.com/ui/) с готовыми ролями для составления плейбуков. 

Последнее "субъективное" достоинство Ansible - легкость освоения формата .yml.

#### Какой, на ваш взгляд, метод работы систем конфигурации более надёжный — push или pull?

Ansible-push - режим, в котором вручную или с помощью скрипта осуществляется накатывание изменений на сервер (используется в Ansible по-умолчанию). При таком режиме последовательность прописанных в плейбуке действий будет выполняться всегда одинаково, независимо от конфигурации настраиваемой ВМ (например, если в плейбуке фигурирует установка nginx `apt install -y nginx`, то данная команда запустится в независимости от того присутствует уже у нас установленный nginx или нет).

Ansible-pull - режим, при котором текущая конфигурация сначала сравнивается с целевой, и только потом, при наличии изменений, осуществляются действия, прописапнные в сценарии (на нашем примере, если nginx уже установлен, то попытки повторной его установки не будет).

На мой взгляд метод push является надежнее, так как необходимая конфигурация формируется четко заданным в .yml файле набором команд и риск ошибки получить некорректную конфигурацию практически отсутствует, но более рациональный с точки зрения ресурсов является метод pull, так как он исключает использование вычислительных ресурсов для выполнения "лишних" операций.

Главный риск при методе pull - гипотетическое нарушение каких-нибудь зависимостей или версий пакетов (риск минимален, но все же присутствует).

## Задача 3 Установите на личный linux-компьютер(или учебную ВМ с linux):

Это и последующие задания выполнены на виртуальной машине под управлением Centos7, развернутой в VirtulaBox, хостовая ОС - Windows10.

#### [VirtualBox](https://www.virtualbox.org/)

1. Создание директории для выполнения домашнего задания:

```
mkdir virtualisation
cd virtualisation
```

2. Установка Virtualbox:

```
wget https://download.virtualbox.org/virtualbox/7.0.12/VirtualBox-7.0-7.0.12_159484_el7-1.x86_64.rpm
yum localinstall --nogpgcheck VirtualBox-7.0-7.0.12_159484_el7-1.x86_64.rpm
```

3. Проверка:

```
virtualbox --version
```

4. Результат (неудачный)
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt1.png)

5. Траблшутинг:
 - модуль ядра
```
yum install -y "kernel-devel-uname-r == $(uname -r)"
yum install -y gcc perl make
sudo /sbin/rcvboxdrv setup
```
- несуществующий путь переменной XDG_RUNTIME_DIR 
```
export XDG_RUNTIME_DIR=/root/virtualisation
chmod 777 /root/virtualisation
```
- проблемы с экраном
```
QT_QPA_PLATFORM=offscreen /usr/lib/virtualbox/vboxdrv.sh
export QT_QPA_PLATFORM=offscreen
systemctl restart vboxdrv
```
- включение GUI
```
startx
```

6. Повторная проверка:
```
virtualbox --version

```
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt2.png)
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt3.png)



#### [Vagrant](https://github.com/netology-code/devops-materials), рекомендуем версию 2.3.4(в более старших версиях могут возникать проблемы интеграции с ansible)

1. Установка:
```
yum install -y https://hashicorp-releases.yandexcloud.net/vagrant/2.3.4/vagrant-2.3.4-1.x86_64.rpm
```

2. Проверка:

```
vagrant --version
Vagrant 2.3.4
```

3. Настройка работы Vagrant с Virtualbox:

```
export VAGRANT_DEFAULT_PROVIDER=virtualbox
```

#### [Terraform](https://github.com/netology-code/devops-materials/blob/master/README.md)  версии 1.5.Х (1.6.х может вызывать проблемы с яндекс-облаком):

1. Установка:
```
wget https://hashicorp-releases.yandexcloud.net/terraform/1.5.7/terraform_1.5.7_linux_amd64.zip
unzip terraform_1.5.7_linux_amd64.zip
mv terraform /usr/local/bin/
```

2. Проверка:
```
terraform --version
Terraform v1.5.7
on linux_amd64
```


#### Ansible

Ansible установлен в рамках выполнения предыдущих домашних работ
```
ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, Jun 20 2023, 11:36:40) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
```

*Приложите вывод команд установленных версий каждой из программ, оформленный в Markdown.*

## Задача 4 

Воспроизведите практическую часть лекции самостоятельно.
1. Проверяем возможность виртуализации:
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt4.png)

По умолчанию такая возможность недоступна, следовательно необходимо включить Nested VT-x, для чего переходим в командную строку хостовой ОС:
```
cd C:\Program Files\Oracle\VirtualBox
VBoxManage.exe list vms
"Ansible" {454c55b4-a66e-48ee-8ea7-3cdf8d9b07c6}
"test_for_Ansible1" {bf9abaca-e2a1-40c5-ad3c-7c5ef9539a2c}
"test_for_Ansible2" {f55e1805-d917-4d30-bdd8-9fe7145ad2cb}
"metasploitable" {2b4b56f8-0f91-489d-8a39-040070fab6b8}
"kali-linux-2023.2-virtualbox-amd64" {f43a94b9-4908-4cf7-bab9-4584b7c5473c}
VBoxManage.exe modifyvm "454c55b4-a66e-48ee-8ea7-3cdf8d9b07c6" --nested-hw-virt on
```
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt5.png)

2. Переходим в Centos7 и через vagrant разворачимаем виртуальную машину через файл-образ "bento/ubuntu-20.04":
- скачиваем файл-образ "bento/ubuntu-20.04"
```
wget https://app.vagrantup.com/bento/boxes/ubuntu-20.04/versions/202309.09.0/providers/virtualbox/unknown/vagrant.box
```
- добавляем его в в список образов Vagrant
```
vagrant box add bento/ubuntu-20.04 ./vagrant.box
vagrant box list
bento/ubuntu-20.04 (virtualbox, 0)
```
- скачиваем vagrantfile, рассмотренный в лекции
```
wget https://github.com/netology-code/virt-video-code/blob/main/vagrant/Vagrantfile
```
- поднимаем виртуальную машину
```
vagrant up
```
- проверяем
```
vagrant ssh
```
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt6.png)
- также проверяем "для наглядности" в GUI Linux
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt7.png)

- проверяем, установился ли docker
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt8.png)
Docker не установлен, вероятная причина - команда `vagrant up` отработала не до конца из-за таймаута, так как ввиду скромных возможностей домашнего железа, развертка виртуальной машины Ubuntu в Virtualbox, установленный в Centos7, развернутой в свою очередь в Virtualbox уже в хостовой Windows 10 происходит крайне медленно и не с первого раза.

- настраиваем виртуальную машину:
```
vagrant provision
```

Примечание: основная проблема при выполнении домашнего задания связана со "сверхусилиями" процессора при запуске ansible playbook
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt9.png)
Virtualbox потребляет все доступные ресурсы ЦПУ.

Внутри виртуальной машины ситуация еще хуже
![alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-02-iaac/virt/virt10.png)
