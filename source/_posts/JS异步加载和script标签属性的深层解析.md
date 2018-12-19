---
title: JS异步加载和script标签属性的深层解析
date: 2018-09-18 14:07:59
tags: javascript
---
JS异步加载和script标签属性的深层解析
<!-- more -->
```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Document</title>
</head>
<body>
<script src="https://www.evacoder.com/404.js"></script>
<script>
// console还是会执行，但是会先返回js 404之后才执行。
// 浏览器确实是按script标签的顺序，依次加载执行的，所以经常说需要将script放在body的底部，防止阻塞页面。
console.log(1);
</script>
</body>
</html>
```

只要不存在defer和async属性，浏览器都会按照script出现的顺序依次解析。

script加上defer或async属性之后，会先console，才返回404

```
<script src="https://www.evacoder.com/b.js" async defer="defer"></script>
```

#### defer

脚本可以延迟到**文档完全被解析**和显示之后才执行。表示脚本在执行时不会影响页面的构造。立即下载，延迟解析（遇到`</html>`之后才执行）。

出现多个script defer时，延迟解析也是按照出现的顺序来解析的？在DOMContentLoaded之前解析？

规定是这么规定，但是现实却是无法确定谁先解析，甚至都不一定在DOMContentLoaded之前解析。所以一般最好只包含一个defer script。

浏览器兼容性：

IE4-不支持，兼容性不错，但是最好还是将defer script放在body底部。

那请求发出的时机呢？？

下载时机：立即下载

执行时机：

#### async

async的意思是应该立即下载脚本，但是不应妨碍页面其他的操作。注意是**下载**而不是**加载**。

```
<script src="https://www.evacoder.com/1.js" async></script>
<script src="https://www.evacoder.com/2.js" async></script>
```

不能保证1.js在2.js之前解析，所以需要保证两者之间互不依赖。

一定会在document.load之前执行，但是可能会在DOMContentLoaded触发之前或之后执行。

下载和解析：

aynsc：立即下载，到底何时执行？加载完就执行？遇到阻塞的情况就最后执行？不阻塞就立刻执行吗？

defer：立即下载，遇到`</html>`之后才执行？所有元素解析完成之后，DOMContentLoaded触发之前执行。类似放在最底部，但是是先下载的。

最大6个资源一起下载

window.onload:

document.onload：

DOMLoading：浏览器开始解析dom

DOMInteractive：浏览器解析好dom树

DOMContentLoaded：同步的js执行完毕

#### sdk的版本控制

引入一个不变的小的js，设置超长缓存或不设置缓存？

在其中通过document.createElement\('script'\)来异步加载js，这个时候生成的js本来就是aynsc的，何时加载完已经和window.onload等一点关系都没有了，此时可以通过script.onload来获取加载完成的状态，ie可以使用onreadystatechange

业务方加载了你的js sdk，如何在不用通知业务的情况下，就可以进行版本更新?

JS sdk load完成之前，如何调用

```
load js

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

```
<noscript>Your browser does not support JavaScript!</noscript>
```


#### readyState

[https://blog.csdn.net/u011700203/article/details/47656857](https://blog.csdn.net/u011700203/article/details/47656857)