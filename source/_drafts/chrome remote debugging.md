---
title: chrome remote debugging
date: 2018-12-29 13:04:44
tags: chrome
---
快速解决和native开发的扯皮，省下时间做自己想做的事。

你总不能说肯定不是我的问题吧。

<!-- more -->
1.在andriod机设置=>开发者选项允许usb连接测试

2.连接usb，信任pc

3.只能调试debug模式（允许remote debugging）的apk

4.PC上.usb连接成功，出现设备

没出现设备可能是驱动没装成功。

5.打开chrome的chrome://inspect

会列出你的andriod设备

6.访问需要调试的页面，PC端出现页面url

没出现你访问的页面？可能是因为你安装的apk不允许remote debugging。

4.点击url下方的inspect

一直是空白的是因为调试需要访问google的服务器，一定要注意需要FQ（你懂的！！！）

不需要在手机上安装chrome for andriod

点击inspect，focus tab， reload，close

有时候需要5037的port forwarding