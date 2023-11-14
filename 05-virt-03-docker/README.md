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

Посмотрите на сценарий ниже и ответьте на вопрос:
«Подходит ли в этом сценарии использование Docker-контейнеров или лучше подойдёт виртуальная машина, физическая машина? Может быть, возможны разные варианты?»

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

- высоконагруженное монолитное Java веб-приложение;
- Nodejs веб-приложение;
- мобильное приложение c версиями для Android и iOS;
- шина данных на базе Apache Kafka;
- Elasticsearch-кластер для реализации логирования продуктивного веб-приложения — три ноды elasticsearch, два logstash и две ноды kibana;
- мониторинг-стек на базе Prometheus и Grafana;
- MongoDB как основное хранилище данных для Java-приложения;
- Gitlab-сервер для реализации CI/CD-процессов и приватный (закрытый) Docker Registry.

## Задача 3

Скачаем контейнер из образа ***centos*** c тегом 7
```
docker pull centos:7
```

Создаем папку data на хостовой машине
```
mkdir data
```

Запускаем контейнер Сentos 7 в фоновом режиме и  подключаем папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера
```
docker run -it -v /root/virtualisation/data:/data -d centos:7 bash
```

Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера.
```
docker pull debian:11
docker run -it -v /root/virtualisation/data:/data -d debian:11 bash
```

Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```
```
docker exec -it 198f9c18d47b bash
cd data
yum install nano
nano file1
Hello everybody, I'm here!
exit
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker6.png)

Добавьте ещё один файл в папку ```/data``` на хостовой машине
```
cd data
nano file2
I see this file on my virtual host
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker7.png)

Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.
```
docker exec -it 8ab95a9d6e30 bash
cd data
ls data
cat file1
cat file2
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker8.png)

## Задача 4 (*) Воспроизведите практическую часть лекции самостоятельно. Соберите Docker-образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.

Создаем папку для выполнения задания
```
mkdir ex4
cd ex4
---

Создаем dockerfile с кодом, аналогичным рассмотренному в лекции

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

Запускаем сборку и отправку в [репозиторий](https://hub.docker.com/u/leonid1984) на docker hub нашего [контейнера](https://hub.docker.com/repository/docker/leonid1984/ansible/general) 

```
docker build -t leonid1984/ansible:2.10.0 .
docker push leonid1984/ansible:2.10.0
```
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker9.png)

Проверяем работоспособность контейнера
![Alt text](https://github.com/LeonidKhoroshev/virtd-homeworks/blob/main/05-virt-03-docker/docker/docker10.png)

