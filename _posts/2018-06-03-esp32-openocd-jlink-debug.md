---
published: false
---
## 前言

最近买了一块esp32，打算在ubuntu 16下试试做嵌入式的开发，本文记录一下如何快速环境搭建开发环境。

## 材料：

1. esp32 （安信可）模块 * 1
2. SEGEER JLink调试器 * 1
3. USB转串口（用于打印USART信息）
4. 自制3.3v usb供电接口

废话少说，开干。

首先，下载开发需要的软件，具体步骤可以查看[esp-idf.readthedocs.io](http://esp-idf.readthedocs.io/en/latest/get-started/linux-setup.html)

1. 前置条件

```
sudo apt-get install gcc git wget make libncurses-dev flex bison gperf python python-serial
```

2. 下载编译工具toolchain

for 64-bit Linux:

https://dl.espressif.com/dl/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz

3. 解压刚才下载的toolchain

mkdir -p ~/esp32
tar -xzf ~/Downloads/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz -C ~/esp32

4. 配置环境变量

```
export PATH="$PATH:$HOME/esp32/xtensa-esp32-elf/bin"
```

5. 下载jlink的驱动

jlink的驱动可以到[segger](https://www.segger.com/downloads/jlink/)的官网下载，此处不赘述。

下载完成后，安装该deb包

```
sudo dpkg -i JLink_Linux_V632d_x86_64.deb
```

6. 下载esp32的openocd

openocd-esp32具体可以到[github下载](https://github.com/espressif/openocd-esp32/releases)

下载并解压到~/esp32

```
tar -xvf ~/Downloads/openocd-esp32-linux64-0.10.0-esp32-20180525.tar.gz -C ~/esp32/
```

7. 最重要的一步，就是接线，如下图：

此处参看了openocd-esp32中一个issue的图：


![](https://user-images.githubusercontent.com/9928032/29207336-62fc60ac-7eb8-11e7-828f-58c26e62b3d4.png)

8. 最终的接线如下图：

![WechatIMG1.jpeg]({{site.baseurl}}/_posts/WechatIMG1.jpeg)

一切准备就绪，接下来开始调试。

1. 打开openocd server

```
cd ~/esp32/openocd-esp32
bin/openocd -s share/openocd/scripts -f interface/jlink.cfg -f board/esp-wroom-32.cfg
```

2. 创建gdbinit

```
cd ~/esp32/hello-world
vi gdbinit
```

```
target remote :3333
mon reset halt
thb app_main
x $a1=0
c
```

3. 手上的esp32，已经烧录了hello-world程序

```
cd ~/esp32/hello_world/
xtensa-esp32-elf-gdb -x gdbinit build/hello-world.elf
```

gdb启动成功，可以开始gdb调试程序了。

