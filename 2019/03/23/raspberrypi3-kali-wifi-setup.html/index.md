# 树莓派3B-Kali-Linux-WiFi配置绕坑指南

<!--more-->
0x00 前言
=========

今天有空给树莓派烧了个2019.1 64位的镜像，但在配置WiFi的时候却遇到了一大堆坑，故写此文来记录一下绕坑过程。

0x01 前期准备
=============

- 树莓派3B 一块
- 读卡器一个
- 8GB以上Class 10内存卡一张
- 电脑一台(实在没有手机也行)
- Micro USB数据线一根
- 5V 2A电源适配器一个

0x02 下载镜像
=============

Windows
-------

下载地址 https://images.offensive-security.com/arm-images/kali-linux-2019.1-rpi3-nexmon-64.img.xz

Linux
-----

```bash
wget https://images.offensive-security.com/arm-images/kali-linux-2019.1-rpi3-nexmon-64.img.xz
```

0x03 解压镜像并烧录到内存卡
===========================

Windows
-------

- 7-Zip [传送门](https://www.7-zip.org/a/7z1900-x64.exe)
- Etcher [传送门](https://www.balena.io/etcher/)

Linux
-----

```bash
xzcat kali-linux-2019.1-rpi3-nexmon-64.img.xz | dd of=/dev/sdb bs=512k
```
PS:此处的``/dev/sdb``是你的内存卡在Linux下的磁盘盘符，具体以``fdisk -l``的输出结果为准！

0x04 SSH登录树莓派
==================

首先需要将树烧录好镜像的内存卡插入到树莓派然后接通电源等待开机。然后可以通过将网线插入到树莓派的RJ45接口或者用手机USB共享网络的方式给树莓派提供网络。

接着在终端下执行``arp -a``来获得树莓派的IP地址。(我这里通过安卓手机USB共享网络来获得，IP为192.168.43.74，实际IP以你实际情况为准)
已获得树莓派IP地址后就可以SSH登录到树莓派，默认用户名为``root``，密码为``toor``.

Windows
-------

- Putty [传送门](https://the.earth.li/~sgtatham/putty/0.71/w64/putty.exe)

Linux
-----

```bash
ssh root@192.168.43.74
```

0x05 编辑Interfaces配置文件
===========================

```bash
vim /etc/network/interfaces
```
正常情况下配置文件里面是这样的内容
```bash
auto lo iface lo inet loopback

auto eth0 iface eth0 inet dhcp
```

如果要配置WiFi我们需要将这两行内容注释掉，并在最下面添加这几行内容
```bash
auto wlan0
iface wlan0 inet dhcp
wpa_conf /etc/wpa_supplicant/wpa_supplicant.conf
```
0x06 编辑WiFi配置文件
=====================

在/etc/wpa_supplicant/目录下创建一个名为``wpa_supplicant.conf``的文件并编辑以下内容
```bash
vim /etc/wpa_supplicant/wpa_supplicant.conf
```
```bash
network={

ssid="Your SSID"

psk="Your password"

key_mgmt=WPA-PSK

priority=

}
```
这里的ssid填为你自己WiFi的SSID，密码对应，key_mgnt选择加密方式，priority选择对应优先级，如果保存多个ssid的话优先级越大就最先选择这个WiFi去连接。这里我的例子为
```bash
network={

ssid="Hacker"

psk="1234567890"

key_mgmt=WPA-PSK

priority=5

}
```

根据你自己的实际情况编辑即可。编辑完以后重启network服务
```bash
service networking restart
```

之后就可以等待树莓派连接到指定的WiFi了！

0x07 参考链接
=============

[树莓派连接WiFi(最稳定的办法)](https://blog.csdn.net/hktkfly6/article/details/73302648)

[树莓派装kali linux wifi无线网络连接设置](https://www.zhaoyanchang.com/detail/10.html)

