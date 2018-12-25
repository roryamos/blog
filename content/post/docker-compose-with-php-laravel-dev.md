---
title: "架構一個基於Docker Compose 的 PHP Laravel 開發環境"
date: 2018-12-19T14:19:38+08:00
categories: [PHP, Docker, GitHub]
tags: [Docker, GitHub, PHP, Laravel, Docker-Compose]
---

### Docker Compose
> 操作過 Docker 指令的同學們，應該都嘗試過用指令的方式管理 Docker 容器, 當需要運行的容器愈來愈多時管理的指令是又多又複雜。
> Docker Compose 是一個定義多個 Docker 容器的工具，讓我們可以使用 YAML 格式的配置文件管理多個容器，
> 並幫助我們快速的佈建一套系統，提供了單一的命令，我們可以一次性的運行或停止相互依賴的容器。

#### 安裝 Docker
基本的安裝可以參考 [Docker 安裝筆記 - 以LNMP為例的應用](/2018/12/12/docker-install/)  

#### 安裝 Docker Compose
在安裝好Docker 之後、我們可以開始安裝 Docker Compose, 首先我們需要先安裝 python-pip
```
sudo yum install -y epel-release
sudo yum install -y python-pip
```
接下來就可以安裝Docker Compose了
```
pip install docker-compose
```
需要的話將 Python 相關的套件更新一下讓 Docker Compose 能夠順利運行
```
yum upgrade python*
```

### PHP Laravel
> Laravel 是一個網頁應用框架實現了MVC的架構，運用了 Composer 這個套件相依性管理系統，提供連接許多種類的RDBMS的方式，
> 提供開發語法糖使開發常見痛點的實現變的簡單，像是權限管理、路由、Sessions、及快取機制。

接下來我們試著以安裝 Laravel 的開發環境為例，進行 Docker Compose 的配置

#### 1. 初始化我們的目錄結構
```
mkdir -p html log/nginx log/php mariadb nginx phpfpm var-lib-mysql
docker pull mariadb/server:10.3
docker run --name mariadb -e MYSQL_ROOT_PASSWORD=passwd -d mariadb/server:10.3
docker cp mariadb:/etc/mysql/my.cnf ./mariadb/my.cnf
docker stop mariadb
docker rm mariadb

docker pull nginx:1.15
docker run --name nginx -v /data/html:/usr/share/nginx/html:ro -p 80:80 -d nginx:1.15
docker cp nginx:/etc/nginx/conf.d/default.conf ./nginx/default.conf
docker stop nginx
docker rm nginx

docker pull php:7.3-fpm
docker run --name php-fpm -p 9000:9000 -d php:7.3-fpm
docker cp php-fpm:/usr/local/etc/php-fpm.d/www.conf ./phpfpm/www.conf
docker cp php-fpm:/usr/local/etc/php/php.ini-production ./phpfpm/php.ini
docker stop php-fpm
docker rm php-fpm
tree .
.
├── docker-compose.yml
├── html
│   └── phpinfo.php
├── log
│   ├── nginx
│   └── php
├── mariadb
│   └── my.cnf
├── nginx
│   └── default.conf
├── phpfpm
│   ├── php.ini
│   └── www.conf
└── var-lib-mysql

8 directories, 6 files
```
#### 2. 配置docker-compose.yml
主要是運行 Mariadb, nginx, php-fpm 這三個容器
```
version: '2'
services:
  mariadb:
    image: mariadb/server:10.3
    ports:
        - "3306:3306"
    volumes:
        - "./var-lib-mysql:/var/lib/mysql"
        - "./mariadb/my.cnf:/etc/mysql/my.cnf"
    environment:
        MARIADB_USER: root
        MARIADB_ROOT_PASSWORD: root
    restart: always
  nginx:
    image: nginx:1.15
    ports:
        - "80:80"
    links:
        - phpfpm
    volumes:
        - "./nginx/default.conf:/etc/nginx/conf.d/default.conf"
        - "./html:/var/www/html"
        - "./log/nginx:/var/log/nginx"
    restart: always
  phpfpm:
    image: php:7.3-fpm
    links:
        - mariadb
    volumes:
        - "./phpfpm/www.conf:/usr/local/etc/php-fpm.d/www.conf"
        - "./phpfpm/php.ini:/usr/local/etc/php/php.ini"
        - "./html:/var/www/html"
        - "./log/php:/var/log/php"
    restart: always
```
#### 4. 配置 nginx default.conf
配置 Nginx HTML 的目錄, 及 php-fpm 運行的細節
```
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /var/www/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    location ~ \.php$ {
        root           /var/www/html;
        fastcgi_pass   phpfpm:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```
#### 5. 開關服務命令
```
docker-compose up --build -d
docker-compose down
```
你也可以把他製作成腳本 restart.sh
```
#!/bin/bash
docker-compose down
docker-compose up --build -d
```
#### 6. 用 composer 來安裝 Laravel
在本地因為我們沒有PHP可以運行的環境, 我們需要 composer 來幫我們安裝 laravel
```
docker pull composer
```
因為我們不需要將 composer 在後台執行, 所以我們只命令製作成一個腳本來使用 composer.sh
```
#!/bin/sh
export PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin
echo "Current working directory: '"$(pwd)"'"
docker run --rm -v $(pwd):/app -v ~/.ssh:/root/.ssh composer $@
```
接著我們安裝 Laravel
```
./composer.sh create-project laravel/laravel html/laravel
```
#### 7. 修改 nginx 配置，來支持 laravel 運行
```
server {
    listen       80;
    server_name  localhost;
    root /var/www/html/laravel/public;
    location / {
        try_files $uri $uri/ /index.php$query_string;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    location ~ ^/.+\.php(/|$) {
        fastcgi_pass   phpfpm:9000;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}   
```
#### 8. Hello World
我們編輯一下 html/laravel/routes/web.php 來讓首頁返回 Hello World 吧!
```
Route::get('/', function () {
    //return view('welcome');
    return "Hello World";
});
```

### Laravel 開發環境快速搭建法
提供一個 github repository 給需要的人享用. https://github.com/roryamos/LaravelDev  
只需要簡單幾個步驟，就完成了 Laravel 開發環境的建置
```
git clone https://github.com/roryamos/LaravelDev.git
cd LaravelDev
docker pull composer
./composer.sh create-project laravel/laravel html/laravel
vi html/laravel/routes/web.php
./restart.sh
```
如果需要測試環境是否正常，編輯一下 html/laravel/routes/web.php
```
vi html/laravel/routes/web.php
Route::get('/', function () {
    //return view('welcome');
    return "Hello World";
});
```
打開首頁 http://伺服器位置, 就可以看到 Hello World 了