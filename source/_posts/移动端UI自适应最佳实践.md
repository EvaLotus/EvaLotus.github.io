---
title: 移动端UI自适应最佳实践
date: 2017-08-18 17:14:58
tags: CSS
---
我们的工作少不了要和UI设计师打交道，一般设计小姐姐给的图都是iPhone6尺寸750\*1334px，这样我们在切图时，只需要将像素值/2作为元素的宽高，字体的尺寸即可。但是一般的设计图中border是1px，怎么把元素的border设置为0.5px呢？

有的小伙伴说，那是他们不专业，随便写个1px就可以了。但是我们rgb不变直接写成1px的话，会明显发现border显得太粗颜色太深。

作为一个有追求的前端工程师，应该怎么做到border的高保真呢？浏览器支持0.5px吗？
<!-- more -->
### 历史
我们先忽略这个0.5px的问题，思考下移动端适配的历史。

最开始我们适配时都是通过百分比来解决的。但是百分比只能解决宽度自适应，高度却不能自适应，小尺寸屏幕上按钮过高，显得臃肿

对于不同屏幕上宽高比一致的方案，使用百分比达不到效果

其他的解决方案
px：px相对于屏幕分辨率而言（与物理设备有关），普通屏是1px对应1个物理像素点，高清屏是1px对应4个物理像素点
em：相对长度单位，相对于当前对象内的字体尺寸，em的值不固定，继承父级字体大小
rem：相对于html根元素的尺寸。只修改根元素html就可以成比例的调整所有字体大小。

### 兼容性

rem兼容性：

iOS6.1+都支持
Android 2.1+都支持
主流浏览器都支持

### em和rem

em：相对于当前元素内的字体尺寸。当前元素font-size=20px，当前元素中的1em=20px

```
h1{font-size:20px;} // 1em=20px
```

rem简单，1rem始终是那么大，但rem使组件不那么模块化。实际开发中可以结合使用:

1.当元素属性值需要根据元素字体尺寸缩放时，就使用em

2.其他情况都简单的使用rem

### rem

rem：font-size of the root element

rem：期望字体/body-font-size rem

rem不仅适用于字体，也可以用于width height margin

使用scss可以直接

```
html{font-size:10px;}1rem=10px;
html{font-size:62.5%;} // =10px

@mixin px2rem($px){
$rem:37.5px;// window.innerWidth/10
@return ($px/$rem)+rem;
}

height:px2rem(90px);
```

使用media query来设置根节点，上面的设置方法有一个很明显的问题font-size是在一个屏幕宽度的区间上有一个基准值。像安卓手机种类的繁多，屏幕大小就更多的情况下，上面的方法很鸡肋。

第二种解决方案，就是使用JavaScript根据当前屏幕的宽度动态计算font-size值，这种方法可以保证屏幕宽度连续变化的时候，font-size基准值也是连续变化的。

那么最后一个问题也来了：为什么将计算rem单位的js放在head标签里面？

一句话总结：在浏览器中文档流是从上往下加载渲染的。为了保证发生不必要的重绘或者是重排肯定是越早给根节点设置font-size值越好。

### rem适用场景

整体的布局还是使用百分比
使用rem的最佳场景是,遇到例如多列带有图片的列表,常常需要图片固定宽高比例
研究了一些网站，比如淘宝，对字体字体一般情况建议使用px
出现1px像素线的地方，仍旧使用border-width:1px;而不是border-width:.1rem;

### .5px的解决

设计稿中常常是iPhone6尺寸750\*1334px,border是1px

```javascript
// 移动端设置viewport
// 直接按照750*1334来切图
// 解决.5px border
var dpr = window.devicePixelRatio,
viewPort = document.createElement('meta'),
head = document.getElementsByTagName('head')[0];
viewPort.setAttribute('name', 'viewport');
viewPort.setAttribute('content', 'initial-scale=' + 1 / dpr + ', maximum-scale=' + 1 / dpr + ', minimum-scale=' + 1 / dpr + ', user-scalable=no');
head.insertBefore(viewPort, document.getElementsByTagName('meta')[0]);

// 任意浏览器的默认font-size=16px; html{font-size:6.25% // 10px;}则=>1rem=6.25%=10px;
// 按照750*1334来切图则 1rem=>37.5px 设置好了rem可以直接在css中rem
// pxToRem中rem直接定为37.5px;尺寸都是照这个来的
document.getElementsByTagName('html')[0].style.fontSize = window.innerWidth / 10 + 'px';
```

### 小程序中

rpx是微信小程序专用尺寸单位，规定屏幕宽为750rpx
注：开发微信小程序时设计师可以用 iPhone6 作为视觉稿的标准。
建议：设计稿使用设备宽度750px比较容易计算750px的话1rpx=1px，这样的话，设计图上量出来的尺寸是多少px就是多少rpx，至于在不同的设备上实际上要换算成多少个rem就交给小程序自己换算

像素：

物理像素：

设备独立像素

设备像素比：dpr

### vm

vw，vh，vmin和vmax是基于viewport的长度单位

* vw: viewport’s width,1vw等于window.innerWidth的1%

* vh：viewport’s height，1vh等于window.innerHeihgt的1%

* vmin: vmin的值是当前vw和vh中较小的值

* vmax: vmax的值是当前vw和vh中较大的值

针对750px的设计稿，将px值除以75得到vw的值

尽管在某些Android机型上还存在兼容问题，我们也可以使用Viewport Units Buggyfill，具体见《如何在Vue项目中使用vw实现移动端适配》

回顾与总结：



细节与思考：

面试中常问的.5px的实现，现在你会了吗？

参考文献：

[http://www.html-js.com/article/4771](http://www.html-js.com/article/4771)
[http://www.alloyteam.com/2016/03/mobile-web-adaptation-tool-rem/](http://www.alloyteam.com/2016/03/mobile-web-adaptation-tool-rem/)
移动端适配和.5px的由来[http://mp.weixin.qq.com/s/5BpIpM6WAvWaX2Y8IrVETA](http://mp.weixin.qq.com/s/5BpIpM6WAvWaX2Y8IrVETA)
