![2023-11-13_17-16-38](https://github.com/LeonidKhoroshev/virtd-homeworks/assets/114744186/6b961108-b119-43c5-b66d-c5493a315b2a)
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
docker run  -d custom-nginx
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

- Запустите первый контейнер из образа ***centos*** c любым тегом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера.
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера.
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```.
- Добавьте ещё один файл в папку ```/data``` на хостовой машине.
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.

## Задача 4 (*)

Воспроизведите практическую часть лекции самостоятельно.

Соберите Docker-образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.


---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

