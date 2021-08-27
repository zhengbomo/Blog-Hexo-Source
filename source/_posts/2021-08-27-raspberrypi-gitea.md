---
title: （三）树莓派4B安装gitea
tags: [树莓派]
date: 2021-08-27 16:56:54
updated:
categories:
---

关于git服务器有`gitea`，`gitlab`，gitea功能太多，个人使用很多用不到，而且内存占用高，而gitea相对简洁，功能够用，

## 安装

```sh
# 下载gitea
wget -O gitea https://dl.gitea.io/gitea/1.15/gitea-1.15-linux-arm64

# 添加权限
chmod +x gitea-1.15-linux-arm64

# 改名
mv gitea-1.15-linux-arm64 gitea
```

添加一个系统用户用于执行gitea

```sh
sudo adduser --system --group --disabled-password --shell /bin/bash --home /home/git --gecos 'Git Version Control' git
```