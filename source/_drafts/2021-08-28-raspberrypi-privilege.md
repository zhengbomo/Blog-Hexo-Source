---
title: Linux的权限管理
tags: []
date: 2021-08-28 19:09:11
updated:
categories:
---

Linux系统有这复杂的权限系统，这里整理常见的一些权限

## 用户权限

在linux的用户系统中，一个用户可以属于一个组，
一个文件的权限可以分为，所有者，所有组，其他用户


## 创建用户

```sh
# 创建组
sudo addgroup watchdog

# gitea（不创建home目录，加入watchdog组，不用密码）
sudo adduser --no-create-home --system --ingroup 'watchdog' --disabled-password --shell /bin/bash gitea

# 可道云
sudo adduser --no-create-home --system --ingroup 'watchdog' --disabled-password --shell /bin/bash kodexplorer
```

删除用户/组

```sh
# 删除用户
sudo userdel gitea

# 删除组
sudo groupdel gitea
```

查看用户和组

```sh
id gitea
```

修改用户所在组

```sh
sudo usermod -g watchdog gitea

# 添加用户到组
sudo gpasswd -a www-data watchdog

sudo gpasswd -a ubuntu watchdog
```

修改文件夹权限

```sh
# 修改文件夹所属用户
sudo chown -R watchdog /home/ubuntu

# 修改文件夹所属用户和组
sudo chown -R ubuntu:watchdog /home/ubuntu

# 修改文件夹所属组
sudo chgrp watchdog /home/ubuntu
```

修改权限（可道云访问）
```sh
sudo chmod -R 774 /home/ubuntu
```

修改所有用户（login用户）的权限，在 `/etc/profile`添加

```sh
umask 006
```


## 文件权限

## 数据库权限

## 服务权限