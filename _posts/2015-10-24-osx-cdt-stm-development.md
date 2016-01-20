---
layout: blog
categories: blog
published: true
title: 基于osx＋eclipse cdt + arm-gcc 开发STM32
---
**1\. 在内存运行代码**

  

步骤：

  

1.1 在项目的预编译设置项中，加入symbol: VECT_TAB_SRAM。这个预编译symbol告诉编译器，把vector
table放置于内存地址。详细代码在system_stm32f4xx.c

  

**void** **SystemInit**(**void**)

{

  ...

  /* Configure the Vector Table location add offset address
------------------*/  
**#ifdef** VECT_TAB_SRAM  
  SCB-&gt;VTOR = SRAM_BASE | VECT_TAB_OFFSET; /* Vector Table Relocation in
Internal SRAM */  
**#else**  
  SCB-&gt;VTOR = FLASH_BASE | VECT_TAB_OFFSET; /* Vector Table Relocation in
Internal FLASH */  
**#endif**

}

**  
**

1.2 修改ldscripts/mem.ld

  

MEMORY  
{  
  RAM (xrw) : ORIGIN = 0x20010001, LENGTH = 64K  
  CCMRAM (xrw) : ORIGIN = 0x10000000, LENGTH = 64K  
  FLASH (rx) : ORIGIN = 0x20000000, LENGTH = 64K  
  FLASHB1 (rx) : ORIGIN = 0x00000000, LENGTH = 0  
  EXTMEMB0 (rx) : ORIGIN = 0x00000000, LENGTH = 0  
  EXTMEMB1 (rx) : ORIGIN = 0x00000000, LENGTH = 0  
  EXTMEMB2 (rx) : ORIGIN = 0x00000000, LENGTH = 0  
  EXTMEMB3 (rx) : ORIGIN = 0x00000000, LENGTH = 0  
  MEMORY_ARRAY (xrw)  : ORIGIN = 0x20002000, LENGTH = 32

}

  

这个脚本是在编译链接阶段，告诉linker，使用什么地址作为ram地址，使用哪些地址作为flash。这里有一点很重要，这个设置必须和前面的systemini
t函数设置相对应，**flash的起始地址必须和SCB-&gt;VTOR的地址一致**，否则程序在运行后，无法配置正确的中断向量表，导致中断函数无法调用。

  

1.3 修改ldscript/sections.ld

  

因为通过cdt创建project的时候，默认是使用uos++，所以sections.ld文件也是专门为uos++而写，如果需要用
startup_stm32f407xx.s 作为默认的引导代码，就必须使用stm32的 LinkerScript.ld 作linker的script。

  

1.4 如果使用stlink作为调试工具时，需要使用openocd插件来调试

  

使用默认配置即可，具体配置如下图，但是有一点特别要注意的是，当你需要在内存调试的时候，必须在Runtime Options，Debug in
RAM处打勾，否则每次启动调试后，PC寄存器都会指向flash入口，而不是RAM的入口。

  

经过查看gdb的调试日志后发现，如果debug in ram不打勾的话，gdb每次load完elf文件后，都会发送monitor reset 和
monitor halt命令，就是这个命令导致无法在内存调试。所以务必要注意。

![22cebd5b5c9fd9baf26ce51f43264f1f](/media/22cebd5b5c9fd9baf26ce51f43264f1f.png)

  

遇到的问题

  

新加入的文件夹中的文件，在编译过程已经把include的路径加入到c compiler的includes列表中，但是编译过程仍然提示Could not
find ‘XXXX’ in index

  

这个问题折腾了我好久，最后发现是通过右击项目，选择Index下的Rebuild，等待rebuild完成后，再重新编译项目，即可解决问题。

  

  

