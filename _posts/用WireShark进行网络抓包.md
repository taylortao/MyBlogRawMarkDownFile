---
title: 用WireShark进行网络抓包
date: 2011-12-09 10:17:00
categories:
 - IT Technology
tags:
 - WireShark
 - Network
toc: false
---
这是本菜鸟参考网上一篇教程进行的网络抓包，记录一下过程啦。

### 步骤
- 第一步：安装WireShark1.6.4的同时，依赖安装了winPCap，winPCap是用于网络封包抓取的一套工具，可适用于32位的操作平台上解析网络封包。
<!-- more -->

- 第二步：打开WireShark开始抓包。然后我们打开人人网 主页。登陆操作，登陆后，停止抓包。
![wireshark sniffer](step2.gif)

- 第三步：在cmd中使用ipconfig 得到本机ip地址，然后使用ping www.renren.com 后得到人人网的ip地址。

- 第四步：在filter框中，输入ip.src == 222.199.225.21 && http && ip.dst ==58.205.210.234 根据源IP、目的IP和http 进行筛选
![filter from result](step41.gif)

可以看到其中有一个POST请求，是提交表单的操作。
![result](step42.gif)

可以看到我的人人账号和密码了。

 

### 结论
基于Http的网络传输协议是以明文传输的，因此是不安全的。尽管基于Https的网络传输是加密的，安全的，但是据说Https服务器的承载能力相比Http服务器很低。
