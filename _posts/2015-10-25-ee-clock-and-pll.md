---
layout: blog
categories: blog
published: true
title: 时钟，分频与倍频
---
**1\. 时钟的作用**

  

     数字电路元件都需要时钟信号，完成计时，同步，计数和时序控制功能。

  

**1.1 时钟信号源 － 晶振**

  

水晶有一个特性，就是给它通电的时候，它会产生机械震荡，反之，当给它外力的时候，它就会产生电。这称为压电效应。而它的振动频率非常稳定，所以它被广泛应用于时钟，
手表，计算机等。

  

**1.2 系统主频，内部时钟频率**

  

晶振的频率一般是12MHz - 20MHz，而对应的cpu频率则非常的高，如600MHz。当需要把较低频率的时钟信号增加到cpu可以接受的频率，就称为倍频。

  

锁相环电路(PLL), 这种电路可以输出系统时钟固定倍数的频率。一个系统中，出于不同的要求，需要以不同的频率运行，低速运算可以省电，而高速运算则增加了功耗。

  

**1.3 设备频率**

  

再一个soc上，除了cpu，其他设备通过AHB与cpu相连，但cpu的频率相对于外设来说又太高，这时，时钟会提供两种较低频率的时钟信号，也就是HCLK和PC
LK这两种信号。

当cpu上电时，PLL未启动，则FCLK = Fin = 外部时钟频率（12MHz）

  * FCLK － cpu提供的时钟信号
  * HCLK －为AHB高速外设提供的时钟信号，例如内存控制器，DMA，LCD控制器
  * PCLK －为APB低速外设提供的时钟信号，例如UART, IIS, I2C, SDI/MMC, GPIO, RTC, SPI

  

**1.4 divider 分频器**

  

但对于一些低功耗的抹开，PCLK仍然过高。这时需要用分频器将频率进一步降低。

  

**1.5 Prescaler 预分频因子**

  

有一些模块，需要通过编程来实现分频的比率，而prescaler就是用于实现这个目的。假如输入频率为fin，输出为fout，那么二者满足以下关系：

  

_fout = fin / (prescaler + 1)_

或者

  

_fout = fin / (prescaler + 1)/divider_

  

**2\. STM32F4的时钟源**

  

  * HSE － 4～26MHz，可作为RTC时钟源
  * HSI - 16MHz
  * LSE － 32KHz，专供RTC使用
  * LSI - 32KHz，专供IWDG和RTC使用
  * PLL － 由HSE或者HSI提供，有两个输出：第一个最高可输出168MHz，而第二个输出供usb和sdio作为时钟源。
  * CSS － 当CSS启用后，HSE出现错误后，自动切换至HSI
  * MCO1 － 输出HSI, HSE, LSE, PLL时钟 (PA8)
  * MCO2 － 输出HSE, PLL, SYSCLK, PLLI2S (PC9)

  

STM32F407时钟

  

![75780162cbf0bcd84f1aa8b5836828b3](/media/75780162cbf0bcd84f1aa8b5836828b3.png)

  

  

AHB最高频率为168MHz (HCLK)

APB1 - 低速APB，最高频率为42MHz (PCLK1)

APB2 - 高速APB，最高频率为84MHz (PCLK2)

  

所有外设的时钟都是从SYSCLK系统时钟衍生而来，除了：

  * USB OTG FS 时钟（48MHz）
  * I2C，由PLL提供，或者由外部时钟提供
  * USB OTG HS时钟（60MHz）
  * ethernet MAC 时钟，有外部时钟提供。

