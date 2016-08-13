---
layout: blog
categories: blog
published: true
title: 如何通过wireshark抓取iphone的HTPP请求
---
本文介绍如何方便简洁的使用wireshark抓取iphone的HTTP请求。本方法无需iphone越狱，无需在iphone上配置代理。简单快捷，容易上手。本
方法是在Macbook上运行。如果你使用的是windows，请绕道。

  

准备材料

1）wireshark最新版本，可以到官网下载

2）iphone 一部，并需要通过usb数据线连接到macbook上

3）itunes

  

**_获取iphone的UUID_**

  

首先打开itunes, 并保持iphone与macbook的连接。查看信息页面， 并在序列号（serial number）处单击，然后就可以看到UUID

![08b1e0269c7ce27c19e71d556f3081e0](/media/08b1e0269c7ce27c19e71d556f3081e0.png)

  

此时，可以右击，并复制

![034c4fe93a791c8c2bfdc7f29b3f2290](/media/034c4fe93a791c8c2bfdc7f29b3f2290.png)

  

  

**创建Remote Virtual Interface**

  

此时，轮到主角出场，我们之所以可以捕获到iphone的包，这个命令功不可没。

  

废话少讲，打开terminal终端

  

输入如下命令:

  

     rvictl -s [Your iphone UUID]

  

![8ba4930e614982db2a1c9936f2210fd8](/media/8ba4930e614982db2a1c9936f2210fd8.png)

  

这样，系统就会为iphone创建一个remote virtual
interface，rvi就相当于一个终端（我是这么理解的），我们可以通过wireshark监听这个终端，然后，就可以实现iphone流量的捕获。

  

**开始监听HTTP 请求**

  

打开wireshark，然后就可以看到一个可用的监听interfaces列表，我们此时就可以选择刚才创建的rvi0。magic就都在这里。

![17f05f75cce30151b0c01b82e0edc6f9](/media/17f05f75cce30151b0c01b82e0edc6f9.png)

  

filter规则，我们选择http，回车后，随便打开一个需要联网的app，那么这个app的所有http请求都一览无余的暴露在你面前了。

![2bbde779a5c02a456196c72ac1b93330](/media/2bbde779a5c02a456196c72ac1b93330.png)

  

简单吧？你也来动手试试

  

Happy coding: )

  

_参考文章_

<http://useyourloaf.com/blog/remote-packet-capture-for-ios-devices/>

  

