---
title: 斐讯N1刷Armbian Linux做服务器
tags: [路由器, 斐讯]
date: 2018-10-15 11:48:43
updated:
categories: 路由器
---


N1上了不到两个月，斐讯就翻车了，现在N1也挖不了矿，作为NAS又太鸡肋，看到可以刷Armbian系统还是很激动的，可以作为服务器折腾一下，这里记录一下刷机的过程

<!-- more -->

## 工具准备
1. 双公头USB线，可以3.9淘宝一根，[https://detail.tmall.com/item.htm?id=13036924933](https://detail.tmall.com/item.htm?id=13036924933)
2. adb调试工具：[https://dl.google.com/android/repository/platform-tools-latest-windows.zip](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)
3. [DiskImager](https://sourceforge.net/projects/win32diskimager/): 降`img`文件写入U盘的工具
4. 降级分区：`boot.img`, `bootloader.img`, `recovery.img`
5. U盘一个：用于写入系统
6. PC一台：我这里用的是Win10
7. USB键盘一个：用于连接N1座一些初始化设置
8. HDMI线和显示器一台：用于连接N1做一些初始化设置
9. armbian固件下载：[https://yadi.sk/d/pHxaRAs-tZiei](https://yadi.sk/d/pHxaRAs-tZiei)，我选的是这个
    Armbian_5.62_Aml-s9xxx_Ubuntu_xenial_default_4.18.7_desktop_20181012.img.xz

## 降级
先降级，然后刷入比较保险，有些帖子说不用降级，但我没成功，还是先降级稳妥些

1. 先打开adb模式：在N1的主界面的【固件版本】点击4次，会看到`adb打开`的提示
    ![](/images/post/1541040423315.jpg)
2. N1与PC需要在同一个局域网，我的N1的IP是：`10.10.10.120`
3. 测试连接是否成功：在终端输入
    ```
    # 进入adb工具目录
    cd path/to/adb
    
    adb connect 10.10.10.120    
    ```
    会看到返回`connected to 10.10.10.120`的提示，说明连接成功
    ![](/images/post/5451539908386_.pic_hd.jpg)
4. 使用双公头链接N1和PC：连接N1靠近HDMI的USB口
5. 用下面命令让N1重启为fastboot模式
    ```
    adb shell reboot fastboot
    ```
    这时候N1会重启，重启后没什么变化，可以通过`fastboot devices -l`命令查看设备
    ![](/images/post/WX20181101-134634.png)

## 刷机
### 刷入降级分区
```
# 进入工具目录
cd /path/to/fastboot

fastboot flash boot boot.img
fastboot flash bootloader bootloader.img
fastboot flash recovery recovery.img
```
如果没有错误提示，说明写入成功，接下来重启
```
fastboot reboot
```
重启完成后，就可以刷新固件了

### 制作U盘启动固件
1. 插入U盘，并格式化
2. 打开`Win32DiskImager`，选择`img`文件和`U盘盘符`
3. 点击写入，等待几分钟后写入成功
4. 写入完成后，可以看到有个Boot的磁盘
    * 5.62后的版本：修改根目录下的`uEnv.ini`文件，将`meson-gxl-s905x-khadas-vim.dtb`换成N1对应的`meson-gxl-s905d-p230.dtb`
    * 之前的版本：复制`dtb/meson-gxl-s905d-p230.dtb`到根目录，并重命名为`dtb.img`
5. 弹出U盘
6. U盘插入N1靠近HDMI的USB口

### 写入系统到N1
1. 连接N1的HDMI到显示器，N1断电重连
2. 显示器可以看到N1从U盘启动，加载U盘的ubuntu系统
3. 跟进提示配置即可，默认用户：root，密码：1234
4. 在`/root/`目录下，有两个文件`install.sh`和`install-2018.sh`，运行这个会把U盘的系统写到N1的`eMMC`，就可以脱离U盘使用了
    ```
    ./install.sh
    ```
5. 写入完成后重启系统，关机的时候拔出U盘
    ```
    # 重启
    reboot

    # 关机
    poweroff
    ```

## 初始化配置
先使用`armbian-config`配置系统和网络，我们先配置网络就行，其他根据需要配置

### 卸载红外模块
N1不支持红外线，下面命令关闭和删除红外服务
```bash
# 关闭红外服务
systemctl stop lircd.service lircd-setup.service lircd.socket lircd-uinput.service lircmd.service
# 卸载红外模块
apt remove -y lirc && apt autoremove -y
```

### 更新软件包
```bash
apt update && apt upgrade -y
```

### 挂在外置存储
插入外置硬盘或U盘，通过`fdisk -l`查看磁盘信息和分区
```shell
Disk /dev/mmcblk1: 7.3 GiB, 7818182656 bytes, 15269888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x91950000

Device         Boot   Start      End  Sectors  Size Id Type
/dev/mmcblk1p1      1368064  1617919   249856  122M  c W95 FAT32 (LBA)
/dev/mmcblk1p2      1619968 15269887 13649920  6.5G 83 Linux


Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa32f3aa6

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1           2 1953525167 1953525166 931.5G  7 HPFS/NTFS/exFAT
```
通过上面，看到硬盘分区为`/dev/sda1`通过`mount`挂在分区
```bash
# 如果不存在就创建
mkdir /mnt/usb_disk

# 挂载分区
mount /dev/sda1 /mnt/usb_disk
```
进入`/mnt/usb_disk`可以看到硬盘分区的文件

### 配置frp用于公网连接
参考这里：[frp内网穿透](/2018-10-18/frp-start/)
