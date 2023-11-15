# Домашнее задание к занятию 4. «Оркестрация группой Docker-контейнеров на примере Docker Compose»


## Задача 1

#### Создайте собственный образ любой операционной системы (например, debian-11) с помощью Packer версии 1.7.0.

1. Устанавливаем дистрибутив Packer из [зеркала](https://hashicorp-releases.yandexcloud.net/packer/) согласно [инструкции](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/packer-quickstart):
```
wget https://hashicorp-releases.yandexcloud.net/packer/1.9.4/packer_1.9.4_linux_amd64.zip
unzip packer_1.9.4_linux_amd64.zip
rm packer_1.9.4_linux_amd64.zip
```

2. Добавляем путь к папке, в которой находится исполняемый файл, в переменную:
```
export PATH=$PATH:/root/virtualisation
```

3.  Создаем конфигурационный файл config.pkr.hcl, где прописываем провайдера Yandex Cloud:

```
nano config.pkr.hcl

packer {
  required_plugins {
    yandex = {
      version = ">= 1.1.2"
      source  = "github.com/hashicorp/yandex"
    }
  }
}
```

4. Устанавливаем плагин:
```
packer init /root/virtualisation/config.pkr.hcl
```

5. Получаем информацию, необходимую для подготовки конфигурации образа:
```
yc config list
yc vpc subnet list
```

6. Создаем конфигурацию образа операционной системы Debian 11 прямо из [инструкции](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/packer-quickstart), только удалим из примера установку Nginx, так как данное программное обеспечение нам не потребуется для выполнения данного задания (забегая вперед можно дополнить конфигурационный файл установкой Prometheus и Grafana, но по условиям задания данные действия будем выполнять другими инструментами):





## Задача 2

**2.1.** Создайте вашу первую виртуальную машину в YandexCloud с помощью web-интерфейса YandexCloud.        

**2.2.*** **(Необязательное задание)**      
Создайте вашу первую виртуальную машину в YandexCloud с помощью Terraform (вместо использования веб-интерфейса YandexCloud).
Используйте Terraform-код в директории ([src/terraform](https://github.com/netology-group/virt-homeworks/tree/virt-11/05-virt-04-docker-compose/src/terraform)).

Чтобы получить зачёт, вам нужно предоставить вывод команды terraform apply и страницы свойств, созданной ВМ из личного кабинета YandexCloud.

## Задача 3

С помощью Ansible и Docker Compose разверните на виртуальной машине из предыдущего задания систему мониторинга на основе Prometheus/Grafana.
Используйте Ansible-код в директории ([src/ansible](https://github.com/netology-group/virt-homeworks/tree/virt-11/05-virt-04-docker-compose/src/ansible)).

Чтобы получить зачёт, вам нужно предоставить вывод команды "docker ps" , все контейнеры, описанные в [docker-compose](https://github.com/netology-group/virt-homeworks/blob/virt-11/05-virt-04-docker-compose/src/ansible/stack/docker-compose.yaml),  должны быть в статусе "Up".

## Задача 4

1. Откройте веб-браузер, зайдите на страницу http://<внешний_ip_адрес_вашей_ВМ>:3000.
2. Используйте для авторизации логин и пароль из [.env-file](https://github.com/netology-group/virt-homeworks/blob/virt-11/05-virt-04-docker-compose/src/ansible/stack/.env).
3. Изучите доступный интерфейс, найдите в интерфейсе автоматически созданные docker-compose-панели с графиками([dashboards](https://grafana.com/docs/grafana/latest/dashboards/use-dashboards/)).
4. Подождите 5-10 минут, чтобы система мониторинга успела накопить данные.

Чтобы получить зачёт, предоставьте: 

- скриншот работающего веб-интерфейса Grafana с текущими метриками, как на примере ниже.
<p align="center">
  <img width="1200" height="600" src="./assets/yc_02.png">
</p>

## Задача 5 (*)

Создайте вторую ВМ и подключите её к мониторингу, развёрнутому на первом сервере.

Чтобы получить зачёт, предоставьте:

- скриншот из Grafana, на котором будут отображаться метрики добавленного вами сервера.


