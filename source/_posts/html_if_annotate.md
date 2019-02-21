---
title: HTML条件注释
date: 2016-12-10 17:37:12
tags: HTML
---
在ie8-下不加入某个功能，回想起那个遥远的年代，初次见到是在Bootstrap中，低版本ie需要加载html5shiv，用到了条件注释。
希望幸福的你们都不需要用到~

<!-- more -->
在ie8-就不要加载某个js，通过条件注释即可实现

```
<!--[if !IE]> 除IE外都可识别 <![endif]-->
<!--[if IE]> 所有的IE可识别 <![endif]-->
<!--[if IE 8]> 仅IE8可识别 <![endif]-->
<!--[if lt IE 6]> IE6以及IE6以下（<）版本可识别 <![endif]-->
<!--[if lte IE 7]> IE7以及IE7以下（<=）版本可识别 <![endif]-->
<!--[if gt IE 7]> IE7以及IE7以上（>）版本可识别 <![endif]-->
<!--[if gte IE 6]> IE6以及IE6（>=）以上版本可识别 <![endif]-->

```

其实此标签是IE提供的，只有IE可以识别，所以在其他的浏览器中会被当做注释忽略掉。
IE10 之后不再支持条件注释，也就是说就会被当成一个普通的注释忽略掉。

```
<!-- 下面这段是 向下隐藏 语法，必须是IE且版本低于IE9-->
<!--[if lt IE 9]>
<script src="~/Scripts/jquery-1.9.1.min.js"></script>
<![endif]-->
<!-- 下面这段是 向下显示 语法，版本高于IE 9或不是IE时-->
<!--[if gte IE 9]><!-->
<script src="~/Scripts/jquery-2.0.0.min.js"></script>
<!--<![endif]-->
```