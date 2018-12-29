---
title: "基於 Docker Compose 的 Discuz X3.4"
date: 2018-12-27T22:09:02+08:00
categories: [PHP, Docker, GitHub]
tags: [Docker, GitHub, PHP, FTP, Discuz, Docker-Compose]
---
### Discuz
> Discuz!是個可免費下載的PHP網路論壇程式，簡稱DZ，由戴志康（Crossday）所創立，目前最新版本是Discuz! X3.4。[2]前身為Crossday Bulletin（CDB），最初改自XMBForum，爾後改寫成為現今的Discuz!社群論壇程式，由康盛創想所有（現已被騰訊收購）。現在Discuz!已成為大中華地區最多用戶使用的論壇程式。

我們直接使用先前介紹 [架構一個基於Docker Compose 的 PHP Laravel 開發環境](2018/12/19/docker-compose-with-php-laravel-dev/) 的整個環境  
準備好 docker 及 docker compose 的環境後我們就可以開始建置 Discuz 了
```
git clone https://github.com/roryamos/LaravelDev.git Discuz
cd Discuz
tree
.
├── composer.sh
├── docker-compose.yml
├── html
│   ├── index.html
│   ├── index.php
│   └── phpinfo.php
├── log
│   ├── nginx
│   └── php
├── mariadb
│   ├── my.cnf
│   └── run.sh
├── nginx
│   ├── default.conf
├── phpfpm
│   ├── php.ini
│   └── www.conf
├── README.md
├── restart.sh
└── var-lib-mysql

8 directories, 12 files
```
#### 1. clone Discuz 最新的代碼
```
git clone https://gitee.com/ComsenzDiscuz/DiscuzX.git html/discuz
```
#### 2. 修改 Nginx 的路徑配置
```
vi nginx/default.conf
--------------------------------
    root /var/www/html/discuz/upload;
    location / {
        root   /var/www/html/discuz/upload;
        index  index.html index.htm index.php;
    }
--------------------------------
./restart.sh
```
打開網址瀏覽 http://youripaddress/, 調整安裝嚮導所提示的檔案權限.  
緊接著你會看到一個問題mysqli_connect() advice_mysqli_connect   
![Image of winrun](/images/discuz/php-mysqli-connect-error.png)
#### 3. 為 php-fpm 添加模塊
```
vi phpfpm/Dockerfile
-------------------------------------
FROM php:7.3-fpm
RUN docker-php-ext-install mysqli pdo pdo_mysql
-------------------------------------
vi docker-compose.yml
-------------------------------------
    phpfpm:
      # image: php:7.3-fpm
      build: phpfpm
-------------------------------------
./restart.sh
```
#### 4. ftp service
通常裝了 Discuz 後，還會需要 ftp 來協助安裝些模板模塊的, 所以我們連ftp 服務都一併搭建好  
以後利用 ftp client 就可以輕鬆管理 Discuz 了
```
vi docker-compose.yml
-------------------------------------
  ftp:
    image: bogem/ftp
    ports:
        - "20:20"
        - "21:21"
        - "47400-47470:47400-47470"
    volumes:
        - "./html/discuz:/home/vsftpd"
    environment:
        FTP_USER: ftpuser
        FTP_PASS: ftppass
        PASV_ADDRESS: 0.0.0.0
    restart: always
-------------------------------------
```
最後，我們照舊提供一個懶人包給看到最後的觀眾朋友們享用.
https://github.com/roryamos/DockerDiscuz
