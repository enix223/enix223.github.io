---
layout: blog
categories: blog
published: false
title: 关于Objective-C的@dynamic学习与研究
tags: ''
---
## 前言

---

这些天在看《Effective Objective-C 2.0》一书，碰到了@dynamic，因为书中没有详细展开说明它的用法，决定Stackoverflow一下，发现SO上面有几个比较热门的问答，是关于@dynamic的用法。

1. [@synthesize vs @dynamic, what are the differences?](http://stackoverflow.com/questions/1160498/synthesize-vs-dynamic-what-are-the-differences)

2. [What does @dynamic do in Objective-C?](http://stackoverflow.com/questions/4621952/what-does-dynamic-do-in-objective-c)

3. [What is @dynamic in iPad/iPhone](http://stackoverflow.com/questions/5432787/what-is-dynamic-in-ipad-iphone)

这些post的回答都非常到位。

大多数朋友都会把@dynamic与@synthesize作比较，Effective一书中，对于对二者的作用是这么讲解的：

    @synthesize 是帮助我们为@property属性自动生成对应的instance variable和相应的ivar的getter和setter。
    @dynamic 则是告诉compiler编译器，“hey，兄弟，你不需要帮我生成property对应的instance variable和相应的getter和setter，我会在其他地方定义，或者留到runtime的时候提供这些方法。”
    

@synthesize其实就是一个语法糖，帮我们简化了编写大量的setter和getter方法。
@dynamic就如其名一样，我不在编译阶段提供property属性的对应的instance variable，getter和setter，而是留到其他地方定义。

SO里面有一个例子非常好：

父类：

    @property (nonatomic, retain) NSButton *someButton;
    ...
    @synthesize someButton;
    
子类：

    @property (nonatomic, retain) IBOutlet NSButton *someButton;
    ...
    @dynamic someButton;
    
    
我们在父类中定义了一个按钮`someButton`，并且借助`@synthesize`帮我们生成了对应的ivar和getter／setter。而我们子类中，常常需要跟跟storyboard的UI做绑定，这时候`@dynamic`就派上用场了。

当我们访问子类`someButton`的属性的时候，因为子类并没有生成对应的ivar和getter／setter，所以程序会继续搜索父类，看看是否存在`someButton`的getter／setter的定义。而在父类的定义中，我们刚好使用@sythesize定义了`someButton`的getter／setter，所以此时，就会实际调用父类的getter／setter实现。

而@dynamic的另外一个使用场景就是Core Data的`NSManagedObject`。

虽然日常使用的不多，但本着深入学习的目的，决定自己体验一下这个关键字的用法。

## 把自己的手弄脏

----
