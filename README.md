# Лабораторная работа №7: Создание многоконтейнерного приложения

## Выполнил

* Mihailov Piotr I2302
* Дата выполнения: 13.04.25

## Цель работы

Ознакомиться с работой многоконтейнерного приложения на базе `docker-compose`

## Задание

Создать php приложение на базе трех контейнеров: `nginx`, `php-fpm`, `mariadb`, используя `docker-compose`.

## Подготовка

Для выполнения лабораторной работы необходимо:

* Установленный Docker
* Опыт выполнения лабораторной работы №5

## Выполнение

### Шаг 1: Клонирование и структура проекта

1. Создаю репозиторий `containers07`.
2. Клонирую репозиторий на локальный компьютер с помощью команды `git clone`
3. В директорию `containers07` создаю директорию `mounts/site` и в данную директорию переписываю сайт на php.

### Шаг 2: Настройка `.gitignore`

Создаю файл `.gitignore` в корне проекта со следующим содержимым:

```gitignore
# Ignore files and directories
mounts/site/*
```

### Шаг 3: Конфигурация Nginx

Создаю файл `nginx/default.conf` со следующим содержимым:

```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

* `default.conf` — конфигурация для nginx. В этом файле указано, как обрабатывать запросы: где находится сайт, как запускать PHP-скрипты через php-fpm и что делать, если путь не найден.

### Шаг 4: Docker-compose

Создаю в директории `containers07` файл `docker-compose.yml` со следующим содержимым

```yml
version: '3.9'

services:
  frontend:
    image: nginx:1.19
    volumes:
      - ./mounts/site:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - internal
  backend:
    image: php:7.4-fpm
    volumes:
      - ./mounts/site:/var/www/html
    networks:
      - internal
    env_file:
      - mysql.env
  database:
    image: mysql:8.0
    env_file:
      - mysql.env
    networks:
      - internal
    volumes:
      - db_data:/var/lib/mysql

networks:
  internal: {}

volumes:
  db_data: {}
  ```

* `docker-compose.yml` — главный файл, который описывает, какие контейнеры нужно запустить (nginx, php-fpm, mysql), как они связаны между собой, какие порты и тома использовать. Управляет всей системой сразу.

### Шаг 5 : Конфигурация SQL

Создаю файл `mysql.env` в корне проекта и добавил в него следующие строки:

```mysql
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
```

* `mysql.env` — файл с переменными окружения для MySQL: пароль root-пользователя, имя базы данных, имя и пароль обычного пользователя. Эти данные используются для автоматической настройки базы при запуске контейнера.

### Шаг 6 : Создание файла app.env

В корне проекта был создан файл `app.env`. Данный файл используется для хранения пользовательских переменных окружения, которые можно переиспользовать в нескольких сервисах. В него вписываю следующую строку:

```env
APP_VERSION=1.0.0
```

### Шаг 7 : Запуск контейнера

Для запуска я использую команду `docker-compose up -d`

![image](/images/zapusk.png)

Далее я перехожу в свой браузер на адрес `localhost` и видим, что все работает прекрасно.

![finish](/images/finish.png)

### Вопросы и ответы

**1. В каком порядке запускаются контейнеры?**

Контейнеры запускаются в порядке, определенном в `docker-compose.yml`. Однако зависимости между ними (например, `php-fpm` и `mysql`) не гарантируют строгий порядок, если не настроены `depends_on`. На практике `frontend` будет ждать `backend`, если nginx настроен правильно.

**2. Где хранятся данные базы данных?**

Данные MySQL хранятся в Docker volume `db_data`, смонтированном в `/var/lib/mysql` внутри контейнера `database`.

**3. Как называются контейнеры проекта?**

Имена контейнеров формируются на основе имени директории и названия сервиса, например:

* `containers06_frontend_1`
* `containers06_backend_1`
* `containers06_database_1`

**4. Вам необходимо добавить еще один файл `app.env` с переменной окружения `APP_VERSION` для сервисов backend и frontend. Как это сделать?**

Создаю файл `app.env` с переменной:

```env
APP_VERSION=1.0.0
```

Нужно подключить его к соответствующим сервисам в `docker-compose.yml` с помощью параметра:

```yaml
env_file:
  - app.env
```

## Выводы

В ходе лабораторной работы я:

* Ознакомился с принципами работы многоконтейнерных приложений.
* Настроил взаимодействие nginx, php-fpm и mysql в рамках одного проекта.
* Использовал Docker Compose для автоматического развертывания приложения.
* Научился подключать внешние конфигурационные файлы и переменные окружения.

## Библиография

* [Repository by M.Croitor](https://github.com/mcroitor/app_containerization_ru/commits?author=mcroitor)
* [Docker Docs – Networking Overview](https://docs.docker.com/network/)
* [Docker Hub php:7.4-fpm](https://hub.docker.com/_/php)
* [Руководство про nginx](https://hub.docker.com/_/nginx)
