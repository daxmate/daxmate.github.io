---
hidden: true
layout: single
title:  "腾讯云自建云梯"
date:   2021-11-21
categories: "IT"
typora-root-url: ../../bestdax.github.io
---

周五，听大学同学说起用腾讯云自建云梯的事情，然后我就来劲了。现在的VPN经常超流量，用起来有点不带劲。在油管上看了一个视频后，按照里面的操作，最终搞定了，34元钱一个月，2T的流量，简单太爽了，不过不知道会不会给封。

简单记录一下过程，备忘。

1. 在腾讯购买轻量应用服务器
   
   <img src="/assets/images/image-20211121155344775.png" alt="image-20211121155344775" style="zoom:50%;" />

几个重点的地方

<img src="/assets/images/image-20211121155357378.png" alt="image-20211121155357378" style="zoom:50%;" />

现在最便宜的那个24元/月的已经买不到了，我买的是34元的，还可以，毕竟有两个T的流量呢。

2. 安装v2ray和v2-ui
   
   1. 点击登录
      
      <img src="/assets/images/image-20211121165306066.png" alt="image-20211121165306066" style="zoom:50%;" />
   
   2. 在出来的终端界面里面输入`sudo -i`
      
      <img src="/assets/images/image-20211121155823777.png" alt="image-20211121155823777" style="zoom:50%;" />
   
   3. 然后再复制以下命令到终端里面：
      
      ```bash
      bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)
      ```
      
      这个会安装v2ray和x-ui到服务器上。那个视频里面提供的那个地址已经失效了，好不容易找到这个。
   
   4. 然后在服务器的防火墙里面放行所有的tcp和udp的连接。
      
      <img src="/assets/images/image-20211121160919586.png" alt="image-20211121160919586" style="zoom:50%;" />

3. 设置VPN
   
   a. 复制ip地址
   
   <img src="/assets/images/image-20211121160523486.png" alt="image-20211121160523486" style="zoom:50%;" />
   
   b. 在浏览器里面输入复制的ip地址后面再加上`:54321`
   
   c. 会看到如下界面
   
   <img src="/assets/images/image-20211121160644420.png" alt="image-20211121160644420" style="zoom:50%;" />
   
   用户名和密码都是`admin`

​        d. 登录后点`击入站列表`

<img src="/assets/images/image-20211121164653723.png" alt="image-20211121164653723" style="zoom:50%;" />

​        e. 点击加号添加一个入口

<img src="/assets/images/image-20211121164852863.png" alt="image-20211121164852863" style="zoom:50%;" />

​        f. 点击查看，然后可以复制链接。在移动或者桌面客户端导入即可。

<img src="/assets/images/image-20211121165055698.png" alt="image-20211121165055698" style="zoom:50%;" />

尽情享受几乎无限的流量吧，想想之前用的VPN一个月才25G，好爽!
