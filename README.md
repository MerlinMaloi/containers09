# Лабораторная работа №9

## Цель работы

Целью работы является знакомство с методами оптимизации образов.

## Задание

Сравнить различные методы оптимизации образов:

- Удаление неиспользуемых зависимостей и временных файлов
- Уменьшение количества слоев
- Минимальный базовый образ
- Перепаковка образа
- Использование всех методов


## Выполнение 

### Начало 

Создал репозиторий containers09 и перенес на свой пк. В папке containers09 создал папку site и поместил в нее файлы сайта.

Для оптимизации используется образ определенный следующим `Dockerfile.raw`:

```
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ с именем `mynginx:raw` с помощью этой команды:

`docker image build -t mynginx:raw -f Dockerfile.raw .`

### Удаление неиспользуемых зависимостей и временных файлов

Удалил временные файлы и неиспользуемые зависимости в `Dockerfile.clean`:

```
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# remove apt cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ с именем mynginx:clean и проверил его размер командами:

```
docker image build -t mynginx:clean -f Dockerfile.clean .
docker image list
```

### Уменьшение количества слоев 

Создал следующий файл Dockerfile.few чтобы уменьшить количество слоев:

```
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ с именем mynginx:few и проверил его размер:

```
docker image build -t mynginx:few -f Dockerfile.few .
docker image list
```

Размер и того и другого raw и clean составил 270 МБ

### Создание минимального базового образа

Заменил базовый образ на alpine и пересобрал образ:

```
# create from alpine image
FROM alpine:latest

# update system
RUN apk update && apk upgrade

# install nginx
RUN apk add nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Собрал образ с именем mynginx:alpine и проверил его размер с помощью команд:

```
docker image build -t mynginx:alpine -f Dockerfile.alpine .
docker image list
```

Размер составил 19.5 МБ

### Перепаковка образа

Я не уверен в том , что поступил исключительно правильно , так как я не использовал следующие команды: 

```
docker container create --name mynginx mynginx:raw
docker container export mynginx | docker image import - mynginx:repack
docker container rm mynginx
docker image list
```

Я сделал иначе так как у меня всплывала ошибка `Error response from daemon: failed to unpack image: failed to extract layer `

Я использовал следующее :

```
docker container create --name mynginx mynginx:raw
docker start mynginx
docker commit mynginx mynginx:repack
docker stop mynginx
docker container rm mynginx
```

Размер также составил 270 МБ

### Использование всех методов

Сделал образ mynginx:min с использованием всех методов:

```
# create from alpine image
FROM alpine:latest

# update system, install nginx and clean
RUN apk update && apk upgrade && \
    apk add nginx && \
    rm -rf /var/cache/apk/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Соберал образ с именем mynginx:minx и узнал размер. Перепаковал как и в предущем образе mynginx:minx в mynginx:min:

```
docker container create --name mynginx mynginx:minx
docker start mynginx
docker commit mynginx mynginx:min
docker stop mynginx
docker container rm mynginx
```

### Запуск и тестирование

```
docker image list                                            
REPOSITORY            TAG           IMAGE ID       CREATED             SIZE  
mynginx               min           56eb5844d367   26 minutes ago      14.5MB
myngin                min           04712c3e5267   30 minutes ago      2.99MB
mynginx               minx          c9fc85484336   30 minutes ago      14.4MB
mynginx               repack        7d6c66987595   33 minutes ago      270MB 
<none>                <none>        9513caf09a2a   37 minutes ago      54.5MB
mynginx               alpine        c9f4879bf206   About an hour ago   19.5MB
mynginx               few           67b5b93302e0   2 hours ago         186MB 
mynginx               clean         e72077370070   2 hours ago         270MB 
mynginx               raw           acd478069dfd   3 hours ago         270MB 
```


### Ответы на вопросы 

1. Какой метод оптимизации образов вы считаете наиболее эффективным?

- Я возможно ошибочно считаю наиболее эффективным использование всех методов из-за наименьшего размера по памяти от 2.99 до 14.5(вероятнее всего 2.99 ошибка)

2. Почему очистка кэша пакетов в отдельном слое не уменьшает размер образа?

- Из-за многослойности системы , очистка не проходит вглубь слоев 

3. Что такое перепаковка образа?

- Перепаковка образа - это процесс пересоздания образа из уже запущеного