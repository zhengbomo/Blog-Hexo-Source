---
title: （一）树莓派4B开机
tags: [树莓派]
date: 2021-08-27 10:42:56
updated: 
categories: 树莓派
---


## 准备

### 1. 下载镜像写入工具

用官方的[Raspberry Pi Imager](https://www.raspberrypi.org/software)就行

### 2. 下载系统

* 官方系统: https://www.raspberrypi.org/software/operating-systems
* ubuntu: https://ubuntu.com/download/raspberry-pi
* 其他: https://www.raspberrypi.org/software/operating-systems

树莓派虽然很强大，但还是很难作为真正的生产力，大多数情况还是作为服务，我没有选择Desktop版本，而是选择`Ubuntu Server 20.04 TLS`

### 3. 写入TF卡

![](/images/post/raspberrypi_burn.png)

### 4. SSH连接

烧录完成后，先不急着开机，会提示拔出卡，重新插入tf卡，可以看到`system-boot`分区，这里我们在里面新建一个空的文本文件，命名为`SSH`，这样开机后就可以通过SSH连接了

### 5. wifi设置

如果你没有网线，也没有外界屏幕，可以设置wifi信息，树莓派在启动后会自动连接wifi，在`system-boot`分区根目录有个`network-config`文件，可以配置无线网络，找到`wifis`相关配置

```
#wifis:
#  wlan0:
#    dhcp4: true
#    optional: true
#    access-points:
#      myhomewifi:
#        password: "S3kr1t"
#      myworkwifi:
#        password: "correct battery horse staple"
#      workssid:
#        auth:
#          key-management: eap
#          method: peap
#          identity: "me@example.com"
#          password: "passw0rd"
#          ca-certificate: /etc/my_ca.pem
```
修改为（SSID: HomeWifi，密码：12345678）
```
wifis:
  wlan0:
    dhcp4: true
    optional: true
    access-points:
      "HomeWifi":
        password: "12345678"
```

### 6. 关闭LED指示灯

可以通过`config.txt`文件的`[pi4]`下面添加下面命令关闭

```txt
# 关闭电源指示灯(红色)
dtparam=pwr_led_trigger=none
dtparam=pwr_led_activelow=off

# 关闭活动指示灯(绿色)
dtparam=act_led_trigger=none
dtparam=act_led_activelow=off

# 关闭网线口指示灯
dtparam=eth_led0=4
dtparam=eth_led1=4
```

## 安装

TF卡插入树莓派通电即可，通过路由器可以看到树莓派连接的IP地址（如：10.10.10.11），通过ssh连接，ubuntu系统默认用户名和密码都是`ubuntu`

```sh
ssh ubuntu@10.10.10.11
```

接下来就可以愉快的玩耍了