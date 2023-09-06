# Kali Linux NetHunter内核编译指南

<!--more-->
## 0x00 前言
近年来随着各种HVV活动的兴起，各种新的概念层出不穷。其中就有近源渗透这个概念。   
黑客行走江湖，哪儿能没有些趁手的兵器装备呢？ 相信很多人都曾梦想过拥有一台黑客专属手机，走到哪儿黑到哪儿。那么现实中这样的手机存在吗？答案是肯定的！NetHunter就能满足你所有的需求！  
[Kali Linux NetHunter](https://www.kali.org/docs/nethunter/)是由*Offensive-Security*团队打造的基于Android平台的渗透测试环境。  
通过使用*Kali Linux NetHunter*我们可以使用诸如外接无线网卡破解WiFi，模拟BadUSB设备进行HID攻击，外接USB蓝牙适配器进行蓝牙攻击......等各种近源渗透活动。  
在*Kali Linux NetHunter*[官网](https://www.kali.org/docs/nethunter/)我们可以查阅官方支持的设备型号列表。  
#### 对读者的要求
如果你会玩安卓刷机且手机型号恰好被官方支持，那么直接按照[官方教程](https://www.kali.org/docs/nethunter/installing-nethunter)一步步来就好。  
如果很不幸你的手机不被官方所支持但你会玩Linux且懂一些安卓开发以及C语言方面的知识想给自己的手机适配NetHunter，那么本篇教程就带你如何给一台不被官方支持的手机适配Kali NetHunter。  

#### 开始前的准备
· 一台能解锁BootLoader且内核源码开源的安卓手机  
· 一台高性能x86_64 PC  

#### 内核源码的选择
一般来说，手机厂商开源的内核源码代码质量参差不齐（一言难尽），如果我们要选择自己适配NetHunter的话最好选择知名第三方开发者Fork的源码进行编译。  
比较知名的有：
· [LineageOS](https://github.com/LineageOS/)  
· [PixelExperience](https://github.com/PixelExperience-Devices/)  
· [crDroid](https://github.com/crdroidandroid/)  
· [MoKee](https://github.com/mokee/)  
· [Havoc-OS](https://github.com/Havoc-OS/)  
· [Arter97](https://github.com/arter97/)  
...等，这里不再一一列举。

#### 交叉编译工具链的选择
对于较老版本的内核（3.18.x以下）的一般是使用Google GCC4.9  
对于较新版本的内核（4.4.x以上）的建议使用Clang来编译  
对于Google gcc编译器，使用以下命令下载  
64位：
```bash
git clone https://mirrors.bfsu.edu.cn/git/AOSP/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9 -b android-10.0.0_r32 --depth=1
```
32位：
```bash
git clone https://mirrors.bfsu.edu.cn/git/AOSP/platform/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9 -b android-10.0.0_r32 --depth=1
```
对于Clang编译器，使用以下命令下载  
Google官方Clang：
```bash
git clone https://mirrors.bfsu.edu.cn/git/AOSP/platform/prebuilts/clang/host/linux-x86 --depth=1
```
Proton-clang:
```bash
git clone https://github.com/kdrag0n/proton-clang.git --depth=1
```
#### 如何查找自己手机的内核源码
对于已经开源内核源码的手机来说，一般只需要在GitHub上搜索关键字就能找到适合你的内核源码  
一般搜索的关键字为`android_kernel_<设备厂商名>_<设备CPU代号名>`  
或者`kernel_<设备厂商名>_<设备CPU代号>`  
又或者`kernel_<设备厂商名>_<设备代号>`
举个例子来说，我的设备是*小米Redmi 4X*，设备厂商是*xiaomi*，CPU代号是*MSM8937*，设备代号是*santoni*那么就可以在GitHub上搜索关键字`android_kernel_xiaomi_msm8937`或者`kernel_xiaomi_santoni`或者`kernel_xiaomi_msm8937`来找对应设备的内核源码。  
这里还要注意的一点是所选取的内核源码尽量要与当前手机所使用的ROM Android版本对应，比如如果手机所使用的ROM是LineageOS的那就去找LineageOS所对应的内核源码，且分支也要一一对应。  
当然你也可以选择在[XDA论坛](https://www.xda-developers.com/)寻找其他第三方优秀作者提供的内核源码。


## 0x01 环境准备
我这里使用VMware虚拟机安装Kali Linux系统来进行演示

*Kali Linux*最新镜像 [下载链接](https://mirrors.bfsu.edu.cn/kali-images/kali-weekly/)  

*VMware Workstation Pro*虚拟机 [下载链接](https://www.vmware.com/cn/products/workstation-pro/workstation-pro-evaluation.html)  

*ADB-FASTBOOT工具* for Linux [下载链接](https://dl.google.com/android/repository/platform-tools-latest-linux.zip)

## 0x02 系统设置
#### 设置更新源
```bash
echo "deb https://mirrors.bfsu.edu.cn/kali kali-rolling main non-free contrib" > /etc/apt/sources.list
```
#### 更新系统
```bash
apt update && apt upgrade -y && apt full-upgrade -y && reboot
```

## 0x03 安装编译依赖
```bash
apt install -y curl wget vim git ccache automake flex lzop bison gperf \
build-essential zip zlib1g-dev g++-multilib libxml2-utils bzip2 libbz2-dev \
libbz2-1.0 libghc-bzlib-dev squashfs-tools pngcrush schedtool dpkg-dev \
liblz4-tool make optipng maven libssl-dev pwgen libswitch-perl \
policycoreutils minicom libxml-sax-base-perl libxml-simple-perl bc \
libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev \
libgl1-mesa-dev xsltproc unzip device-tree-compiler kmod python3 python3-pip
```

## 0x04 下载交叉编译工具链
```bash
git clone https://github.com/kdrag0n/proton-clang.git /root/proton-clang --depth=1
```

## 0x05 下载内核源码
```bash
git clone https://github.com/crdroidandroid/android_kernel_xiaomi_msm8937.git
cd android_kernel_xiaomi_msm8937
```

## 0x06 编译内核
#### 设置环境变量
```baah
export ARCH=arm64
export SUBARCH=arm64
export KBUILD_BUILD_HOST=kali
export KBUILD_BUILD_USER=root
export LOCALVERSION=-NetHunter
export PATH="/root/proton-clang/bin:$PATH"
mkdir out
args="-j$(nproc --all) \
ARCH=arm64 \
SUBARCH=arm64 \
O=out \
CC=clang \
CROSS_COMPILE=aarch64-linux-gnu- \
CROSS_COMPILE_ARM32=arm-linux-gnueabi- \
CLANG_TRIPLE=aarch64-linux-gnu- \
AR=llvm-ar \
NM=llvm-nm \
OBJCOPY=llvm-objcopy \
OBJDUMP=llvm-objdump \
STRIP=llvm-strip "
```
#### 打入补丁
这里根据你的内核版本选择对应内核版本的补丁(patches)  
我这里内核是4.9所以选择4.9内核的补丁
```bash
git clone https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-kernel.git
patch -p1 < kali-nethunter-kernel/patches/4.09/add-wifi-injection-4.14.patch
patch -p1 < kali-nethunter-kernel/patches/4.09/fix-ath9k-naming-conflict.patch
```

#### 生成defconfig
```bash
make ${args} mrproper
make ${args} santoni_treble_defconfig
```
#### 图形化配置内核选项

以下内容不同版本内核可能会有所不同，以实际情况为准！

```bash
make ${args} menuconfig
```
![menuconfig](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/menuconfig.png "menuconfig")
```sh
首先进入"Gerenal Setup"  
选择到"Local version - append to kernel release"  
清空里面所有内容  
然后取消勾选"Automatically append version information to the version string"  
接着选中"Default hostname"，输入"kali"  
接着勾选"System V IPC"  
然后返回上一级菜单  
```
如图所示  
![general](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/general.png "general")

```sh
接着进入到"Enable loadable module support"  
勾选以下几个选项:  
"loadable module support"  
"Forced module loading"  
"Modules unloading"  
"Forced module unloading"  
"Module versioning support"  
然后返回上一级菜单
```
如图所示  
![module](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/module.png "module")

```bash
接着进入到"Networking support" -> "Bluetooth subsystem support" -> "Bluetooth drivers support"  
勾选以下几个选项:  
"HCI USB driver"  
"Broadcom protocol support"  
"Realtek protocol support"  
"HCI UART driver"  
"HCI BCM203x USB driver"  
"HCI BPA10x USB driver"  
"HCI BlueFRITZ! USB driver"  
然后返回上一级菜单
```
如图所示  
![bluetooth-driver](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/bluetooth_driver.png "bluetooth-driver")

```bash
勾选以下几个选项：  
"Bluetooth Classic (BR/EDR) features"  
"RFCOMM protocol support"  
"RFCOMM TTY support"  
"BNEP protocol support"  
"HIDP protocol support"  
"Bluetooth Low Energy (LE) features"  
然后返回上一级菜单  
```
如图所示  
![bluetooth](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/bluetooth_subsystem.png "bluetooth")

```bash
进入到"Wireless"  勾选以下几个选项：  
"nl80211 testmode command"  
"use statically compiled regulatory rules database"  
"cfg80211 wireless extensions compatibility"  
"Generic IEEE 802.11 Networking Stack (mac80211)"  
"Enable mac80211 mesh networking (pre-802.11s) support"  
然后返回上一级菜单
```
如图所示  
![wireless](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/wireless.png "wireless")

```bash
接着进入到"Device Drivers" -> "Network device support" -> "USB Network Adapters"  
勾选以下几个选项：  
"USB RTL8150 based ethernet device support"  
"Realtek RTL8152/RTL8153 Based USB Ethernet Adapters"  
"ASIX AX88xxx Based USB 2.0 Ethernet Adapters"  
"ASIX AX88179/178A USB 3.0/2.0 to Gigabit Ethernet". 
然后返回上一级菜单
```
如图所示  
![usb_net](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/usb_network.png "usb_net")

```bash
接着进入到"Wireless LAN"  
勾选以下几个选项：  
"Atheros/Qualcomm devices"  
"Atheros HTC based wireless cards support"  
"Linux Community AR9170 802.11n USB support"  
"Atheros mobile chipsets support"  
"Atheros ath6kl USB support"  
"MediaTek devices"  
"MediaTek MT7601U (USB) support"  
"Ralink devices"  
"Ralink driver support"  
"Realtek devices"  
"Realtek 8187 and 8187B USB support"  
"Realtek rtlwifi family of devices"  
"RTL8723AU/RTL8188[CR]U/RTL819[12]CU (mac80211) support"  
"Include support for untested Realtek 8xxx USB devices (EXPERIMENTAL)"  
"ZyDAS devices"  
"USB ZD1201 based Wireless device support"  
"ZyDAS ZD1211/ZD1211B USB-wireless support"  
"Wireless RNDIS USB support"  

在"Ralink driver support"中勾选以下几个选项：  
"Ralink rt2500 (USB) support"  
"Ralink rt2501/rt73 (USB) support"  
"Ralink rt27xx/rt28xx/rt30xx (USB) support"  
"rt2800usb - Include support for rt33xx devices"  
"rt2800usb - Include support for rt35xx devices (EXPERIMENTAL)"  
"rt2800usb - Include support for rt3573 devices (EXPERIMENTAL)"  
"rt2800usb - Include support for rt53xx devices (EXPERIMENTAL)"  
"rt2800usb - Include support for rt55xx devices (EXPERIMENTAL)"  
"rt2800usb - Include support for unknown (USB) devices"  

在"Realtek rtlwifi family of devices" 中勾选
"Realtek RTL8192CU/RTL8188CU USB Wireless Network Adapter"  
然后返回主菜单
```
如图所示  
![Atheros](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/wireless_lan.png "Atheros")  

![MediaTek](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/mediatek.png "MediaTek")  

![Ralink](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/ralink.png "Ralink")  

![Realtek](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/realtek.png "Realtek")  

![ZyDAS](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/zydas.png "ZyDAS")

```bash
进入到"Device Drivers" -> "Multimedia support" 勾选：  
"Digital TV support"  
"Software defined radio support"  
"Media USB Adapters"  

在"Media USB Adapters"中 勾选：  
"Airspy"  
"HackRF"  
"Mirics MSi 2500"  
然后拉到最下面，取消勾选  "Autoselect ancillary drivers (tuners, sensors, i2c, spi, frontends)"  
取消勾选 "I2C Encoders, decoders, sensors and other helper chips" 内所有选项  
取消勾选 "Customize TV tuners" 内除了 "Rafael Micro R820T silicon tuner" 以外所有选项  
在 "Customise DVB Frontends" 内取消勾选除了：  
"Realtek RTL2830 DVB-T"  
"Realtek RTL2832 DVB-T"  
"Realtek RTL2832 SDR"  
以外所有的选项  
然后返回主菜单
```
如图所示  
![Multimedia_support](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/multimedia.png "Multimedia_support")  

![Media_usb](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/media_usb.png "Media_usb")  

![unselect](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/unselect.png "unselect")  

![I2C_EN](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/i2c_encoder.png "I2C_EN")  

![Custom_TV](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/custom_tv.png "Custom_TV")  
![Custom_DVB](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/custom_dvb.png "Custom_DVB")

```bash
进入到"Device Drivers" -> "HID support" 勾选：  
"Battery level reporting for HID devices"  
"/dev/hidraw raw HID device support"  
"User-space I/O driver support for HID subsystem"  
"Generic HID driver"  
勾选"Special HID drivers"  "USB HID support"  "HID over I2C transport layer"  内所有选项
然后返回上一级菜单
```
如图所示
![HID_support](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/hid_support.png "HID_support")  
![Special_HID](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/special_hid.png "Special_HID")  
![USB_HID](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/usb_hid.png "USB_HID")  

![I2C_HID](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/i2c_hid.png "I2C_HID")

```bash
接着进入到"Device Drivers" -> "USB support"  勾选：  
"Support for Host-side USB"  
"OTG support"  
"USB Modem (CDC ACM) support"  
"USB Wireless Device Management support"  
"USB Mass Storage support"  
"USB Serial Converter support"  

在"USB Serial Converter support" 中勾选：  
"USB Serial Console device support"  
"USB Generic Serial Drive"  
"USB Serial Simple Drive"  
"USB Winchiphead CH341 Single Port Serial Driver"  
"USB CP210x family of UART Bridge Controllers"  
"USB FTDI Single Port Serial Driver"  
"USB Prolific 2303 Single Port Serial Driver"  

在"USB Gadget Support"中勾选：  
"USB functions configurable through configfs"  
"Generic serial bulk in/out"  
"Abstract Control Model (CDC ACM)"  
"Object Exchange Model (CDC OBEX)"  
"Network Control Model (CDC NCM)"  
"Ethernet Control Model (CDC ECM)"  
"Ethernet Control Model (CDC ECM) subset"  
"QCRNDIS"  
"RNDIS"  
"RMNET_BAM"  
"Ethernet Emulation Model (EEM)"  
"Mass storage"  
"Function filesystem (FunctionFS)"  
"MTP gadget"  
"PTP gadget"  
"Accessory gadget"  
"Audio Source gadget"  
"Uevent notification of Gadget state"  
"MIDI function"  
"HID function"  
"USB Diag function"  
"USB Serial Character function"  
"USB CCID function"  
"USB QDSS function"  
接着返回主菜单，退出并保存配置
```
如图所示
![USB_support](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/usb_support.png "USB_support")  

![USB_Serial](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/usb_serial.png "USB_Serial")  

![USB_Gadget](https://image-ghfh.oss-cn-beijing.aliyuncs.com/nethunter/usb_gadget.png "USB_Gadget")


#### 保存配置

```bash
make ${args} savedefconfig
```



#### 编译内核
```bash
make ${args} 2>&1 | tee kernel.log
```
#### 编译内核模块
```bash
make ${args} INSTALL_MOD_PATH="." INSTALL_MOD_STRIP=1 modules_install
```


## 0x07 构建NetHunter-Kernel-Installer内核包
#### 下载Kali官方构建脚本
```bash
git clone https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-project /root/kali-nethunter-project --depth=1
```
#### 编辑机型列表
```bash
mkdir -p /root/kali-nethunter-project/nethunter-installer/devices/  
touch /root/kali-nethunter-project/nethunter-installer/devices/devices.cfg  
vim /root/kali-nethunter-project/nethunter-installer/devices/devices.cfg  
```
按照[官方教程](https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-devices#how-to-add-a-newunsupported-device)，添加以下内容并保存
```bash
# Xiaomi Redmi4X for crDroid Android 11  
[santoni]  
author = "DroidKali"  
arch = arm64  
version = "v1.0"  
flasher = anykernel  
modules = 1  
slot_device = 0  
block = /dev/block/bootdevice/by-name/boot  
devicenames = santoni,Redmi4x  
```
#### 创建机型对应的文件夹
```bash
mkdir -p /root/kali-nethunter-project/nethunter-installer/devices/eleven/santoni/modules/system/lib/modules  
```
#### 复制所需要的文件
```bash
cp out/arch/arm64/boot/Image.gz-dtb /root/kali-nethunter-project/nethunter-installer/devices/eleven/santoni  
rm -rf out/lib/modules/${make kernelversion}-NetHunter/source  
rm -rf out/lib/modules/${make kernelversion}-NetHunter/build  
cp -r out/lib/modules/${make kernelversion}-NetHunter /root/kali-nethunter-project/nethunter-installer/devices/eleven/santoni/modules/system/lib/modules/  
```
#### 生成NetHunter-Kernnel-Installer安装包
```bash
cd /root/kali-nethunter-project/nethunter-installer/  
python3 build.py -d santoni --eleven --kernel
```

## 0x08 下载ADB-FASTBOOT工具包

```bash
wget https://dl.google.com/android/repository/platform-tools-latest-linux.zip
unzip paltform-tools-latest-linux.zip -d /usr/share/
echo '''export PATH="/usr/share/platform-tools:$PATH"''' > /root/.zshrc
source /root/.zshrc
rm -rf platform-tools-latest-linux.zip
```

## 0x09 刷入内核安装包
#### 重启到Recovery模式
```bash
adb reboot recovery
```
#### 刷入内核刷机包
```bash
adb sideload kernel-nethunter-eleven-santoni-20210905_111235.zip
```

## 0x10 相关链接

[Kali NetHunter | Kali Linux Documentation](https://www.kali.org/docs/nethunter/)  

[NetHunter gitlab repository](https://gitlab.com/kalilinux/nethunter/)  

["黑客手机"免费送-知乎专栏](https://zhuanlan.zhihu.com/p/24946336)  

[跟我把Kali NetHunter编译至任意手机](https://www.vuln.cn/7086)  

[Building a Kernel for the Razor Phone 2 - Live feed](https://forums.kali.org/showthread.php?44899-Building-a-kernel-for-the-Razor-Phone-2-Live-feed)  

[Information on Compiling Android Kernels with Clang](https://github.com/nathanchance/android-kernel-clang)  

[[内核向] - 交叉编译器的选择](https://www.akr-developers.com/d/129)


