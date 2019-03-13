---
title: 「 功能点 」国际化的一些思考和注意事项
date: 2017-04-18 18:20:35
tags: 功能点
---
国际化一般由前端来维护，记录下踩过的坑。
<!-- more -->

翻译同学一般使用 https://crowdin.com 来提交翻译

和翻译同学合作需要注意以下几点：

#### 1.注意尽量不要使用拼接的字符串

比如：

tips=请同意隐私政策和用户协议

千万不要因为用户协议和隐私政策需要在多处用到，就将字符串切分为多个：

agreement=用户协议

privacy=隐私政策

notice=请同意

and=和

这样翻译同学会和莫名其妙，而且不同语言因为语序不一样，我们生硬的组合起来，展示在网页上的整句话会完全不可读。

#### 2.注意单复数的处理：

left_times=您还能发送{n}条短信

在其他语言中会有单复数的区分，1 message left，2 messages left。 在阿语中单复数的表达甚至有多达6种。 Android的resource是使用xml，处理单复数如下

<plurals name="watch_face_count">
<item quantity="one">%1$d个手盘</item>
<item quantity="other">%1$d个手盘</item>
</plurals>
#### 3.需要注意日期，金额等的处理：

#### 4.需要注意rtl语言的特殊ui处理：

有很多语言，比如iw_IL，ar，等语言习惯都是从左到右的书写习惯，此时除了需要加上body{direction:rtl}

其他的布局也需要注意将left和right互换。

#### 5.需要注意语言中单双引号的处理：

英语en_US和法语fr_FR有单引号，iw_IL希伯来语中有双引号作为字符。

如果出现在js的字符串中，导致语法错误，程序直接无法运行，此时我们需要用上转义字符来避免问题。

如果出现在html中，我们倒是没问题。

#### 6.需要注意某些语言过长的处理：

有些语言，比如mr_IR一个单词就非常长，所以一点要做好超长的处理。

要注意：

一般情况下都是前端同学来维护国际化的资源文件，确实是一件费时费力枯燥无味的工作，但是在工作中切忌眼高手低，分配到你的工作一定要做好。随着语言的种类越来越多，为了方便维护，最开始就要考虑好上面的问题的解决方案。

注意发布前需要在crowdin上查看下语言翻译进度，是否达到100%了。

需要同步工具，将翻译同学的git项目内容分散copy到我们实际的项目中

国家地区码：https://www.jianshu.com/p/f425950a09c7

TODO:
前后端分离国际化应该怎么做

参考文献：

http://taobaofed.org/blog/2016/03/21/internationalization/

https://github.com/ProtoTeam/blog/blob/master/201710/1.md

https://segmentfault.com/a/1190000009058572

https://github.com/DDFE/DDFE-blog/issues/14