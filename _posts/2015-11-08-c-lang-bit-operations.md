---
layout: blog
categories: blog
published: true
title: c语言的位操作
---
位运算符

  * ~ 取反
  * &amp; 位与
  * |  位或
  * ^ 位异huo

  

**掩码**

  

flags &amp;= MASK;

例如，MASK = 000100, flags = 111111, flags &amp;= MASK = 000100

  

**打开位**

  

flags |=  MASK;

例如，MASK = 000100, flags = 111000, flags = 111000 | 000100 = 111100

  

**位关闭**

  

flags &amp;= ~MASK;

例如，MASK = 000101, flags = 111100, flags = 111100 &amp; (~000101) = 111100
&amp; 111010 = 111000

  

**位转置(toggle)**

  

flags ^= MASK;

例如，MASK = 000101, flags = 101010, flags = 101010 ^ 000101 = 101111

1 ^ 0 = 1, 0 ^ 0 = 0，所以对于不需要改变的位，只要把MASK对应的位置为0。

  

查看位的值

  

if(( flags &amp; MASK ) == MASK){...}

  

  



  

  

