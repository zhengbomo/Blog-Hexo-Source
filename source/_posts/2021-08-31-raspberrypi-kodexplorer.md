---
title: （四）树莓派安装可道云
tags: [树莓派]
date: 2021-08-31 22:11:26
updated:
categories:
---

可道云有两个版本，`kodbox`和 `kodexplorer`，kodbox为kodexplorer的重构版本，新增了一些功能，个人用我认为kodexplorer就够了


## 下载安装

可道云是基于php开发的，这里需要先安装`apache2`和`php`

```sh
sudo apt update

# apache2
sudo apt install apache2

# 安装php
sudo apt install php
```

安装php依赖库

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

修改`apache2`站点配置 `/etc/apache2/sites-available/000-default.conf`，添加端口8081用于可道云（也可以使用原来的80端口）

```conf
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

重启apache2

```sh
sudo systemctl restart apache2
```

这时候就可以访问可道云了

## 权限

apache2是使用

设置www-data权限

修改可道云创建文件的权限

```php
```

可道云上传图片默认会进行压缩的，修改`static/app/vender/webuploader/webuploader.js`的`compress`为false