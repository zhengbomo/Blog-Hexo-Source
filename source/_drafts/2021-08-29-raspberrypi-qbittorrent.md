---
title: 树莓派安装qbittorrent
tags: []
date: 2021-08-29 10:55:07
updated:
categories:
---

```sh
sudo apt install qbittorrent-nox
```

启动

```sh
qbittorrent-nox -d
#端口号默认是8080
 
#如果想要自定端口的话，请输入以下命令
qbittorrent-nox --webui-port=8083 -d
```
默认
用户名: admin
密码: adminadmin


开机启动

创建文件 `sudo vim /etc/systemd/system/qbittorrent.service`


```sh
[Unit]
Description=qbittorrent server
After=syslog.target
After=network.target

[Service]
RestartSec=2s
User=ubuntu
Group=watchdog
UMask=0002
ExecStart=/usr/bin/qbittorrent-nox --webui-port=8083

[Install]
WantedBy=multi-user.target
```


```sh
sudo systemctl daemon-reload
# 开机启动
sudo systemctl enable qbittorrent
# 启动
sudo systemctl start qbittorrent

sudo systemctl status qbittorrent
```
