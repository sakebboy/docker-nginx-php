# docker-nginx-php
Docker nginx + php7.2 + php-fpm 

## 運行指令
啟用安裝docker-compose

```docker-compose up -d```

執行運作docker-compose

```docker-compose start```

停止運作docker-compose

```docker-compose stop```

## docker-compose
```
+ nginx 
  + conf.d 
    - site.conf 
  - Dockerfile 
+ app 
  - index.html 
  - hello-world.php
+ php-cli
  - Dockerfile
+ php-fpm
  - Dockerfile
```
  
## nginx
FROM nginx:latest

## php-cli
FROM php:7.2-cli

## php-fpm
FROM php:7.2-fpm

## PHP modules
php-fpm, apcu, sqlsrv, pdo_sqlsrv, intl, mysqli, pdo, pdo_mysql, mbstring, bcmath, soap, zip, opcache, gd,ldap, oci8, pdo_oci

