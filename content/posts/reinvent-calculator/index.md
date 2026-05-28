---
title: 'Reinvent! 更适合程序员的计算器'
date: 2026-05-25T11:29:39+08:00
draft: false
---

# 1. 前言

计算器是程序员必备的工具，macos/windows/ubuntu上都自带了的计算器，且功能都足够的强大，但是他们在实际使用中仍然有一些问题无法解决。

1. **无法表达signed有符号整数**，在物联网开发中，我们时常需要跟有符号数字打交道。有符号整数需要使用最高位来表示符号为，1表示负数，0表示正数。而目前的系统自带的计算器都是以unsigned整数来设计，自然也无法表达负数。
2. **缺少跨平台、操作一致的计算器**，macos/windows/ubuntu自带的计算器功能强大，但是三者在使用上存在差异，如果有一款计算器可以跨平台使用，且功能保持一致，那么我们可以在不同平台间保持使用习惯的一致性，大大提高我们的工作效率。

针对以上的问题，我打算开发一款基于cloudesk.top云桌面平台的网页应用，实现一个跨平台的，无需安装的，操作一致的计算器。

# 2. 使用介绍

## cloudesk介绍

为了应付日常开发需要用到的一些常用工具，我开发了cloudesk.top，方便有同样需求的朋友使用，该网站有如下优点
* **无广告**，拒绝烦人的广告，让工作更专心
* **数据安全和隐私保证**，cloudesk上的所有应用均在浏览器侧运行（即SPA应用），没有后端接口和数据库，保证了数据的隐私安全，仅接入了google analytics，采集网站的访问统计，其余数据均不会上传到任何服务器。

## 计算器介绍

计算器的访问网址: https://cloudesk.top/calculator

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F3tVAovHeAyIfr3OPtd1ia1U5kicjPRY7MO8ak9Usic4pl1PdYH3ofL3tlyGmFiaVfztHXTict4UcBQv3ceHZytCz3f01f1Cy97PibgORAIkdfyJrE%2F0%3Ffrom%3Dappmsg)

手机端也可以访问，且自动适配手机视图


![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F3tVAovHeAyIMdnQrf0gFS39XCBlqQH4YZcVzNtsgvl8tuq2GABwiaxfXcNUquiaibia3tJZmwn1PFQ0ZBf4rwra5gFJ8vhib2Z6Y0D9xqibgIBP2g%2F0%3Ffrom%3Dappmsg)


## 三种模式

与macos的计算器类似，计算器也是采用了3种布局，适合不同的计算场景，其中包括：
* 基础视图，界面简介明了，主要是完成基础的加减乘除运算
* 科学计算视图，界面比基础视图复杂一点，包含了多种科学计算的函数，例如三角函数、指数运算等。
* 程序员视图，专门面向程序员的视图，包括进制转换、逻辑运算等。

基础视图和科学视图跟普通的计算器操作一致，函数的意义看字面也可以知道，唯一需要特别说明的，就是科学视图左上角的“第二功能”按钮，按下该按钮后，部分函数就会显示为另外的函数，例如sin会变为sin-1，表示sin的反函数。

另外视图之间可以通过键盘的快捷键实现快速切换，具体可以看视图名称下方的快捷键提示。

## 程序员视图

程序员视图是从事嵌入式、物联网应用行业使用频率最高的视图。cloudesk的计算器在主流的程序员视图中增加了signed符号的处理，以及整型类型的细化与区分。

### signed int/unsigned int

目前主流的计算器都只能按unsigned int来处理数字，如果你要经常跟负数打交道，那么这些计算器就无法满足您的要求，而这就是cloudesk计算器与主流计算器不一样的地方。

cloudesk计算器支持8种int类型:
* 无符号整数，uint8/uint16/uint32/uint64
* 有符号整数，int8/int16/int32/int64

我们使用时可以根据实际需要切换，例如下图选择的int32模式：

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2F3tVAovHeAyIkK8d2aEcvDBEjvNqDw8oYIOOpfFRm6UuePdsYIHeicktibfuRkRmTxAUraMTJr2COmTNbA4qVWOrQQH6yhQOJnpY2YE5h2Tibq8%2F0%3Ffrom%3Dappmsg)

最高比特(bit31)为符号位，即1表示负数。

### 比特反转

我们可以通过下方的BIT GRID方便的反转某个比特的取值，例如需要反转bit16的值，我们可以直接在BIT GRID中点击BIT16对应的比特按钮，点击后该值会反转为1

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F3tVAovHeAyKzbNfY5qlaUjKlaCLsuKNibQbpjpIgRoUzdIBFm03bCzAwWUhiclx4X7DNpO5wrN2rXurZrAhvPy63sOwueLvhL6MapTgbrZ8Ik%2F0%3Ffrom%3Dappmsg)

### 进制切换

如果需要切换不同的进制，我们可以通过点击数字下方的进制实现快速切换
* HEX，16进制
* DEC，10进制
* OCT，8进制
* BIN，2进制

![](https://wsrv.nl?url=http%3A%2F%2Fmmbiz.qpic.cn%2Fsz_mmbiz_png%2F3tVAovHeAyLwTI4A5lteTvtTyTCY5ical45cchQ07SFgdtLvhtqpwu4u5ibicKHvQtWBKV3VtjUT5JuYjrNVgOZuR5ByQ8orqDoyLjPOlIqYfs%2F0%3Ffrom%3Dappmsg)


## 结语

如果大家有任何想法和建议，欢迎发送邮件到我的技术支持的邮箱： support#cloudesk.top(#换成@)。更多精彩文章，请订阅我的公众号：

<section class="mp_profile_iframe_wrp custom_select_card_wrp" nodeleaf="">
  <mp-common-profile class="mpprofile js_uneditable custom_select_card mp_profile_iframe" data-pluginname="mpprofile" data-id="gh_3b6d80f3af36" data-nickname="X驿站" data-headimg="https://cdn-doocs.oss-cn-shenzhen.aliyuncs.com/gh/doocs/md/images/mp-logo.png" data-service_type="1" data-verify_status="0"></mp-common-profile>
  <br class="ProseMirror-trailingBreak">
</section>
