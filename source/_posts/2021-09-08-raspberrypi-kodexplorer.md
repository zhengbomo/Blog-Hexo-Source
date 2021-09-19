---
title: （五）树莓派安装可道云
tags: [树莓派]
date: 2021-09-08 22:11:26
updated:
categories: 树莓派
---

文件管理的工具有很多，如`filebrowser`，`可道云`，`nextcloud`，`seafile`，几种都试过，最终觉得可道云最符合自己的操作习惯，可道云有两个版本，`kodbox`和 `kodexplorer`，kodbox为kodexplorer的重构版本，新增了一些功能，个人用我认为kodexplorer就够了

<!-- more -->

## 下载安装

可道云是基于`php`开发的，这里需要先安装`apache2`和`php`

```sh
sudo apt update

# apache2
sudo apt install apache2

# 安装php
sudo apt install php

# 解压
sudo apt install unzip
```

安装php依赖

```sh
sudo apt install php-curl php-mbstring 
```

下载可道云

```sh
# 下载
wget https://static.kodcloud.com/update/download/kodexplorer4.46.zip

# 解压
unzip kodexplorer4.46.zip

# 移动
sudo mv kodexplorer /var/www/kodexplorer

# 设置权限
sudo chmod -R 777 /var/www/kodexplorer
```

## 配置apache2

修改`apache2`站点配置 `/etc/apache2/sites-available/000-default.conf`，添加端口`8081`用于可道云（也可以使用原来的80端口）

```conf
# 这是apache默认的站点
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# 可道云
<VirtualHost *:8081>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/kodexplorer
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

添加端口监听 `/etc/apache2/ports.conf`

```conf
Listen 80

# 可道云端口
NameVirtualHost *:8081
Listen 8081

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

配置完成后，重启`apache2`

```sh
sudo systemctl restart apache2
```

这时候就可以访问可道云了 `http://192.168.2.*:8081`

## 权限

`apache2`默认使用的用户为`www-data`，这里给`www-data`添加组权限，这样管理文件方便点（这里我用的组是`watchdog`，可以根据自己的习惯或需要设置）

```sh
# 添加www-data到组watchdog
sudo gpasswd -a www-data watchdog

# 也可以www-data为watchdog组
sudo usermod -g watchdog www-data
```

## 其他设置

在`可道云`上默认的文件创建权限为`755`，可以到`config/config.php`修改，我自己是改为`774`，组内成员可以修改

```php
define('DEFAULT_PERRMISSIONS',0755);	//新建文件、解压文件默认权限，777 部分虚拟主机限制了777;
```

