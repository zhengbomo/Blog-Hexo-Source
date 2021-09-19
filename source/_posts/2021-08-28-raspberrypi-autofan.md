---
title: （二）树莓派4B自动控制风扇开关
tags: [树莓派]
date: 2021-08-28 11:27:39
updated:
categories:
---

树莓派4B性能强大，发热也大，我在上面跑可道云和aria2，在没有风扇的情况下基本都要到60-70℃了，通常都会买个外壳接一个小风扇用于散热，淘宝上有很多，挑一个自己喜欢的，我这台设备加了风扇后可以稳定在40-50之间

默认情况下，风扇是随着电源开关控制的，即使关机了，风扇也会转，这里介绍使用三极管控制风扇开关的方法

<!-- more -->

{% img /images/post/raspberrypi/raspberrypi_fan.jpg 300 %}

## 接线引脚

树莓派4B的引脚如下图

{% img /images/post/raspberrypi/raspberrypi_pin.png 400 %}

买来的风扇的正负极接4, 6引脚

## 通过三极管添加控制线

风扇接上树莓派引脚后就会开启，随电源开关，无法进行控制，关机的时候也会转，通常有两种方式

1. 使用三极管接线从而达到控制风扇的目的
2. taobao买T9温控模块（https://item.taobao.com/item.htm?id=553295324487）

这里第一种方式，添加三极管

* `三极管`，我这里用的是S8050（NPN型）的三极管
    我是在这里买的，2.8块钱50个
* `杜邦线-公对母`: 2根
* `杜邦线-母对母`: 2根

三极管三级

{% img /images/post/raspberrypi/triode.jpg 250 %}

接线示意图（分别接到4，6，12号引脚上）

{% img /images/post/raspberrypi/fan_wiring.png 500 %}

效果图

{% img /images/post/raspberrypi/raspberrypi_fan_final.jpg 800 %}

> 有朋友可能买到的是S8850（PNP型）的三极管，接线和上面不一样，需要注意，可以参考这个链接，不过我没试过，[https://blog.csdn.net/Xxy605/article/details/115960846](https://blog.csdn.net/Xxy605/article/details/115960846)

接完之后开机，会发现风扇默认是不转的，我们需要手动控制风扇的开关

## 通过python脚本控制开关

安装依赖文件

```sh
sudo apt update
sudo apt install python3-pip

sudo apt -y install python3-rpi.gpio
sudo pip install RPi.GPIO
```

下面脚本控制风扇开关

```python
import RPi.GPIO as GPIO

# 控制线接的是12号引脚
FAN = 12

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BOARD)

# 设为输出模式
GPIO.setup(FAN, GPIO.OUT)

# 开风扇
GPIO.output(FAN, GPIO.HIGH)

# 关风扇
GPIO.output(FAN, GPIO.LOW)

# 设为输入模式
GPIO.setup(FAN, GPIO.IN, pull_up_down=GPIO.PUD_UP)

# 读取当前风扇状态
isOff = GPIO.input(FAN) == GPIO.LOW;

# 关闭
GPIO.cleanup()
```

可以`PWM`控制风扇的风速

```py
pwm = GPIO.PWM(FAN, 50)

# 满速（0-100）
pwm.start(100)

# 关闭
pwm.stop()
```

## 查看CPU温度

通过读取文件`/sys/class/thermal/thermal_zone0/temp`获得CPU温度

```sh
# 查看当前CPU温度
cat /sys/class/thermal/thermal_zone0/temp

# 观察CPU温度，每秒更新一次
watch -n 1 cat /sys/class/thermal/thermal_zone0/temp
```

## 温控风扇脚本

下面是通过温度控制风扇开关的脚本（由于我的风扇比较小，就没有考虑控制风速，只做开关）

```py
# -*- coding: utf-8 -*-
 
import RPi.GPIO as GPIO
import time
 
# 控制风扇的GPIO
FAN_GPIO = 12
# 低温阈值，低于它则关闭风扇
MIN_TEMP = 45
# 高温阈值，高于它则全速运转
MAX_TEMP = 50
# 多长时间读取一次CPU温度，单位秒
SAMPLING = 60
 
 
# 单位为千分之一度
def get_cpu_temp():
    with open('/sys/class/thermal/thermal_zone0/temp') as f:
        cpu_temp = int(f.read())
    return cpu_temp
 
def main():
    GPIO.setwarnings(False)
    GPIO.setmode(GPIO.BOARD)
    GPIO.setup(FAN_GPIO, GPIO.OUT)
    GPIO.output(FAN_GPIO, GPIO.LOW) 
    
    time.sleep(5)
 
    try:
        while 1:
            temp = get_cpu_temp()
            print('CPU temperature:', temp)
            if temp < MIN_TEMP * 1000:
                GPIO.output(FAN_GPIO, 0)
            elif temp > MAX_TEMP * 1000:
                GPIO.output(FAN_GPIO, 1)
            else:
                # 中间地带，开
                GPIO.output(FAN_GPIO, 1)
            time.sleep(SAMPLING)
    except KeyboardInterrupt:
        pass
 
    GPIO.cleanup()


if __name__ == '__main__':
    main()
```

脚本保存到`/home/ubuntu/server/fan/autofan.py`

## 开机启动

创建service（`/etc/systemd/system/autofan.service`）

```service
[Unit]
Description=auto fan control
After=syslog.target
After=network.target

[Service]
RestartSec=2s
User=root
Group=root
WorkingDirectory=/home/ubuntu/server/fan/
ExecStart=/usr/bin/python3 /home/ubuntu/server/fan/autofan.py

[Install]
WantedBy=multi-user.target
```

刷新并加载

```sh
# 重新加载配置  
sudo systemctl daemon-reload
# 开机启动
sudo systemctl enable autofan
# 启动
sudo systemctl start autofan
# 状态
sudo systemctl status autofan
```

之后每次重启都会自动根据温度开关风扇了
