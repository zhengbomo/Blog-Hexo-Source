---
title: （四）树莓派安装Samba
tags: [树莓派]
date: 2021-09-05 10:55:59
updated: 2021-09-05 10:55:59
categories: 树莓派
---

树莓派挂载了一个硬盘，用来存放文件，视频，照片，需要共享到其他设备查看，并且需要权限管理

常用的文件共享工具有

* `DLNA`: 主要用于多媒体共享，没有权限管理，所有人都可以看到，由于视频是服务器解码，对于大码率（4K）视频支持比较好
* `Samba`: 主要用于局域网文件共享，支持权限控制，大码率视频支持较弱
* `FTP`: 速度比Samba快，支持权限控制

<!-- more -->

我这里还是选择用`Samba`，因为电视和手机支持比较好，而对于大码率视频，则使用DLNA（minidlna），这里介绍安装samba的过程

## 安装

```sh
sudo apt update
sudo apt install samba
```

## 配置

配置文件在`/etc/samba/smb.conf`，下面配置放到文件最后面

```conf
[学习]
  comment = 学习，可读写，只有bomo可以查看
  path = /mnt/h1/learn
  browseable = yes
  writable = yes
  available = yes
  valid users = ubuntu,bomo
  guest ok = no
  
[视频]
  comment = 视频，所有人可以查看，只读
  path = /home/ubuntu/git
  browseable = yes
  writable = no
  available = yes
  guest ok = yes  
```

由于samba的用户必须是系统用户，这里我们创建用户`bomo`，并且不需要密码，不需要

```sh
# 添加系统用户bomo，属于组watchdog，无密码，不创建home目录
sudo adduser --system --ingroup watchdog --disabled-password --shell /bin/bash --no-create-home bomo
```

虽然samba的用户必须是系统用户，但是`密码`可以单独设置

```sh
# 设置用户samba密码
smbpasswd -a bomo
```

设置完成，重启samba

```sh
# 重启samba
sudo systemctl restart smbd
```

接下来就可以在其他支持samba的设备发现并登录了

## 设置文件权限

安装`samba`的时候，会自动添加开机启动服务`/lib/systemd/system/smbd.service`，这里我们添加`UMask`和`Group`，方便管理文件

```sh
[Unit]
Description=Samba SMB Daemon
Documentation=man:smbd(8) man:samba(7) man:smb.conf(5)
Wants=network-online.target
After=network.target network-online.target nmbd.service winbind.service

[Service]
Type=notify
NotifyAccess=all
PIDFile=/run/samba/smbd.pid
User=root
Group=watchdog
UMask=0002
LimitNOFILE=16384
EnvironmentFile=-/etc/default/samba
ExecStartPre=/usr/share/samba/update-apparmor-samba-profile
ExecStart=/usr/sbin/smbd --foreground --no-process-group $SMBDOPTIONS
ExecReload=/bin/kill -HUP $MAINPID
LimitCORE=infinity


[Install]
WantedBy=multi-user.target
```

刷新服务重启

```sh
sudo systemctl daemon-reload
# 启动
sudo systemctl restart smbd
```