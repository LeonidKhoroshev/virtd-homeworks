## Домашнее задание к занятию 3. «Введение. Экосистема. Архитектура. Жизненный цикл Docker-контейнера» - Леонид Хорошев


## Задача 1

#### Cоздайте свой репозиторий на https://hub.docker.com

Репозиторий создан в рамках выполнения предыдущих домашних работ
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker1.png)

#### Выбрираем самый скачиваемый [образ Nginx](https://hub.docker.com/_/nginx)
```
docker pull nginx
```
#### Cоздаем свой fork образа с требуемой в задании функциональностью

Создаем dockerfile на основе скачанного образа
```
nano dockerfile

FROM nginx
LABEL maintainer = khoroshevlv@gmail.com
COPY index.html /var/www/html/index.html
COPY index.html /usr/share/nginx/html/index.html
```

Создаем файл index.html, отвечающий условиям задания
```
nano index.html

<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```

Собираем контейнер
```
docker build -t custom-nginx
```

Проверяем, что он работает
```
docker run -p 80:80  -d custom-nginx
docker ps -a
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker2.png)

Проверяем отображение страницы через браузер
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker4.png)
Выглядит странно, проверяем корректность данных непосредственно внутри контейнера
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker5.png)
Внутри запущенного контейнера содержание страницы index.html отображается верно, видимо что-то со шрифтами.

#### Публикуем созданный fork [custom-nginx](https://hub.docker.com/r/leonid1984/custom-nginx) в своём репозитории
```
docker push leonid1984/custom-nginx
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker3.png)

## Задача 2

#### Посмотрите на сценарий ниже и ответьте на вопрос:
«Подходит ли в этом сценарии использование Docker-контейнеров или лучше подойдёт виртуальная машина, физическая машина? Может быть, возможны разные варианты?»

1. Высоконагруженное монолитное Java веб-приложение - одинаково продуктивно будет разместить приложение как на вирьтуальной машине так и на физическом сервере. В целом зависит от его "нагруженности", теоретически возможен и контейнер, но скорее всего такое приложение будет взаимодействовать с базами данных, веб интерфейсом, условными спутниками (в случае, если для него нужна геопозиция пользователя) и т.д. Можно упаковать все это в контейнеры и оркестрировать условно Kubernetes или Docker Swarm, но оптимальнее по ресурсам - развернуть приложение на виртуальной машине либо физическом сервере.

2. Nodejs веб-приложение - оптимальный вариант - контейнер, так как в целом ява-приложения не сильно требовательны к ресурсам, а контейнерная среда позволит их быстро перезапускать в случае нештатной ситуации.
   
3. Мобильное приложение c версиями для Android и iOS - зависит от масштабов. Для высоконагруженных популярных приложений лучше всего подойдет виртуальная машина, она позволит обеспечить требуемую отказоустойчивось при большом количестве подключений (отказоустойчивый кластер), масштабирование, шардирование (как вертикальное, так и горизонтальное). В случае с более "нишевыми продуктами" подойдут контейнеры под управлением Docker swarm либо Kubernetes. 

4. Шина данных на базе Apache Kafka - решение используется в основном в микросервисной архитектуре и, скорее всего, микросервисы развернуты в контейнерах. Исходя из этого - целесообразней будет развернуть шину данных в [контейнере](https://hub.docker.com/r/bitnami/kafka/) и оркестрировать наши микросервисы через одну систему (упомянутые ранее Kubernetes или Docker Swarm).
   
5. Elasticsearch-кластер для реализации логирования продуктивного веб-приложения — три ноды elasticsearch, два logstash и две ноды kibana - отказоустойчивый кластер виртуальных машин. Стек ELK достаточно требователен к ресурсам (оперативной памяти), поэтому при наличии большого числа индексов и необходимости анализа разных по составу и количеству логов от разных приложений - идеальным вариантом будет кластер (технически возможна организация из физических машин, но экономическая целесообразность такого решения - нулевая). Также данную архитектуру можно развернуть и в контейнерах (под каждый сервис есть официальные [docker-образы](https://www.elastic.co/blog/getting-started-with-the-elastic-stack-and-docker-compose)), но такая схема будет менее надежна, ведь вероятность того, что контейнер упадет и пропустит выжные данные от системы безопасности (Suricata например) выше, нежели отказ виртуальной машины.
   
6. Мониторинг-стек на базе Prometheus и Grafana - целесообразнее всего [контейнерная среда](https://grafana.com/docs/grafana-cloud/send-data/metrics/metrics-prometheus/prometheus-config-examples/docker-compose-linux/), так как данные технологии не требовательны к ресурсам, но должны иметь возможность быстрого масштабирования;
   
7. MongoDB как основное хранилище данных для Java-приложения - [конейнерная среда](https://habr.com/ru/companies/flant/articles/549040/). Она обеспечивает  масштабилование и гибкость при нестабильных нагрузках (хотя и сложнее в настройках).

## Задача 3

1. Скачаем контейнер из образа ***centos*** c тегом 7
```
docker pull centos:7
```

2. Создаем папку data на хостовой машине
```
mkdir data
```

3. Запускаем контейнер Сentos 7 в фоновом режиме и  подключаем папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера
```
docker run -it -v /root/virtualisation/data:/data -d centos:7 bash
```

4. Запускаем второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера.
```
docker pull debian:11
docker run -it -v /root/virtualisation/data:/data -d debian:11 bash
```

5. Подключаем к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```
```
docker exec -it 198f9c18d47b bash
cd data
yum install nano
nano file1
Hello everybody, I'm here!
exit
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker6.png)

6. Добавляем ещё один файл в папку ```/data``` на хостовой машине
```
cd data
nano file2
I see this file on my virtual host
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker7.png)

7. Подключаемся во второй контейнер и отображаем листинг и содержание файлов в ```/data``` контейнера.
```
docker exec -it 8ab95a9d6e30 bash
cd data
ls data
cat file1
cat file2
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker8.png)

## Задача 4 (*)
#### Воспроизведите практическую часть лекции самостоятельно.

#### Cоберите Docker-образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.

1. Создаем папку для выполнения задания

```
mkdir ex4
cd ex4
```

2. Создаем dockerfile с кодом, аналогичным рассмотренному в лекции

```
nano Dockerfile
FROM alpine:3.14

RUN CARGO_NET_GIT_FETCH_WITH_CLI=1 && \
    apk --no-cache add \
        sudo \
        python3\
        py3-pip \
        openssl \
        ca-certificates \
        sshpass \
        openssh-client \
        rsync \
        git && \
    apk --no-cache add --virtual build-dependencies \
        python3-dev \
        libffi-dev \
        musl-dev \
        gcc \
        cargo \
        openssl-dev \
        libressl-dev \
        build-base && \
    pip install --upgrade pip wheel && \
    pip install --upgrade cryptography cffi && \
    pip uninstall ansible-base && \
    pip install ansible-core && \
    pip install ansible==2.10.0 && \
    pip install --ignore-installed mitogen ansible-lint jmespath && \
    pip install --upgrade pywinrm && \
    apk del build-dependencies && \
    rm -rf /var/cache/apk/* && \
    rm -rf /root/.cache/pip && \
    rm -rf /root/.cargo

RUN mkdir /ansible && \
    mkdir -p /etc/ansible && \
    echo 'localhost' > /etc/ansible/hosts

WORKDIR /ansible

CMD [ "ansible-playbook", "--version" ]
```

3. Запускаем сборку и отправку в [репозиторий](https://hub.docker.com/u/leonid1984) на docker hub нашего [контейнера](https://hub.docker.com/repository/docker/leonid1984/ansible/general) 

```
docker build -t leonid1984/ansible:2.10.0 .
docker push leonid1984/ansible:2.10.0
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker9.png)

4. Проверяем работоспособность контейнера
   
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker10.png)

