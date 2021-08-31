---
title: 树莓派开机自动挂载硬盘
tags: []
date: 2021-08-28 19:13:42
updated:
categories:
---


挂载硬盘

```sh
# 查看硬盘
sudo fdisk -l

# 挂载硬盘
sudo mount /dev/sdb1 /mnt/h1

# 挂在硬盘可以设置目录的所有者和umask

# 设置用户和组
sudo mount -o umask=0002,gid=watchdog,uid=ubuntu /dev/sdb2 /mnt/h1

# 取消挂载
sudo umount /dev/sdb1
```

## 自动挂载硬盘

1. 查看硬盘UUID
```sh
# 1. 查看硬盘uuid
sudo blkid

/dev/sdb2: LABEL="H1" BLOCK_SIZE="512" UUID="607D042E218EF09B" TYPE="ntfs" PARTUUID="11489bc6-30c9-4344-9a29-e00ae46c88a7"
```
得到UUID为："6129-D165"

2. 开机设置 `/etc/fstab`

```sh
UUID="607D042E218EF09B" /mnt/h1 ntfs defaults 0 1

# 添加umask
UUID="607D042E218EF09B" /mnt/h1 ntfs user,rw,umask=002,uid=ubuntu,gid=watchdog 0 1

```



使用 `df`命令查看硬盘挂载情况

```sh
ubuntu@ubuntu:~$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
tmpfs             189000    4080    184920   3% /run
/dev/sda2       30449484 4382088  24779448  16% /
tmpfs             944992       0    944992   0% /dev/shm
tmpfs               5120       0      5120   0% /run/lock
tmpfs               4096       0      4096   0% /sys/fs/cgroup
/dev/sda1         258095  129706    128389  51% /boot/firmware
tmpfs             188996       4    188992   1% /run/user/1000
/dev/sdb2      976556028   95836 976460192   1% /mnt/h1
```

格式化

```sh
# 把硬盘分区格式化为ntfs
sudo mkfs -t ntfs /dev/sdb1

# 
```

