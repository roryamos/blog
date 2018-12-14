---
title: "Docker install "
date: 2018-12-12T21:38:51+08:00
categories: [Docker]
tags: [Docker, LNMP, Nginx, Mariadb, PHP]
---
> Docker 是一個開源專案，誕生於 2013 年初，最初是 dotCloud 公司內部的一個業餘專案。它基於 Google 公司推出的 Go 語言實作。 專案後來加入了 Linux 基金會，遵從了 Apache 2.0 協議，原始碼在 GitHub 上進行維護。

## Docker Install
官網上有各種平台的安裝指南 https://docs.docker.com/install/

#### 我們介紹的是在 CentOS 下的安裝步驟 :
#### 1. 安裝一台乾淨的CentOS 7 Minimal, ISO https://www.centos.org/download/
#### 2. 安裝 Docker
```
# yum update
# yum install -y wget
# wget -qO- https://get.docker.com/ | sh
```
在安裝完成後會出現個提示訊息說，如果你要以非root用戶來運行Docker、你需要執行下面的命令。
```
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user
```
#### 3. 啟動 Docker
```
# systemctl enable docker.service
# systemctl start docker.service
```
## Docker 的應用以 LNMP 為例
### Mariadb
我們使用目前Mariadb 最新版本的docker image 10.3
```
# docker pull mariadb/server:10.3
```
用基本的方式指定root 的密碼來運行 Mariadb 容器
```
# docker run --name mariadb -e MYSQL_ROOT_PASSWORD=passwd -d mariadb/server:10.3
```
將容器內的 port 對外開放 -p 3306:3306
```
# docker run --name mariadb -e MYSQL_ROOT_PASSWORD=passwd -p 3306:3306 -d mariadb/server:10.3
```
需要配置 my.cnf 的話 -v /data/my.cnf:/etc/mysql/my.cnf
```
# docker cp mariadb:/etc/mysql/my.cnf my.cnf
# docker stop mariadb
# docker rm mariadb
# docker run --name mariadb -e MYSQL_ROOT_PASSWORD=passwd -v my.cnf:/etc/mysql/my.cnf -p 3306:3306 -d mariadb/server:10.3
```
### Nginx
```
# docker pull nginx:1.15
```
掛載網頁的根目錄並對外開放 80 port
```
# mkdir /data/html
# docker run --name nginx -v /data/html:/usr/share/nginx/html:ro -p 80:80 -d nginx:1.15
```
配置 conf -v /data/default.conf:/etc/nginx/conf.d/default.conf
```
# docker cp nginx:/etc/nginx/conf.d/default.conf /data/default.conf
# docker stop nginx
# docker rm nginx
# docker run --name nginx -v /data/default.conf:/etc/nginx/conf.d/default.conf -v /data/html:/usr/share/nginx/html:ro -p 80:80 -d nginx:1.15
```
### PHP
```
# docker pull php:7.3-fpm
```
運行 php-fpm 並對外開放 9000 port
```
# docker run --name php-fpm -p 9000:9000 -d php:7.3-fpm
```
配置 php-fpm
```
# docker cp php-fpm:/usr/local/etc/php-fpm.d/www.conf /data/www.conf
# docker cp php-fpm:/usr/local/etc/php/php.ini-production /data/php.ini
# docker stop php-fpm
# docker rm php-fpm
# docker run --name php-fpm -v /data/html:/usr/share/nginx/html -v /data/www.conf:/usr/local/etc/php-fpm.d/www.conf -v /data/php.ini:/usr/local/etc/php/php.ini -d php:7.3-fpm
```
配置Nginx 
```
# vi /data/default.conf
-----------------------------------
    location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   php-fpm:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
```
因為我們是安裝在一個主機裡, 接下來我們要讓三個容器能夠互相透過網路訪問, php-fpm link mariadb, nginx link php-fpm 
```
# docker stop php-fpm
# docker rm php-fpm
# docker run --name php-fpm --link mariadb -v /data/html:/usr/share/nginx/html -v /data/www.conf:/usr/local/etc/php-fpm.d/www.conf -v /data/php.ini:/usr/local/etc/php/php.ini -d php:7.3-fpm
# docker stop nginx
# docker rm nginx
# docker run --name nginx --link php-fpm -v /data/default.conf:/etc/nginx/conf.d/default.conf -v /data/html:/usr/share/nginx/html:ro -p 80:80 -d nginx:1.15
```