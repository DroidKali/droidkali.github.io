# wifi-deauth-attack初探

### 0x00 前言
前段时间玩过一阵子ESP8266 WiFi Deauth攻击，闲的无聊来水一波文章！
<!--more-->
### 0x01 背景
我们所使用的802.11 WiFi协议包含了一个Deauthentication(解除身份验证)的特性，其作用是为了将用户从无线网络中分离。而黑客们正是利用了WiFi的这个特性，使得黑客可以随时使用无线AP的伪造源地址，向无线AP设备发送一个或多个Deauthentication攻击数据包，从而使得合法客户端设备从无线AP设备上掉线。简单来说就是黑客可以随时让你的无线终端设备(例如智能手机，平板，笔记本)等设备从WiFi中掉线然后再也无法连接，除非黑客停止攻击。

该协议不需要对Deauthentication攻击框架进行加密，甚至是建立会话。该问题在[802.11w-2009](https://www.networkworld.com/article/2312251/network-security/how-802-11w-will-improve-wireless-security.html)中有提及解决，但是几乎所有的无线AP设备厂商默认都禁用了这个属性，所以直到今天这个漏洞仍影响着全球近九成以上的无线AP设备。

### 0x02 材料准备
>·ESP8266 无线模块一块 (某宝自行搜索)
>·一条Micro USB数据线 (有条件的话用USB转TTL也行)
>·电脑一台 

### 0x03 环境准备
>·Arduino IDE [传送门](https://www.arduino.cc/en/Main/Software?setlang=cn)
>·NodeMcu Flasher [传送门](https://github.com/nodemcu/nodemcu-flasher)

### 0x03 刷入固件
首先把我们的ESP8266连接到电脑上，安装对应的驱动
>·CH340串口驱动 [传送门](http://www.wch.cn/download/CH341SER_ZIP.html)
>·CP2102串口驱动 [传送门](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

接着我们去下载ESP8266专用的WiFi Deauth攻击固件，固件地址:
>https://github.com/spacehuhn/esp8266_deauther/releases

这里需要注意的是固件的选择要根据你的模块Flash大小来选择，ESP-12的Flash大小为4MB，ESP-07的为1MB,ESP-01的为512KB/1MB。详细说明可以参考[官方wiki](https://github.com/spacehuhn/esp8266_deauther/wiki/Installation#flash-size)
接下来我们开始刷入固件，打开NodeMcu Flasher，点击Config选项卡，选择你的固件，接着点击Advanced选项卡，在Baudrate(波特率)处选择115200，Flash size根据你的板子Flash大小来选择，我这里是ESP-12所以选择4MB，Flash speed选择80MHz，SPI mode选择DIO,最后我们点击Operation选项卡，在COM port处选择你的板子的端口，可以在windows的设备管理那里看到，然后点击Flash(F)按钮，接着就是等待固件烧录完成了。固件烧录完成后NodeMcu Flasher左下角的NODEMCU TEAM图标会变为绿色。

### 0x04 攻击实战
接着我们打开手机WiFi设置，能搜索到一个SSID名为***pwned***的WiFi，这个就是我们烧录好固件之后生成的，密码为***deauther***，连接上去。接着我们打开手机浏览器，在地址栏输入***http://192.168.4.1***进入deauther的后台管理界面，如图:![esp8266-scan](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/esp8266-scan.png)由于最新***v2.1.0***版的固件自带了一个开机自动扫描AP的脚本，所以就不需要我们连上后再手动扫描了，当然如果你想得到更准确的结果的话你也可以重新扫描。扫描周围的无线热点后，我们选择一个目标开始攻击，这里我选择的FAST_4E14这个热点，然后选择攻击界面，如图:![esp8266-attack](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/esp8266-attack.png)选择Deauth攻击，接着刷新界面就能看到攻击效果了![esp8266-attacking](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/esp8266-attacking.png)此时被攻击的无线AP上所有已连接的设备会全部掉线直至我们停止攻击。

### 0x05 安全建议 
由于WiFi Deauth攻击是802.11协议上的一个缺陷造成的，所以目前并没有什么有效的防御措施，只能等无线联盟更新协议来弥补这个缺憾。个人的话尽量按照以下几点来做吧
>·路由设置白名单或MAC地址绑定
>·开启访客WiFi
>·隐藏SSID
>·不用WiFi时尽量关闭路由

### 0x06 补充说明 
可能大家对ESP8266 Deauther的最后一个攻击模式不太清楚，这里引用天马安全团队杨大佬的文章来解释一下:我们所使用的无线客户端设备在连接无线AP时会采用两种扫描方法，主动扫描和被动扫描。在主动扫描中，客户端发送Probe Request,接收由AP发回的Probe Response。在被动扫描中，客户端在每个频道监听AP周期性发送的Beacon无线数据帧。之后是认证(Authentication)和连接(Association)过程。
说白了就是我们所使用的无线客户端(例如手机，平板，笔记本等设备)在连接WiFi的时候也会向外广播一个无线数据帧，这个无线数据帧就是**Probe Request**帧,这个数据帧里面包含了我们的无线设备曾经连过哪些WiFi，它们的SSID是什么，MAC地址等等，而无线AP接收到这个数据帧之后也会进行回复响应，回复给无线客户端设备另外一个数据帧-**Probe Response**帧，这样就可以做到无线客户端设备和无线AP设备之间的快速连接。
但是这个特性也可以被黑客给利用，黑客可以监听空气中的无线数据包，分析里面的内容，可以抓到这个Probe Request帧，得到里面包含的无线客户端设备曾经连过的WiFi热点的SSID,MAC地址，从而伪造一个同名同MAC地址的无线接入点来实施钓鱼攻击。这就是后来的Karma和Mana攻击的原理！具体的内容可以看看FreeBuf上的相关文章，我这个菜鸡就不在这里班门弄斧了！

>[聊聊WiFi Hacks：为何你的Karma攻击不好使了](https://www.freebuf.com/articles/wireless/145259.html)
>
>[WiFi Pineapple的Karma攻击与原理探究](https://www.freebuf.com/column/144882.html)
>
>[WiFi Pineapple的Karma攻击与原理探究](https://www.freebuf.com/articles/77055.html)
>
>[主动触发被动模式从而挟持无线客户端 – Passive Karma Attack](https://www.freebuf.com/articles/wireless/44378.html)

