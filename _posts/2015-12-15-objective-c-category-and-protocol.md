---
layout: blog
categories: blog
published: true
title: Category & Protocol
---
**创建一个category**

  

![42bd7b915143dbf5d238ecc62b2ce44a](/media/42bd7b915143dbf5d238ecc62b2ce44a.png)

  

按照一般的约定，存放category的文件名按如下方式命名：

&lt;class name&gt;+&lt;category name&gt;.h

&lt;class name&gt;+&lt;category name&gt;.m

  

例如，对于Fraction的例子为：

Fraction+MathOps.h

Fraction+MathOps.m

  

**Class Extension**

  

定义category的时候，( )中留空，通过_class
extension_，我们可以对一个类添加properties和methods。而这对于普通的category是不允许的。class
extension的实现代码，直接添加到implement section中，而category必须要在另外一个implement
section中实现。另外，class extension的方式是私有的。

_**  
**_

**_请注意，category可以覆盖之前定义的方法。所以在实际操作中，应记住这一点。如果必须这么做，可以在你的category中调用原来的方法，再添加你需要的代码。另外，category的方法会被子类继承。_**

_**  
**_

_**  
**_

**Protocol**

  

协议是一个方法列表，在类之间共享。一个协议中的方法可以都被实现（required），也可以选择性的实现（optional）。

  

定义一个protocol

![a1807a0f339820cd904c17bb5d9c5f18](/media/a1807a0f339820cd904c17bb5d9c5f18.png)

其中paint和erase都是必须实现的，而outline是可以实现也可以不实现。

  

遵循一个protocol

![b458be6389dd87fffab95051ecb928eb](/media/b458be6389dd87fffab95051ecb928eb.png)

  

测试一个object是否conform一个协议

conformToProtocol:

  

![913df0d2add921c78029077a94d347ad](/media/913df0d2add921c78029077a94d347ad.png)

  

测试一个object是否实现了一个方法

![d9b596c2caf69cfdf0dd2455fa5acc32](/media/d9b596c2caf69cfdf0dd2455fa5acc32.png)

  

或者在明确指定一个object必须是遵循某个protocol

![7a34c6fd52e7860177bab340f305b661](/media/7a34c6fd52e7860177bab340f305b661.png)

  

protocol也可以继承

![dbeb0c049ff03fdaf604df23abc0ca4a](/media/dbeb0c049ff03fdaf604df23abc0ca4a.png)

  

**delegation 委托**

  

委托，也就是定义protocol的类，自己不实现该protocol里面的方法，而是委托给遵循该protocol的类来实现这些方法。也就是定义protocol
的类委托给实现protocol的类。

  

cocoa和iOS都内部实现都依赖于委托。例如你要显示一个Table的时候，需要用到UITableView类，但是UITableView类并不知道具体要显示
多少行，title是什么，而这些恰恰是需要开发者来实现，所以就有一个协议UITableViewDataSource的协议，让开发者去实现。

  

**Informal Protocol非正式协议**

  

非正式协议其实就是一个category，但是没有具体的实现其中的方法。_informal protocol_有时候也被称为_abstract
protocol_。一般非正式协议都是在父类中定义。某个对象是否遵循informal
协议只能在runtime时检测，无法在compile的时候预知。（respondsToSelector:方法检测）

  

![07fcd0e806799ca00a02e050e3e44013](/media/07fcd0e806799ca00a02e050e3e44013.png)

  

  

  

