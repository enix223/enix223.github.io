---
layout: blog
title: "Access Google via Redhat Openshift"
date: 2015-09-17 00:19:44 +0800
comments: true
categories: blog
---

Openshift介绍
----------------

Openshift是Redhat公司旗下的PaaS的一个重要产品。相对与国内的PaaS产品，它的价格优势相当明显，Openshift分3个等级的账号：

1. Free账号：可以创建3个应用(gear)，24小时无访问应用自动变为IDLE状态。1GB的本地存储
2. Bronze账号：可以创建最多16个应用(gear)，永不停机，每个应用1GB空间，超出部分按1$ per month计算。支持SSL。
3. Silver账号：每月20$，永不停机，最多可创建多于16个账号，6GB附送空间，多出部分按1 GB per month收取。

跟国内的新浪SAE相比，无限流量是个非常大的优势，新浪SAE分钟流量超额，则会禁用部分功能，而且新浪SAE是沙箱结构，也就是说，应用依赖的环境是非原生的语言环境，例如禁用了多线程，禁用了本地文件读写等。致使很多本可以在本地开发环境畅通无阻的运行的应用，迁移至云平台后，无法运行，给开发人员调试带来极大的麻烦。

Openshift与百度BAE相比，最吸引人的地方就是在无需付费的情况下，即可创建应用。百度BAE相对于新浪SAE，优势在于原生的语言运行环境。开发人员面对的是非修改过的语言环境，对于运行调试都带来极大的方便。

同时Openshift也各种开发语言环境，例如JAVA, PHP, PYTHON, RUBY。对于热衷于RAILS的用户，国内的PaaS大多不支持。所以Openshift是一个不错的选择。


注册Openshift账号
-----------------

1. 打开[openshift][openshift]官网，因GW的原因，你懂得，所以需要用https访问。

![vpn]({{site.url}}/img/vpn/1.png)

2. 填写注册信息

![vpn]({{site.url}}/img/vpn/2.png)

3. 注册完成后，登陆，并创建1个应用。对于创建那个应用，可随意。我选择的是django 1.6

![vpn]({{site.url}}/img/vpn/3.png)

4. 在PUBLIC URL填写应用名字，可随意，只要不要与现有的应用名称重复即可。填写完毕后，点击下方的create application

![vpn]({{site.url}}/img/vpn/4.png)


WINDOWS用户
-------------------

### 安装msysgit

对于大部分Geek来说，无人不知git，而且后面也需要用到ssh，而msysgit出了包含git所需的命令外，也包括了大量linux的命令，例如后面用到的ssh。所以这里需要安装msysgit。

下载[msysgit][msysgit]，并安装。安装步骤就不再赘述。默认选项安装即可。


### 安装ruby

Openshift的本地开发环境需要用到ruby，而且其控制命令rhc是通过ruby来编写，所以这里同样需要安装ruby

下载[ruby 2.2.2][ruby]，同样默认安装

LINUX/MAC OS 用户
-------------------

因LINUX系统和MAC OS系统均已内置了SSH，git，所以无需另外安装特殊的软件，而ruby却因不同的linux发布版本，有的自带了，有的却没有，所以按需安装

DEBIAN系

    $ sudo apt-get install ruby-full

REDHAT系

    $ sudo yum install ruby

MAC系

    $ brew install ruby


安装rhc
----------------

在安装后ruby环境后，接下来我们需要安装rhc，Openshift的本地开发环境。

    $ gem install rhc

安装完成后，我们就可开始Geek之旅。


生成Openshift的ssh key
------------------------------

### 初始化环境

    $ rhc setup

如下截图填写信息，用户名和密码为刚刚在Openshift注册的用户名和密码：

![vpn]({{site.url}}/img/vpn/5.png)   


开始极客之旅
---------------

按如下步骤，查看Openshift新创建的应用的访问地址：

![vpn]({{site.url}}/img/vpn/6.png) 

![vpn]({{site.url}}/img/vpn/7.png)  

复制红色圈圈标注的ssh地址，然后打开Git bash，粘贴刚才复制的命令，并在ssh后加入参数-D，和端口号12345：

    $ ssh -D 12345 xxxxxx@appName-yourname.rhcloud.com


待连接成功后，恭喜你，你的私人代理服务器已架设成功！！

![vpn]({{site.url}}/img/vpn/8.png) 


最后一步，当然就是使用你的私人代理，畅想无阻隔的冲浪，这里必须注意的是，只需要填写SOCK，因为用的是SSH TUNNEL

![vpn]({{site.url}}/img/vpn/9.png) 

大功告成， 洗洗睡

![vpn]({{site.url}}/img/vpn/10.png) 


[openshift]: https://www.openshift.com
[msysgit]: http://dlsw.baidu.com/sw-search-sp/soft/4e/30195/Git_V2.5.1_64_bit_setup.1441791170.exe
[ruby]: http://dlsw.baidu.com/sw-search-sp/soft/ff/22711/rubyinstaller_V2.2.2.95_setup.1439890355.exe