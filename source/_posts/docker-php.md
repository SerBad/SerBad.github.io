---
title: Docker在PHP中的实践过程
date: 2017-09-18 20:59:36
tags: Docker
update:
comments: false
---
最近微服务很火，很多人都在尝试，我们公司也在这段时间尝试着来时间微服务化，其中就涉及到Docker。
在实践docker中踩了很多坑，也对Docker有了更多的认识，下面记录一下。
<!--more-->

Docker在打包Spring boot项目时候，因为Spring boot内部集成了tomcat并且提供了直接打包成jar包的方式，[Spring boot如何打包](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-maven-plugin.html)。

Dockerfile文件如下：
```Java
FROM java:8

COPY target/your-project-name.jar /app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]

```

因为公司中还有一个之前的PHP项目，所以也需要对PHP进行打包，查了很多资料，最后在实践中发现，PHP项目需要一个nginx来管理网络，用如何的Dockerfile文件打包的话就会有问题。
```java
FROM php:7.0-cli
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
FROM php:7.1-fpm
RUN pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis
RUN docker-php-ext-install  mysqli pdo pdo_mysql

EXPOSE 80

COPY . /php
WORKDIR /php
```
经过再三查找资料，发现这么一篇文章：
http://geekyplatypus.com/dockerise-your-php-application-with-nginx-and-php7-fpm/
在里面找到了解决办法，使用docker-compose就好了，docker-compose.yml文件如下：
```yml
version: '2'

services:
    web:
        image: nginx:latest
        ports:
            - "8080:80"
        volumes:
            - ./:/php
            - ./site.conf:/etc/nginx/conf.d/default.conf
        networks:
            - code-network
    php:
        image: php:fpm
        volumes:
            - ./:/php
        networks:
            - code-network

networks:
    code-network:
        driver: bridge

```
上面提到的site.conf文件如下：
```conf
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /php;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```
直接在命令行执行：
```
  $sudo docker-compose up
```
就可以看看到运行的结果了，可是因为我们是直接部署到阿里云容器服务上面的，docker-compose这种方式就有了缺陷，因为阿里云容器服务不支持``server``这种方式，不得已最终经过查询，找到了：
https://hub.docker.com/r/richarvey/nginx-php-fpm/

直接使用如下Dockerfile就可以了：
```Java
FROM richarvey/nginx-php-fpm:latest

MAINTAINER Ric Harvey <ric@ngd.io>

ENV PHPREDIS_VERSION 3.0.0
RUN mkdir -p /usr/src/php/ext/redis \
    && curl -L https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz | tar xvz -C /usr/src/php/ext/redis --strip 1 \
    && echo 'redis' >> /usr/src/php-available-exts \
    && docker-php-ext-install redis

EXPOSE 80

ADD ./ /var/www/html/
```

既然当初选择了Docker这种方式，就要想办法吃透这门技术，否则在生产环境中的话出了问题就麻烦了。以上对与PHP的Docker化的实践过程，盯着一行行的日志信息查找原因，因为对PHP不是特别熟，还向PHP的同事请教了很多问题。
记录下来面的日后再遇到，虽然以上内容没有针对每个Docker的技术点进行介绍，找时间把每个技术点都记录下来。简单来说就是，PHP必须要搭配nginx使用。
