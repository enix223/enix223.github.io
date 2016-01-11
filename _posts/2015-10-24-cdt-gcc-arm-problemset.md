---
layout: blog
categories: blog
published: true
title: CDT gcc-arm 插件相关问题集锦
---
  * 为何编译不自动编译asm文件？例如项目中，有一个asm启动代码文件startup_stm32f407xx.s，为何该文件不被编译？

答案：经过查阅gcc-arm的官方网站，编译器只会识别以 xxx.S结尾的文件，而本项目中，该文件的文件名是小写的后缀startup_stm32f407xx
.s，所以编译器会跳过该文件，不进行编译。我们只需要把文件后缀改为大写的.S后缀即可startup_stm32f407xx.S

  1. Build project的时候，最好链接阶段，遇到 undefined reference to `_init`错误

/Users/enix/Public/gcc-arm-none-eabi-4_9-2015q2/bin/../lib/gcc/arm-none-
eabi/4.9.3/../../../../arm-none-eabi/lib/armv7e-m/libg_nano.a(lib_a-init.o):
In function `__libc_init_array':

init.c:(.text.__libc_init_array+0x1c): undefined reference to `_init'

collect2: error: ld returned 1 exit status

make: *** [stm32-camera.elf] Error 1

解决方法：打开project properties设置页面，选择c/c++ Build -&gt; Settings -&gt; Tool Settings
-&gt; Cross ARM C ++ Linker -&gt; General， 取消勾选 “Do not use standard start
files —nonstartfiles”，确认再编译即可通过。

  * 编译过程中遇到错误：/Users/enix/Public/gcc-arm-none-eabi-4_9-2015q2/bin/../lib/gcc/arm-none-eabi/4.9.3/../../../../arm-none-eabi/bin/ld: cannot open linker script file mem.ld: No such file or directory

  

  

