---
title: '基于Vscode搭建Puya PY32F00x系列单片机开发环境'
date: 2025-11-29T09:53:39+08:00
draft: false
---

# 初衷

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_jpg%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaC4y6FFy5K1VRIaQTZ6p9L3uHtStGte81ictc0icTy81iacgiaJXMahiame5Q%2F0%3Ffrom%3Dappmsg)

最近在某宝发现了一个性价比很高的单片机**普冉PY32F003**，可以替代ST和GD，芯片只需要**0.5**元左右，**32KB flash**，**4K RAM**，主频也有**24MHz**，性价比确实高。PY32F003对标STM32F0，也是基于Arm Cortex M0+架构，可以兼容STM32的API，对于了解STM32的同学来说，学习成本几乎为0。

既然PY32F003也是Cortex M0+架构，我们当然也可以使用Keil或者IAR环境，那为什么我还要尝试使用Vscode来搭建开发环境呢？主要的诉求是：
* 出于对软件只是产权的尊重，不想用盗版（我自己也开发软件，推己及人，尊重别人的劳动成果）
* 囊中羞涩，买不起Keil正版。当需要商业目的开发也不怕有知识产权风险。
* 希望使用ubuntu系统进行开发，因为主要的办公电脑是ubuntu + macos
* 希望使用熟悉的开发工具Vscode

所以本文将介绍基于github的`openPuya/PY32F0xx_Firmware`仓库 + VSCODE + EIDE搭建开发环境。

# 实验材料准备

1. PY32F003 Start kit开发板，可在某宝中搜索**py32f003开发板**，我购买的是**芯岭技术**的开发板(19.9元)。
2. DAPLINK烧录工具，在某宝中搜索**DAPLINK**，我购买的是信泰微的（9.8元）

# 开发环境搭建

Vscode这里不做过多的介绍，懂得都懂，借助Vscode强大的插件系统，几乎各种语言的开发都可以基于Vscode实现，当然嵌入式也不例外。

Vscode的下载请移步[vscode官网](https://code.visualstudio.com/)

## 获取PY32F0xx SDK

PY32的SDK可以从一个第三方的开源网站[OpenPuya](https://py32.org/) 中获取，该网站搜集了非常全面的普冉各个型号的MCU的资料及其SDK，本文介绍的PY32F003也可以从该网站获取，对应的GitHub地址为:

https://github.com/OpenPuya/PY32F0xx_Firmware

我们可以通过git工具下载sdk

```shell
cd ~/source
git clone https://github.com/OpenPuya/PY32F0xx_Firmware
```

## EIDE

EIDE是Embedded IDE的简称，EIDE可以提供开发`8051/STM8/Cortex-M/MIPS/RISC-V`芯片的开发环境，该插件支持Windows/Linux/Macos系统，我们可以在Vscode的插件市场找到该插件，找到该插件后点击安装按钮进行安装即可，如下图所示：

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLSxD3RM829M7Y2qrEEAa3j7gQ3RrRTpia3jJAgoZ0HAO9iaAJPbawtBR2H843icQeqLEiaVEkbzr4uuMA%2F0%3Ffrom%3Dappmsg)

安装好插件后，我们还需要安装dotnet环境（原来dotnet也可以在linux下运行），我们可以使用如下命令添加apt源：

```shell
curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | sudo gpg --dearmor -o /usr/share/keyrings/microsoft.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/ubuntu/$(shell lsb_release -rs)/prod noble main" | sudo tee /etc/apt/sources.list.d/microsoft-prod.list
```

执行安装dotnet运行环境

```
sudo apt update
sudo apt install -y dotnet-runtime-8.0
```

## 安装编译工具链

PY32F0xx sdk没有带编译工具链，我们需要自行安装arm的gnu编译工具，还有gdb（调试工具）

```shell
sudo apt install -y gcc-arm-none-eabi gdb-multiarch
```

## 安装pyocd

有了编译工具链和sdk，我们还差一个烧录工具，开源的烧录方案比较出名的就包括:

* openocd
* pyocd

PY32 sdk内包含了一个修改过的openocd压缩包，但是该压缩包是只面向windows的（只提供了exe文件），但是我是在ubuntu中开发，这个`openocd`无法使用。幸好EIDE支持`pyocd`，而`pyocd`支持py32几乎全系列的芯片，所以本文将使用`pyocd`作为芯片的烧写工具和调试工具。

我们可以使用系统自带的python和pip工具进行安装`pyocd`

```shell
pip install pyocd
```

安装pyocd完毕后，我们可以查询一下pyocd支持哪些py32芯片：

```shell
$ pyocd pack find py32
  Part           Vendor   Pack                Version   Installed  
-------------------------------------------------------------------
  ...
  PY32F003x4     Puya     Puya.PY32F0xx_DFP   1.2.8     True       
  PY32F003x6     Puya     Puya.PY32F0xx_DFP   1.2.8     True       
  PY32F003x7     Puya     Puya.PY32F0xx_DFP   1.2.8     True       
  PY32F003x8     Puya     Puya.PY32F0xx_DFP   1.2.8     True       
  ...
```

搜索返回了80几行，几乎支持所有的PY32系列单片机。我们找到我们开发板对应的那一款`PY32F003x6`，然后执行安装：

```shell
pyocd pack install PY32F003x6
```

安装完成后，pyocd就算配置完成了。关于pyocd的配置，这里需要感谢参考文章《[VScode调试开发PY32系列单片机](https://zhuanlan.zhihu.com/p/697267413)》的作者。

# 踏上开发之旅

万事俱备，只欠东风。开发环境已经搭建好，现在我们就开始使用py32 sdk来开发第一个应用。

## 使用EIDE打开项目

我们使用vscode打开前面通过git下载的`PY32F0xx_Firmware`仓库，项目结构大概是这样的：

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaC5iaLCpSfsko3VYZ8TRicavceZPXMMGm3ohabSd00Y4hupLYgC83KCpRw%2F0%3Ffrom%3Dappmsg)

我们需要打开EIDE界面，并打开相应的项目目录，此处我们以`GPIO_Toogle`示例程序作为例子：`Projects/PY32F003-STK/Example/GPIO/GPIO_Toggle/EIDE`

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCHR10dZHSvg0pzVKxsGKTLpJu9yWrTfBHh6SHYhqTOroFGrZapeiaVaQ%2F0%3Ffrom%3Dappmsg)

打开项目时，我们记得需要选择EIDE文件夹下面的`Project.code-workspace`文件

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCR8m6XDn3B7R05FibpOeo6pqDn7LFl43yGJeiaeDeVLevDYrSNP6ol1AQ%2F0%3Ffrom%3Dappmsg)

打开成功后，我们就可以看到EIDE的配置:

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCfrrJ8OmnUIPObSw5ib4yYgj14ruia6t1hyoVsV5ZpCNk80ZBIb69DkgQ%2F0%3Ffrom%3Dappmsg)

## 修复项目配置

因为PY32F0xx_Firmware应该是使用旧的EIDE版本创建，新版的EIDE会新建一个.eide文件夹，且使用的是yaml配置文件替代json配置文件，所以打开项目后，会生成一个新的文件夹，另外由于Ubuntu系统是文件名称大小写敏感，所以有些文件在项目打开后会出现了⚠️感叹号，表示无法找到该文件，例如`py32f0xx_hal_*`文件，我们需要手动修改文件的路径:

选择出现警告的文件，并右击，并选择"Modify File Path"
![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaChMHjV4YR74Iqpg6UbibxqdC1u1ic7FnnE7Fk5fzciaWm9HfOqvMib9E0GA%2F0%3Ffrom%3Dappmsg)

我们需要目录的大小写，例如`drivers`改成`Drivers`，`src`改成`Src`：

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCKHkDPOib4w9V2xlY0KUHtOAPfNDxuFmibT83s7fDic5TPPHEYuFWflALw%2F0%3Ffrom%3Dappmsg)

例如hal_cortex.c文件修改后的路径应该为：

```
../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_cortex.c
```

另外一种快速修改目录的方法是，直接修改`EIDE/.eide/eide.yml`文件

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCST9pFbCHqMXtkzXB1WibUyK4ClKmU0LibQ3GQFg20ND4aNuOqQbCWIHg%2F0%3Ffrom%3Dappmsg)

批量修改:

```yaml
    - name: PY32F0xx_HAL_Driver
      files:
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal.c
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_rcc.c
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_rcc_ex.c
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_flash.c
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_gpio.c
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_dma.c
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_pwr.c
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_uart.c
        - path: ../../../../../../Drivers/PY32F0xx_HAL_Driver/Src/py32f0xx_hal_cortex.c
      folders: []
```

同样的，其他配置中，如果出现小写的目录，都需要根据实际情况修改为跟目录名匹配:

* src -> Src
* drivers -> Drivers

最后确认修改后，我们重新打开项目，确认没有出现⚠️图标

## 修改烧写配置为pyocd

修复完路径问题后，我们需要把烧写工具改为pyocd，同样的，我们需要在EIDE中完成操作:

1. 点击Flasher Configurations右边的切换按钮：
![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCVtXu51xKUWK249WfZb5SNSW9xxw4VOYgpSibRWiaaRjHFOYbtYFR337g%2F0%3Ffrom%3Dappmsg)

2. 选择pyocd

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCcZ8rxyvMuWe4d7iaPzZyzCsVtBo3xvzBwTyzibacR2GhZKayVzgcPVWg%2F0%3Ffrom%3Dappmsg)

3. 修改Target name为`py32f003x6`

## 确保linker script正确

最后我们要确保linker script跟我们的开发板要对应，例如我购买的是py32f003 start kit开发板，对应的是`py32f003x6`，而sdk中使用的是py32f003x8.ld，二者在flash容量的大小和ram大小都不一样，我们要根据实际情况修改ld文件的flash与内存配置：

```
/* Specify the memory areas */
MEMORY
{
RAM (xrw)      : ORIGIN = 0x20000000, LENGTH = 4K
FLASH (rx)      : ORIGIN = 0x8000000, LENGTH = 32K
}
```
例如`py32f003x6`对应的是32K flash, 4K RAM，那么我们把ld文件关于ram和flash的配置修改正确，如上所示。

## 编译固件

完成了上述配置后，我们可以对项目进行编译，编译方法如下所示:

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCfGGHPFMNUbeibcaUia4b7DtXnfiaGrgqMQGDE6Pkibg816M8wnaiazz2MsQ%2F0%3Ffrom%3Dappmsg)

我们点击编译按钮，编译完成后，我们可以查看固件的大小:

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaC1plKeic6ibOTtQ6wIrDbBGo6Q9JHkESNoPrYcXP2GZdGstY4nkosFfdg%2F0%3Ffrom%3Dappmsg)

## 烧录固件

烧录固件，我使用的是DAP LINK烧录工具

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_jpg%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCtGQ96febBU1H8ibSFJwGxN3IlXU2JsovYYLY0rniczKyMpYnoqmMDgWg%2F0%3Ffrom%3Dappmsg)

烧录工具的接线如下所示：

```
DAP GND -- PY32F003 GND
DAP SCK -- PY32F003 PA14_SWCLK
DAP SWD -- PY32F003 PA13_SWIO
```

接好线后，我们可以点击EDIE Project工具的下载**烧录**按钮进行烧录，烧录成功的话，我们会在日志中看到类似如下的结果：

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaC51mYahzOfhB5QGs7rian3DSg6VlGJickSXxo5tyGMKrYBVn4iayJMDKIw%2F0%3Ffrom%3Dappmsg)

不出意外的话，我们就可以看到py32f003的绿色LED灯在间隔的闪烁：

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_jpg%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaClA8ChiapJrbSozwG3bZ7AyiaNvicRgYAB7wdyJxue6z46MF963CePYrGw%2F0%3Ffrom%3Dappmsg)

# 如何调试

当我们的程序出现错误时，我们希望能执行断点和单步调试，这里我们需要借助pyocd + gdb完成调试工作。

#### 启动gdbserver

pyocd提供一个gdbserver功能，我们需要使用DAPLINK连接我们的开发板，并通过如下命令启动gdbserver:

打开vscode的一个终端，并输入如下命令：

```shell
pyocd gdbserver -t py32f003x6
```

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaC8MpeTmXECvNM7Gduk07OXDNibxQ2aculowlNkCr9mb46Vel9kXoFk7A%2F0%3Ffrom%3Dappmsg)

启动完毕后，我们可以看到如上述所示的日志，gdbserver监听的是3333端口。

#### gdb调试

熟悉linux程序开发的同学，对gdb肯定不陌生，这里我们使用gdb来完成调试工作，我们需要传递编译后的elf文件给gdb:

我们需要再打开一个vscode终端，并输入如下命令：

```shell
gdb-multiarch Projects/PY32F003-STK/Example/GPIO/GPIO_Toggle/EIDE/build/Project/Project.elf
```

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F9nAWafFNqLTSddR7zyTZ0XNxPzpiarRaCXADfjqicSneXmVppSQTvLdlJzCmjwKwBbfOVichtY1F44iaUxpUkU8trw%2F0%3Ffrom%3Dappmsg)

进入gdb后，我们可以使用如下命令来连接上一步打开的gdbserver：

* 连接gdbserver

   ```
   target remote :3333
   ```

* 在main.c文件的62行添加断点
   ```
   b main.c:62
   ```

* 列出现在的断点
   ```
   info b
   ```

* 重置mcu
   ```
   monitor reset halt
   ```

* 继续运行
   ```
   c
   ```

* 打印变量的值
   ```
   p var1
   ```
