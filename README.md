# Porpose

set up lnmp for laravel in docker container

# Prerequisites
- Install [Docker](https://www.docker.com/community-edition#/download)
- Install [Laravel](https://laravel.com/docs/5.6/installation)

# Starting up
## Step 1
In workspace, run `laravel new docker-laravel` to create a project `docker-laravel`
    
## Step 2
In side of docker-laravel root folder, create `docker-compose.yml` file
    
docker-compose.yml:
```
version: '3'

services:
  webserver:
    container_name: webserver
    build: ./images/nginx
    ports:
      - "8080:80"
    volumes:
      - ./:/var/www/html
      - ./images/nginx/vhost.conf:/etc/nginx/conf.d/default.conf
    working_dir: /var/www/html
    links:
      - application
      - redis

  application:
    container_name: application
    build: ./images/application
    ports:
      - "9500:9000"
    volumes:
      - ./:/var/www/html
    working_dir: /var/www/html
    links:
      - database
    environment:
      - DB_PORT=3306
      - DB_HOST=database

  database:
    container_name: database
    image: mysql:5.7
    ports:
      - "3306:3306"
    volumes:
      - ./dbdata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=docker-database
  redis:
    container_name: redis
    image: redis:latest
    ports:
      - "6500:6379"

```    
## Step 3
create Dockerfile for application container
    
./images/application/Dockerfile:    
```
FROM php:7.1.12-fpm

RUN apt-get update && apt-get install -y libmcrypt-dev mysql-client \
    && docker-php-ext-install mcrypt pdo_mysql

```    
## Step 4
create vhost.conf for Nginx container

./images/nginx/vhost.conf:    
```
server {
    listen 80;
    index index.php index.html;
    root /var/www/html/public;

    location / {
        try_files $uri /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass application:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```
## Step 5
run `$ docker-compose up -d` to start those containers

`$ docker-compose ps` can show you the current running containers
```
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
b328ba1b437d        nginx:latest                "nginx -g 'daemon ..."   9 hours ago         Up 9 hours          0.0.0.0:8080->80/tcp     webserver
7c17d078882c        dockerlaravel_application   "docker-php-entryp..."   16 hours ago        Up 9 hours          0.0.0.0:9500->9000/tcp   application
74065ee9c417        redis:latest                "docker-entrypoint..."   16 hours ago        Up 9 hours          0.0.0.0:6500->6379/tcp   redis
bb685c64b0e4        mysql:5.7                   "docker-entrypoint..."   17 hours ago        Up 9 hours          0.0.0.0:3306->3306/tcp   database
```

     
