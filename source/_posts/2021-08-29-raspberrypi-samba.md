---
title: 树莓派安装samba
tags: []
date: 2021-08-29 10:55:59
updated:
categories:
---

```sh
echo y | sudo apt install samba
```

配置 `/etc/samba/smb.conf`

```conf
[ubuntu]
  comment = 目录1
  path = /home/ubuntu
  browseable = yes
  writable = yes
  available = yes
  valid users = ubuntu,bomo
  guest ok = yes
  
[git]
  comment = 目录2
  path = /home/ubuntu
  browseable = yes
  writable = yes
  available = yes
  valid users = ubuntu,bomo
  guest ok = yes
  
[shared]
  comment = 目录3
  path = /home
  browseable = yes
  writable = no
  available = yes
  valid users = ubuntu,bomo
  guest ok = yes
```


https://www.cnblogs.com/kevingrace/p/8662088.html

设置用户组

```sh
sudo usermod -g root ubuntu
```

# 开机启动配置

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


# 重启服务

```sh
sudo systemctl daemon-reload
# 开机启动
sudo systemctl enable smbd
# 启动
sudo systemctl start smbd
# 查看运行状态
sudo systemctl status smbd
```