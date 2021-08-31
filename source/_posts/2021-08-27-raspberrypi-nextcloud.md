---
title: （三）树莓派4B安装nextcloud
tags: [树莓派]
date: 2021-08-27 16:21:33
updated:
categories:
---


## 安装

nextcloud依赖 `apache2`, `php`, `mysql`（如果使用sqlite存储则不用）, 这里使用`mariadb-server`代替mysql

```sh
sudo apt update

# 安装apache2
sudo apt install apache2

# 安装php
sudo apt install php

# 安装mariadb
sudo apt install mariadb-server

# 解压工具
sudo apt install unzip
```

安装php依赖

```sh
sudo apt install php-gd php-json php-mysql php-curl php-mbstring 
sudo apt install php-intl php-mcrypt php-imagick php-xml php-zip
```

下载[nextcloud](https://nextcloud.com/install/#instructions-server)，找到最新版本`22.1.0`

```sh
# 下载
wget https://download.nextcloud.com/server/releases/nextcloud-22.1.0.zip

# 解压
unzip nextcloud-22.1.0.zip

# 移动到
sudo mv nextcloud /var/www/html
``` 

## 配置mysql

如果是首次安装，先设置账号密码，让`root`支持外部访问，这里密码设为: `111111`

```sql
CREATE USER 'root'@'%' identified by '111111';
GRANT ALL PRIVILEGES on *.* to 'root'@'%';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '111111';
FLUSH PRIVILEGES;
```

创建新用户nextcloud用于访问数据库

```sql
use mysql;
CREATE user 'nextcloud'@'%' identified by '111111';
GRANT ALL PRIVILEGES on *.* to 'nextcloud'@'%';
ALTER USER 'nextcloud'@'%' IDENTIFIED WITH mysql_native_password BY '111111';
FLUSH PRIVILEGES;
```

创建数据库

```sql
CREATE DATABASE nextcloud;
```

## 配置apache2

在`/etc/apache2/sites-available/000-default.conf`配置新站点，使用8081端口

```conf
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# nextcloud
<VirtualHost *:8081>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/nextcloud

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

在`/etc/apache2/ports.conf`添加端口

```conf
Listen 80

# 添加8081端口监听
NameVirtualHost *:8081
Listen 8081

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

## 配置权限

apache2默认情况下是通过`www-data`用户运行的，修改该用户的文件权限，在 `/etc/apache2/envvars`添加umask设置（ubuntu）

```sh
# 修改组为watchdog
export APACHE_RUN_GROUP=www-data

# 添加
umask 0002
```

给`www-data`用户添加组，用于访问

## 重启apache

```sh
sudo systemctl restart apache2
```