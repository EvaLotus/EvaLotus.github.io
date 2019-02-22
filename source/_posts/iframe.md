---
title: 「 HTML 」iframe100问
date: 2019-02-18 22:51:35
tags: HTML

---

上篇说到iframe弹窗的登录安全，接下来我们带着问题出发，详细的了解下iframe这个历史的产物。

虽然足够古老，但是也能解决很多现代的问题。
<!-- more -->

#### Frame 间的关系

在页面中，window.top是最顶层的window，window和self都是当前window，window.parent是外层父window。

对于不存在 iframe 的页面，self，window，parent，top 这四者都相等，还有一个这 iframe中最常用的属性，frames

```js
window === window.frames // 很奇怪的是这个一直都是true，在有子iframe的页面也是
```

在有子 iframe 的页面中，这个`window.frames`就是本页面中包含的直接子frame的类数组，这个属性连接了frame 的parent，child，siblings，但是这个类数组中的元素不是直接的iframe dom，而是iframe的contentWindow，相当于是iframe环境下的window对象。

```js
window === window.frames // 一直都是true，所以在有子iframe的页面，window居然也是类array

document.getElementsByTagName('iframe')[0].contentWindow === window.frames[0]; // true
```

#### 父frame操作子frame

主要通过window.frames来操作，有时候获取到子 frame 的属性为 undefined，那是因为子frame加载需要时间，要注意等子frame加载完成之后再获取。

```js
var fr1Window = window.frames[0]; // 或者document.getElementsByTagName("iframe")[0].contentWindow;
// 操作dom
fr1Window .document.getElementsByTagName("body")[0].style.backgroundColor = "#ff6700";
// 操作变量
fr1Window.arr1=[1,2,3];
fr1Window.arr1.push('Eva');
// 操作src
document.getElementsByTagName("iframe")[0].src='https://evacoder.com';
```

#### 子frame操作父frame

主要通过window.parent来操作

```js
var parentWindow = window.parent;// window.top 是在多层级的页面中最顶层window
// 操作dom
parentWindow .document.getElementsByTagName("body")[0].style.backgroundColor = "#ff6700";
// 操作变量
parentWindow.arr1=[1,2,3];
parentWindow.arr1.push('Eva');
// 也能直接操作父frame的location，所以一定要注意iframe安全
parentWindow.location.href ='https://evacoder.com';
```

#### 子frame操作相邻 frame

```js
var siblingWindow = window.parent.frames[1];
siblingWindow.document.body.style.backgroundColor = '#ff5700';
siblingWindow.arr1 = [1, 2, 3];
siblingWindow.arr1.push('Eva');
// 也能直接操作siblings frame的location，所以一定要注意iframe安全
siblingWindow.location.href ='https://evacoder.com';
```

如此看来也没什么特别的，互相都可以操作，但是加上跨域就有所限制了。

#### iframe FAQ

1**.跨域情况下父子frame可以修改互相操作dom么？**

不行，为了安全，父子frame之间都不可互相操作dom，遵循同源策略。

但是如果一定要实现父子通信的功能，就需要另开一篇来说iframe跨域的解决方案了，主要使用 post message，先不展开细说。

2**.跨域情况下子frame可以修改父window的location吗？如何禁止？**

可以，虽然跨域情况下不能dom操作，读取`parent.location.href`都做不到，但神奇的是却可以直接设置parent.location.href，修改父window的location。

在新版chrome会提示需要有用户交互的时候才能触发跳转，只要加上按钮引导用户点击还是可以实现跳转。

总之不是自己管理的页面都不是可信的，为了防止我们内嵌的子页面被坏人篡改或有其他漏洞，我们最好要防止内嵌的iframe修改我们的页面。

可以通过HTML5的新特性`sandbox`属性来限制iframe的行为，这个属性是反向的，如果空字符串则会应用所有的限制，sandbox包含的属性会移除对应的限制。

| "" | 应用以下所有的限制。 |
| :------------------- | :--------------------------------------------- |
| allow-same-origin | 允许 iframe 内容被视为与包含文档有相同的来源。 |
| allow-top-navigation | 允许 iframe 内容从包含文档导航（加载）内容。 |
| allow-forms | 允许表单提交。 |
| allow-scripts | 允许脚本执行。 |

如果不开放allow-scripts权限，iframe中的任何js都执行不了，那会造成功能失效，不是我们的初衷。

其他权限可以按需打开，只要不加上allow-top-navigation，内嵌iframe就无法操作我们的页面

所以我们需要在引入iframe处添加

```js
<iframe src="iframe.html" sandbox="allow-scripts allow-forms">
```

内嵌的iframe就无权限修改我们页面的地址了，如果加上了`sandbox="allow-top-navigation"`则会恢复权限。

但是在实际中大部分 iframe 都没有设置 sandbox 属性。

`allow-same-origin`一看就很有意思，我们将在后文中提到。

插播一个，这种父子 frame 的关系和 window.open()打开的页面也有相似之处，有一个注意点是如果使用`target='_blank'`打开别人的网站，一定要注意别人同样可以通过`window.opener`来控制你的网站

```js
window.opener.location='https://evacoder.com';
```

一般人都不会想到后打开的页面居然可以控制原始页面

为了限制`target='_blank'`打开的页面通过`window.opener`控制父页面，可以加上以下代码，阻止被打开的页面控制原页面。

```html
<a target='_blank' rel='noreferrer noopener' href='https://www.otherpage.com'></a>
// noreferrer 针对firefox，noopener 针对其他
```

3**.如何防止自己的页面被嵌在iframe中？**

在上一篇文章中，我们提到了为了安全禁止了登录页被嵌在iframe中，但是一般更多的是为了预防点击劫持（clickjacking）的考虑。

如果我们有个广告页，需要靠点击量来计费，坏人把我们的页面放在一个透明度为0的iframe中，用户就可能无感知的情况下点击，广告费就猛增。

以前的做法一般是通过js在页面中加入

```js
if (top !== self) {
top.location = location
}
```

就可以保证自己始终是在顶层窗口中显示，就避免了被内嵌在别人的页面中。

但是现在一般都通过服务端直接设置`X-Frame-Options：SAMEORIGIN`响应头来只允许被同域的页面内嵌。

**4.post请求可以跨域么？**

我们常说的浏览器有跨域请求限制，实际上是 ajax 不能跨域，但我们可以用 古老的form 表单来跨域呀，在 iframe 中提交，还可以做到form表单提交而且页面不刷新

```js
<iframe style="display:none">
　<form method="POST"　action="http://api.mi.com">
　　　　<input type="hidden" name="other" value="XXX">
　　　　<input type="hidden" name="money" value="10000">
　</form>
</iframe>
```

**5.可以在iframe的引用页修改iframe元素的样式吗？**

比如我用了google reCAPTCHA的sdk，出来的验证码是通过iframe加载的，我想要修改google的样式。

如上文所述，跨域了是不能进行 DOM 操作的，不然会有严重的安全问题，post message 也只是能在页面间传递信息而已。

**6.iframe 影响了类型判断**

在使用 iframe 时，每个frame 定义的全局变量会自动成为该 frame 中window的全局属性，而每个window对象都包含原生类型的构造函数，window对象都不相等，window.Array 和 window.frames\[0\].Array当然是不等的，所以会出现古怪的bug。

```js
var el = document.createElement('iframe');
document.body.appendChild(el);
var arr = [1, 2, 3];
console.log(arr instanceof Array); // true
console.log(arr instanceof window.frames[0].Array); // false
```

虽然我们一般不会如上面的例子那样去判断arr是不是别的frame下的Array，常用的场景是在iframe中获取parent frame的某个arr来进行类型判断，这样通过 instanceof 来判断对象的class就会出现出人意料的bug。

所以建议使用`Array.isArray(arr)`或者`toString.call(arr)==='[object Array]'`来判断类型。

**7.iframe的使用场景**

iframe 早已成为浏览器的规范实现，所以兼容性其实还可以，可以放心使用。

早年 iframe 经常被用在后台管理系统中的 navbar和 siderbar 的实现，主要是为了复用，现在早就不建议这么做了。

iframe 最大的好处是它一个独立的运行环境，邮箱框架中使用的比较多。

我接触到的有用在无刷新文件上传和登录页定制化（背景图开放给第三方上传管理）。

**8.`target="_blank"`**
今天突然发现引入的google reCAPTCHA iframe里的a链接：“常见问题”点完的效果是直接打开一个tab，原来**iframe不仅能修改父frame的location，还可以打开一个新的tab**，同样是如2中所述，加上sandbox属性可以禁止。


具体可以参考 [知乎：iframe 的好处和坏处](https://www.zhihu.com/question/20653055) 可以了解些历史故事，但是如果有其他替代方案最好还是不要使用 iframe。

参考

[知乎：iframe 的好处和坏处](https://www.zhihu.com/question/20653055)
[Chrome 将默认屏蔽“非用户行为触发”的父页面跳转](https://zhuanlan.zhihu.com/p/38538801)

想到其它有意思的点再补充吧~

今天真的很忙，一整天被呼来唤去的，想起每日一更的约定又难实现，真的要嘤嘤嘤的哭出来，压力大的每天晚上都睡不好，其实也没人逼我，就想试试自己能坚持多久。

可能要求数量的话会导致质量降低，但是有这样的约定挂在心头总比每日毫无追求来的好。

坚持一万小时训练法，等我变成三十岁的老阿姨，一定会有很大的变化。