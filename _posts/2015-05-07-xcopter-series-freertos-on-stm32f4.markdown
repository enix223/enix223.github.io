---
layout: blog
title: "xCopter开发日记 - 移植FreeRTOS到STM32F4平台"
date: 2015-05-07 15:23:11 +0800
comments: true
categories: blog
---

本文阐述如何在STM32F4平台上，通过FreeRTOS实时操作系统开发应用。本文使用的是STM32F401-Discovery开发板。板上有4个LED灯，一个用户按键。4个按键分别为： 

* LED4
* LED3
* LED5
* LED6

主程序开始初始化4个LED灯，和设置按键为EXTI中断模式。程序新建3个任务，第一个任务完成开发板的初始化，包括LED灯初始化，用户按键的初始化。第二个任务是反转每一站灯的状态，并等待500ms，第三个任务是中断响应任务，把第二个任务的等待时间更改为100ms（此任务有最高的优先级）。

## 准备工作

### 开发用到的软件

1. [STM32F4Cube][STM32F4Cube] 由ST公司提供给用户的硬件抽象层(HAL)，和stm32f4所有开发板对应的例子和ARM公司的DSP库，非常实用。  
2. [ST-LINK][ST-LINK] ST-LINK的WINDOWS USB驱动
3. FreeRTOS 源代码
3. IAR 7.2

### 需要用到的硬件

1. STM32F401-Discovery 开发板，芯片为STM32F407VG。TB有售。

## IAR创建项目

详细的设置，请参考另一篇文章，[xCopter开发日记 - STM32F4 LED example][post1]。创建项目完成后，目录结构如下：

![stm32f4-1-1]({{site.url}}/img/stm32f4/2-1-1.png)


[STM32F4Cube]: http://www.st.com/st-web-ui/static/active/en/st_prod_software_internet/resource/technical/software/firmware/stm32cubef4.zip
[ST-LINK]: http://www.st.com/st-web-ui/static/active/en/st_prod_software_internet/resource/technical/software/driver/st-link_v2_usbdriver.zip
[post1]: /blog/2015/05/04/xcopter-stm32f4-led-example