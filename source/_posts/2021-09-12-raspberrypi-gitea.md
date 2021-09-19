---
title: （六）树莓派4B安装gitea
tags: [树莓派]
date: 2021-09-12 16:56:54
updated: 2021-09-12 16:56:54
categories: 树莓派
---

最近git项目多了一些比较大的文件（数据库），第三方服务器的lfs基本都要收费，所以考虑自己搭建一个git服务器存放代码，关于git服务器有`gitea`，`gitlab`，gitlab功能太多，个人使用很多用不到，而且内存占用高，而gitea相对简洁，功能够用，选择gitea

<!-- more -->

## 安装

```sh
# 下载gitea
wget -O gitea https://dl.gitea.io/gitea/1.15.2/gitea-1.15.2-linux-arm64

# 改名
mv gitea-1.15.2-linux-arm64 gitea

# 添加权限
chmod +x gitea
```

## 启动

指定端口为`8899`

```sh
./gitea web -p 8899
```

访问`http://192.168.2.11:8899`就能进入gitea

{% img /images/post/raspberrypi/gitea_init.png 600 %}

由于是自己用，我这里选择SQLite，配置完成后，会跳转到`http://localhost:3000/user/login`，由于服务在树莓派上，我们需要把localhost改为树莓派的地址，配置完成后会`custom/conf/app.ini`生成配置文件，我们需要吧localhost换成对应的地址，如果绑定了域名，可以换成域名

```ini
APP_NAME = Gitea: Git with a cup of tea
RUN_USER = ubuntu
RUN_MODE = prod

[security]
INTERNAL_TOKEN     = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYmYiOjE2MzE0NjI1NjB9.zFm-QU16evTggiB-CZC2t36MAya4rPeQtUEkeNGGoiA
INSTALL_LOCK       = true
SECRET_KEY         = DwHi5bhFbvt1nX2FhzORtb85qfF1AawqWyB2By81lNEcqfcQfyTpAfrOwkMRKRxw
PASSWORD_HASH_ALGO = pbkdf2

[database]
DB_TYPE  = sqlite3
HOST     = 127.0.0.1:3306
NAME     = gitea
USER     = gitea
PASSWD   = 
SCHEMA   = 
SSL_MODE = disable
CHARSET  = utf8
PATH     = /home/ubuntu/server/gitea/data/gitea.db
LOG_SQL  = false

[repository]
ROOT = /home/ubuntu/server/gitea2/data/gitea-repositories

[server]
SSH_DOMAIN       = 192.168.2.11
DOMAIN           = 192.168.2.11
HTTP_PORT        = 8899
ROOT_URL         = http://192.168.2.11:8899/
DISABLE_SSH      = false
SSH_PORT         = 22
LFS_START_SERVER = true
LFS_CONTENT_PATH = /home/ubuntu/server/gitea/data/lfs
LFS_JWT_SECRET   = PfvIFZjn6C53f_zY96M3yeavAu8dLCUGEU-3CyEWFkc
OFFLINE_MODE     = true

[mailer]
ENABLED = false

[service]
REGISTER_EMAIL_CONFIRM            = false
ENABLE_NOTIFY_MAIL                = false
DISABLE_REGISTRATION              = false
ALLOW_ONLY_EXTERNAL_REGISTRATION  = false
ENABLE_CAPTCHA                    = false
REQUIRE_SIGNIN_VIEW               = false
DEFAULT_KEEP_EMAIL_PRIVATE        = false
DEFAULT_ALLOW_CREATE_ORGANIZATION = false
DEFAULT_ENABLE_TIMETRACKING       = true
NO_REPLY_ADDRESS                  = noreply.localhost

[picture]
DISABLE_GRAVATAR        = true
ENABLE_FEDERATED_AVATAR = false

[openid]
ENABLE_OPENID_SIGNIN = false
ENABLE_OPENID_SIGNUP = false

[session]
PROVIDER = file

[log]
MODE      = console
LEVEL     = info
ROOT_PATH = /home/ubuntu/server/gitea/log
ROUTER    = console
```

## 开机启动

添加一个系统用户`gitea`用于执行`gitea`

```sh
# 创建用户gitea，用户组为watchdog
sudo adduser --system --ingroup watchdog --disabled-password --shell /bin/bash --no-create-home --gecos 'Git Version Control' gitea

# 给用户添加文件权限
sudo chown -R gitea:watchdog /home/ubuntu/server/gitea
```

添加开机启动脚本`/etc/systemd/system/gitea.service`

```service
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
#After=mysqld.service

[Service]
RestartSec=2s
User=gitea
Group=watchdog
UMask=0002
WorkingDirectory=/home/ubuntu/server/gitea
ExecStart=/home/ubuntu/server/gitea/gitea -c /home/ubuntu/server/gitea/custom/conf/app.ini -p 8899

[Install]
WantedBy=multi-user.target
```

开机启动

```sh
# 重新加载配置文件
sudo systemctl daemon-reload

# 开机启动
sudo systemctl enable gitea
# 关闭开机启动
sudo systemctl disable gitea

# 启动
sudo systemctl start gitea
# 停止
sudo systemctl stop gitea
# 重启
sudo systemctl restart gitea

# 查看状态
sudo systemctl status gitea
```

接下来就可以直接在web页面玩耍了