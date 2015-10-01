---
layout: blog
title: "Markdown Syntax"
date: 2015-04-30 15:19:38 +0800
comments: true
categories: blog
---

[top]:

# 1. 标题

## 1.1 标题样式1

代码:

	This is a H2
	------------

	This is a H1
	============


显示效果:

This is a H2
------------

This is a H1
============

---

## 1.2 标题样式2

代码：
	
	# This is H1
	## This is H2
	### This is H3

	或者是

	# This is H1 #
	## This is H2 #
	### This is H3 #


显示效果:

# This is H1
## This is H2
### This is H3


# 2. 区块引用

## 2.1 每一行均加入>标示引用

代码：

	> This is a quote block, you can see me?
	> Huh?
	> This is another block, you can see me?
	> Huh, Huh?


## 2.2 只有第一行加入>标示为引用

代码：

	> This is a quote block, you can see me?
	Huh?
	> This is another block, you can see me?
	Huh, Huh?

以上代码显示效果均为：

> This is a quote block, you can see me?
Huh?
> This is another block, you can see me?
Huh, Huh?


## 2.3 包含段落的引用

代码

	> This is a quote block, you can see me?
	Huh?

	> This is another block, you can see me?
	Huh, Huh?

显示效果均为：

> This is a quote block, you can see me?
Huh?

> This is another block, you can see me?
Huh, Huh?


## 2.4 区块嵌套

嵌套引用必须在嵌套结束后，添加一个`>`，并且后面不带任何字符

代码

	> This is a quote block, you can see me?
	Huh?
	>> This is an nesting quote block, you can see me?
	Huh?
	> 
	> This is another block, you can see me?
	Huh, Huh?
	>> This is an nesting quote block.
	>>> This is another nesting quote block
	>> This is an outside block
	>
	>> This is an outside block
	>
	> This is an outside block	


显示效果：

> This is a quote block, you can see me?
Huh?
>> This is an nesting quote block, you can see me?
Huh?
>
> This is another block, you can see me?
Huh, Huh?
>> This is an nesting quote block.
>>> This is another nesting quote block
>
>> This is an outside block
>
> This is an outside block

## 2.5 区块引用也可以加入其他Markdown元素

代码：

	> ### This is title 1
	>
	> This is a block with `inline code`.
	>
	> 1.	Block1
	> 1.	Block2
	>
	> This is a code:
	>	
	>	def Block1():
	>		print('foo bar')
	>

显示效果：

> ### This is title 1
>
> This is a block with `inline code`.
>
> 1.	Block1
> 1.	Block2
>

包含代码的区块:

代码：

	> 
	>     return shell_exec("echo $input | $markdown_script");

显示效果：

> 
>     return shell_exec("echo $input | $markdown_script");

## 3. 列表

### 3.1 无序列表

Markdown可以使用`*`, `+`, `-`作为无序列表的标记：

代码：

	* Red
	* Green
	* Blue
	+ Orange
	- Purple

显示效果：

* Red
* Green
* Blue	
+ Orange
- Purple

### 3.2 有序列表

代码：

	1. Item 1
	2. Item 2
	3. Item 3

显示效果：

1. Item 1
2. Item 2
3. Item 3

代码：

	1. 序号无需在意
	1. 可以使用任何数字
	1. 只要你喜欢
	1. Markdown会自动解析为有序编号

显示效果：

1. 序号无需在意
1. 可以使用任何数字
1. 只要你喜欢
1. Markdown会自动解析为有序编号

如果需要包含多行，则可以使用如下形式

代码：

	* 包含多行的序列
	我是第二行
	* 我是第二个元素
	我是第二个元素的第二行
	* 我是第三个元素
	  对齐开头的行

显示效果：

* 包含多行的序列
我是第二行
* 我是第二个元素
我是第二个元素的第二行
* 我是第三个元素
  对齐开头的行

如果段落是以数字加点开头的话，为避免错误解析为序列，必须使用`\`来转义

代码：

	2015\. This is an line start with number and dot.

显示效果：

2015\. This is an line start with number and dot.


## 4. 代码引用

代码块只需在开头简单缩进4个空格或一个制表符

代码：

	This is a code block

			A.object = new Object();

显示效果：

This is a code block

	A.object = new Object();


## 5. 分隔线

代码：

	---
	***
	* * *
	---------------

以上代码显示效果均为：

---

## 5. 链接

### 5.1 外链接

代码：

	This is an [link](http://foo.bar "foobar") to http://foo.bar

显示效果：

This is an [link](http://foo.bar "foobar") to http://foo.bar

### 5.2 本地链接

代码：

	This is an [link](/path/to/pc "link") to /path/to/pc

显示效果：

This is an [link](/path/to/pc "link") to /path/to/pc

### 5.3 网页anchor

代码：

	This is an [link][top] to the top of this page

显示效果：

This is an [link][top] to the top of this page

### 5.4 参考式链接

代码：

	This is an [Google][] link created by reference.
	This is an [link][hi] created by reference. 

	[hi]: http://foo.bar
	[Google]: http://www.google.com

显示效果：

This is an [Google][] link created by reference.
This is an [link][hi] created by reference. 

[hi]: http://foo.bar   "optional title"
[Google]: http://www.google.com   (optional title)


## 6. 强调文字

代码：

	*hello*
	_world_
	**how are you**

显示效果：


*hello*
_world_
**how are you**

## 7. 图片

图片是使用链接的形式来达到效果的。

代码：

	![Alt Text](/path/to/img.jpg)
	![Alt Text][reflink]

	[reflink]:  /path/to/img.jpg

显示效果：

![Alt Text](/path/to/img.jpg)
![Alt Text][reflink]

[reflink]:  /path/to/img.jpg

## 7. 自动转换为链接

代码：

	http://foo.bar
	enix@enixyu.com

显示效果：

http://foo.bar
enix@enixyu.com

## 8. 转义

代码：

	
	\* This is an `*`
	\- This is an `-`
	\+ Thi is an `+`

显示效果：


\* This is an `*`
\- This is an `&`
\+ Thi is an `+`