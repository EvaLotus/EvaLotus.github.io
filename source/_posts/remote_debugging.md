---
title: 「 工具 」前端远程调试及空白或404解决
date: 2019-02-17 20:11:15
tags: Tool
---
因为自己的开发环境搭建的太舒适，在别人的电脑上查问题感觉完全无法展现自己的能力。

再加上已经习惯了chrome dev tool的每一种功能，一换到safari上查兼容性问题，感觉自己变成了白痴。

更不要说去测试webview里的问题，console都没法看，真的要一行一行alert吗？

啥？你居然不会remote debugging！

那你怎么证明到底是你还是native 开发的锅？啥证据都没有。

虽然模拟器也能完成调试的功能，但是不能完全依赖模拟的结果，而且有些特定的机型并没有模拟器，有时候必须要在真机上复现问题，最重要的是模拟器没有chrome devtools呀。

<!-- more -->

#### Andriod
其实这个配置非常简单，用起来也方便，小伙伴们主要遇到的就是打开页面空白或者404的问题。

所以最重要的是要检查下PC能否科学地 **网上冲浪**

有时候能访问google都不一定成功，需要试下这个地址

[https://chrome-devtools-frontend.appspot.com/](https://chrome-devtools-frontend.appspot.com/)

如果不行的话，配置以下host

```bash
172.217.161.180 chrome-devtools-frontend.appspot.com
172.217.161.180 chrometophone.appspot.com
```

现在访问基本上是秒开啦。

访问：`chrome://inspect/#devices`

然后就可以方便的调试了。

1.在开发者模式下允许通过usb来调试

2.重新连接下usb

![](/images/remote_debugging1.png)

出现上图，点击inspect就可以开始调试啦。

不必安装chrome浏览器，其他的浏览器和webview都可以进行调试。

如果webview测试不了，还需要安卓端配置下允许调试

```java
WebView.setWebContentsDebuggingEnabled(true);
```

#### iOS
iOS的remote debugging比较难，难在首先你需要一台mac哈哈哈，但是mac却可以直接调试Andriod。

1.iPhone上设置->Safari-> 高级-> Web 检查器，打开

2.mac 上 safara->偏好设置->高级->在菜单栏显示“开发”菜单

3.usb连接两者

4.打开 mac safari，顶部菜单中的开发，选择设备点开即可
![](/images/remote_debugging2.png)

也是可以调试webview，safari

掌握了远程调试的技能，基本可以解决大部分问题了。
