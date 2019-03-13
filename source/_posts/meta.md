---
title: meta全解释
date: 2016-11-18 12:44:27
tags: HTML
---

整理下HTML中常用的meta。
<!-- more -->
```html
<meta http-equiv='x-dns-prefetch-control' content='on'/>;
<link rel="dns-prefetch" href="http://bdimg.share.baidu.com" />

```
dns-prefetch是具有此域名不需要点击就在后台解析，域名解析和内容载入是串行的网络操作

Chrome内置了DNS Prefetching技术, Firefox 3.5 也引入了这一特性，由于Chrome和Firefox 3.5本身对DNS预解析做了相应优化设置，所以设置DNS预解析的不良影响之一就是可能会降低Google Chrome浏览器及火狐Firefox 3.5浏览器的用户体验。

SEO作用

```html
<!-- H5标准声明，使用 HTML5 doctype，不区分大小写 -->
<!DOCTYPE html>
<!-- 标准的 lang 属性写法 -->

<head lang="en">
  <!-- 声明文档使用的字符编码 -->
  <meta charset="utf-8">
  <!-- 强制浏览器的渲染方式，默认chrome，ie按最最新版本来渲染.chrome=1的意思是chrome frame不改变ie外观就可以使ie里变成chrome -->
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1″ />
<!-- 页面描述 -->
<meta name=" description" content="不超过150个字符" />
  <!-- 页面关键词 -->
  <meta name="keywords" content="" />
  <!-- 网页作者 -->
  <meta name="author" content="name, email@gmail.com" />
  <!-- 搜索引擎抓取 -->
  <meta name="robots" content="index,follow" />
  <!-- 为移动设备添加 viewport -->
  <meta name="viewport" content="initial-scale=1, maximum-scale=3, minimum-scale=1, user-scalable=no">
  <!-- 启用360浏览器的极速模式(webkit) -->
  <meta name="renderer" content="webkit">
  <!-- 360提出的浏览器内核控制meta标签，页面渲染使用：webkit内核，ie兼容内核，ie标准内核 -->
  <!-- <meta name="renderer" content="webkit|ie-comp|ie-stand"> -->
  禁止百度转码。百度转码可能会加上广告等，加上了百度也可能不遵守，最好都加上
  <meta http-equiv="Cache-Control" content="no-transform | no-siteapp" />
  <!-- uc强制竖屏 -->
  <meta name=" screen-orientation" content="portrait">
  <!-- QQ强制竖屏 -->
  <meta name="x5-orientation" content="portrait">
  <!-- UC强制全屏 -->
  <meta name="full-screen" content="yes">
  <!-- QQ强制全屏 -->
  <meta name="x5-fullscreen" content="true">
  <!-- 设置页面不缓存 -->
  <meta http-equiv="expires" content="0″>
<meta http-equiv=" pragma" content="no-cache">
  <meta http-equiv="cache-control" content="no-cache">
  <!-- 将页面中http升级为https访问 -->
  <meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
</head>

<body>
</body>

</html>

```


