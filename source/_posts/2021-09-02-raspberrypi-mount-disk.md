---
title: （三）树莓派开机自动挂载硬盘
tags: [树莓派]
date: 2021-09-02 19:13:42
updated:
categories: 树莓派
---

我的硬盘是长期接在树莓派上的，这里设置开机自动挂载，避免重启后，读不到硬盘数据

<!-- more -->

## 挂载硬盘

```sh
# 查看硬盘，得到硬盘为/dev/sda1
sudo fdisk -l

# 需要确保挂载的目录是存在的，如果不存在则创建
mkdir /mnt/h1

# 1. 挂载硬盘
sudo mount /dev/sdb1 /mnt/h1

# 2. 挂载硬盘可以设置目录的所属用户所属组和umask
sudo mount -o umask=0002,gid=watchdog,uid=ubuntu /dev/sdb1 /mnt/h1

# 取消挂载
sudo umount /dev/sdb1
```

## 自动挂载硬盘

1. 查看硬盘UUID
```sh
# 1. 查看硬盘uuid
sudo blkid

# 输出得到下面信息
/dev/sda1: LABEL="pi" UUID="3E5F551D2B409931" TYPE="ntfs" PTTYPE="atari"
```

得到UUID为：`3E5F551D2B409931`

2. 开机挂载`/etc/fstab`

```sh
UUID="3E5F551D2B409931" /mnt/h1 ntfs defaults 0 1

# 添加所属用户，组，umask
UUID="3E5F551D2B409931" /mnt/h1 ntfs user,rw,umask=0002,uid=ubuntu,gid=watchdog 0 1
```

使用 `df`命令查看硬盘挂载情况

```sh
ubuntu@ubuntu:~$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
tmpfs             189000    4080    184920   3% /run
tmpfs             944992       0    944992   0% /dev/shm
tmpfs               5120       0      5120   0% /run/lock
tmpfs               4096       0      4096   0% /sys/fs/cgroup
/dev/sda1      976760828 72838540 903922288   8% /mnt/h1
```

完成

