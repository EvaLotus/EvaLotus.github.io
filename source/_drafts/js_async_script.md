---
title: 深入理解浏览器的执行顺序
date: 2018-09-18 14:07:59
tags: javascript
---
性能优化中有一点是老生常谈，需要将script标签放在页面的底部，CSS放在页面顶部，我们都知道这是为了防止JS的执行阻塞页面的渲染，以免用户看到白页。

那CSS是否会阻塞渲染呢？

必须要将script标签放在页面的底部，CSS放在页面顶部吗？

你真的懂浏览器的渲染机制吗？
<!-- more -->
看了些defer，async的文章，copy一堆概念可能自己都看不懂，先丢开困扰人的defer，async属性，我们先从古老的jQuery说起。

```html
<script src="jQuery.JS"></script>
<script>
  console.log($);
</script>
```
有没有想过后面的`console.log($)`执行的时候可能jQuery还没加载完毕？

不不不，不用担心，这里并**不是异步设计**，虽然以文件形式引入的JS，但是整个html依然是**从上到下执行**进行DOM构建，遇到script标签时会先暂停DOM构建，把外部引入的jQuery**下载并执行完毕**之后才会继续解析后面的DOM，所以完全不用担心$未声明的问题。

简言之：
浏览器遇到同步JS，会：
- 暂停DOM构建
- 请求下载JS
- 从上到下执行JS
- 继续DOM构建

既然是同步执行，其实不仅script会引入外部资源，img，CSS，iframe也会引入外部资源呀，也是同步加载有阻塞吗？
TODO这些外部资源的超时时间是如何设置的呢？

我们思考下日常的现象：

1.img：我们浏览电商网站时明显可以看到图片是最后展现出来的，如果图片的加载会暂停DOM的构建，那肯定会出现长时间的白屏，所以img引入外部资源应该是不会阻塞DOM的构建

2.iframe：鉴于是独立环境，默认也先认为应该是不会阻塞DOM的构建

3.CSS：在网络情况极其不好的情况下，一般会白屏比较久，但最后出现的还是UI完整的网页。有时会出现UI完全乱掉的情况，但是等待一会儿也不会自动变成显示正常的UI，因为这时候一般都是CSS加载超时了。所以CSS应该也不会阻塞DOM的构建

来demo下测试下。


和我们想的差别不太大，但是有个颠覆三观的细节发生了，**CSS居然会阻塞JS的执行**，
我们把CSS改为img和iframe，发现没等资源返回JS立马执行了。

本来以为<link rel="stylesheet">不过是个普通的DOM节点，没想到居然会影响JS的执行
其实也可以理解，JS阻塞DOM的构建的原因是：JS可能会改变页面的结构，改变DOM Tree或CSSOM Tree。


我们先来复习下浏览器的渲染机制，不用浏览器使用的内核不同，所以他们的渲染过程也是不一样的。目前主要有两个：
webkit渲染过程


Gecko渲染过程

从上面两个流程图我们可以看出来，浏览器渲染的流程如下：

- 解析HTML生成DOM Tree，遇到link、script、img等标签时，浏览器会向服务器发送请求资源。
- 加载CSS文件完成后同步解析生成CSSOM Tree。
- Render Tree:Dom Tree和CSSOM Tree都解析完成后结合生成Render Tree。
- Layout:计算元素的位置开始Layout定位。
- Paint:通过显卡，将Layout后的节点内容(含已下载图片)呈现到屏幕上。

从流程我们可以看出来

DOM解析和CSS解析是两个并行的进程，所以这也解释了为什么CSS加载不会阻塞DOM的解析。
然而，由于Render Tree是依赖于DOM Tree和CSSOM Tree的，所以他必须等待到CSSOM Tree构建完成，也就是CSS资源加载完成(或者CSS资源加载失败)后，才能开始渲染。因此，CSS加载会阻塞DOM渲染。
由于JS可能会操作之前的DOM节点和CSS样式，浏览器会维持html中CSS和JS的顺序，所以CSS会阻塞后面JS的执行。

img的加载不会阻塞DOM的解析，但img加载后并不渲染，它需要等待Render Tree生成完后才和Render Tree一起渲染出来。未下载完的图片需等下载完后才渲染。


避免浏览器的重绘repaint和回流reflow

#### CSS加载是否会造成阻塞
我们拿img，CSS，iframe来引入被墙的资源试试，或者直接在chrome dev Tool中设置slow 3G。
这部分引用[陈纪庚的文章](https://juejin.im/post/5b88ddca6fb9a019c7717096)，写的非常清楚，感谢。
```html
<style>
h1 {
  color: red !important
}

</style>
<script>
function h() {
  console.log(document.querySelectorAll('h1'))
}
setTimeout(h, 0)

</script>
<link href="https://cdn.bootCSS.com/bootstrap/4.0.0-alpha.6/CSS/bootstrap.CSS" rel="stylesheet">
```
可以发现CSS未加载完成时就console出了DOM元素，但是此时页面居然是白屏？

所以可以说明：
- CSS加载不会阻塞DOM的构建
- CSS加载会阻塞DOM树的渲染

构建DOM
构建CSSOM 组成RenderTree，Layout计算出Render Tree各个节点的具体位置，通过显卡将节点呈现在屏幕上

注意是边解析边渲染的
必须等CSS加载完成之后才会绘制
1.是为了避免出现一会儿蓝色一会儿绿色的情况，2.是为了节约性能
也有可能就是出现了一会儿蓝色一会儿绿色，只不过没有展示出来


CSS加载会阻塞JS的运行吗？
上文说到CSS加载不会阻塞DOM的构建，`<link href="XXX"  rel="stylesheet">`本身也是DOM树中的一个节点，却出现了奇怪的现象
```
<script type="text/javascript">
  console.log('1.在css加载之前就执行了' + new Date().valueOf());

</script>
<link href="https://cdn.bootcss.com/bootstrap/4.0.0-alpha.6/css/bootstrap.css" rel="stylesheet">
<script type="text/javascript">
  console.log('2.必须等css加载完之后才会执行'+new Date().valueOf());

</script>
```
这里我们不说是CSS影响了JS的执行，换个说法会更好理解一点，JS会察言观色，看到前面有CSS，就等CSS执行完了之后才执行。
整体来说，可以理解为：，
- script标签会影响DOM的构建，需要等script标签执行完才继续DOM的构建
- 除了script标签，其他引用外部资源的都不会影响DOM的构建
- script有自动优化的功能：因为JS可能会操作之前的Dom节点和CSS样式，所以浏览器会维持html中CSS和JS的顺序，即浏览器信任我们，按照我们写的顺序去加载js和css，上面的例子中1会很快就打印出来，2会等css加载完之后才会打印。

来复习下浏览器的渲染机制

DOM解析和CSS解析是两个并行的进程，所以这也解释了为什么CSS加载不会阻塞DOM的解析。
然而，由于Render Tree是依赖于DOM Tree和CSSOM Tree的，所以他必须等待到CSSOM Tree构建完成，也就是CSS资源加载完成(或者CSS资源加载失败)后，才能开始渲染。因此，CSS加载是会阻塞Dom的渲染的。
由于JS可能会操作之前的Dom节点和CSS样式，因此浏览器会维持html中CSS和JS的顺序。因此，样式表会在后面的JS执行前先加载执行完毕。所以CSS会阻塞后面JS的执行。



浏览器虽然从上到下解析，但是实际上解析渲染之前会去看CSS和script的位置？
所以CSS放在顶部还是底部都无所谓，反正浏览器会等CSS加载完毕之后才开始渲染


注意上面说到**下载并执行**

是否还记得在初学jQuery时，通常会使用
```JS
$(document).ready(function(){
  ...
});
// 或
$(function(){
	...
})
```
保证我们的代码在**DOM加载完成**后执行，啥叫DOM加载完成呢？外面不包这么一层会有什么区别呢？
比如我们的script标签写在了body顶部，`$('btn').click(...)`调用时页面中根本没有btn，所以如果不包在上面的代码中，添加事件根本不会生效，包在上面的callback中，就可以保证事件正常执行。

外面不包这么一层的话，那就必须要把script写在body底部。

如果非不把script标签写在body底部
```html
<script>
function longTask() {
  var i = 0;
  while (i < 10000) {
    console.log(i);
    i++;
  }
}
longTask();
// 耗时操作执行完毕之后才会渲染出div
</script>
<div>Can you see me?</div>
```
JS执行完毕之后才会渲染出div，用户会长时间看到白页

```JS
(function() {
  var ie = !!(window.attachEvent && !window.opera);
  var wk = /webkit\/(\d+)/i.test(navigator.userAgent) && (RegExp.$1 < 525);
  var fn = [];
  var run = function() { for (var i = 0; i < fn.length; i++) fn[i](); };
  var d = document;
  d.ready = function(f) {
    if (!ie && !wk && d.addEventListener)
      return d.addEventListener('DOMContentLoaded', f, false);
    if (fn.push(f) > 1) return;
    if (ie)
      (function() {
        try { d.documentElement.doScroll('left');
          run(); } catch (err) { setTimeout(arguments.callee, 0); }
      })();
    else if (wk)
      var t = setInterval(function() {
        if (/^(loaded|complete)$/.test(d.readyState))
          clearInterval(t), run();
      }, 0);
  };
})();
```
啥叫DOM加载完毕呢？如果有img呢？
```html
<img src="https://evacoder.com/longWaiting.png" alt="">
```
比如这个图片很大，需要很久才能返回，也是同步执行的吗？TODO？？？



总结：所以不考虑async和defer属性的script，整个html是从上到下执行的，遇到外部文件也是同步执行，会存在阻塞渲染。

页面加载过程中有哪些我们需要注意的时间点呢？

document.onDOMContentLoaded：






注意所有的外部资源都有下载和执行的不同的时机。



在DOM加载完毕之后浏览器会通过JS为DOM元素添加事件，

有哪些我们需要注意的时间点呢？
window.onload:页面中所有元素及资源完全加载到浏览器（async，defer）


是指</html>解析完毕吗？
和把script标签放在body底部，有区别吗？

为什么jQuery中要求加上
<!-- more -->


#### async

async的意思是应该立即下载脚本，但是不应妨碍页面其他的操作。注意是**下载**而不是**加载**。

```html
<script src="https://www.evacoder.com/1.JS" async></script>
<script src="https://www.evacoder.com/2.JS" async></script>
```

不能保证1.JS在2.JS之前解析，所以需要保证两者之间互不依赖。

一定会在document.load之前执行，但是可能会在DOMContentLoaded触发之前或之后执行。

下载和解析：

aynsc：立即下载，到底何时执行？加载完就执行？遇到阻塞的情况就最后执行？不阻塞就立刻执行吗？

defer：立即下载，遇到`</html>`之后才执行？所有元素解析完成之后，DOMContentLoaded触发之前执行。类似放在最底部，但是是先下载的。

最大6个资源一起下载

window.onload:

document.onload：

DOMLoading：浏览器开始解析dom

DOMInteractive：浏览器解析好dom树

DOMContentLoaded：同步的JS执行完毕

#### sdk的版本控制

引入一个不变的小的JS，设置超长缓存或不设置缓存？

在其中通过document.createElement\('script'\)来异步加载JS，这个时候生成的JS本来就是aynsc的，何时加载完已经和window.onload等一点关系都没有了，此时可以通过script.onload来获取加载完成的状态，ie可以使用onreadystatechange

业务方加载了你的JS sdk，如何在不用通知业务的情况下，就可以进行版本更新?

JS sdk load完成之前，如何调用

```JS
load JS

loadSDK(function(){


});
```

通过动态插入script标签的方式

加载完成之前会阻止onload事件的出发，而现在很多页面的代码都在onload时

DOMContentLoaded：页面\(document\)已经解析完成，页面中的dom元素已经可用。但是页面中引用的图片、subframe可能还没有加载完

onLoad：页面的所有资源都加载完毕（包括图片）

async是html5的新属性，async 属性规定一旦脚本可用，则会异步执行（一旦下载完毕就会立刻执行）。

需要注意的是async 属性仅适用于外部脚本（只有在使用 src 属性时）

noscript 元素用来定义在脚本未被执行时的替代内容（文本）。

```html
<noscript>Your browser does not support JavaScript!</noscript>
```
#### 页面加载的两个重要时间点



#### readyState

[https://blog.csdn.net/u011700203/article/details/47656857](https://blog.csdn.net/u011700203/article/details/47656857)