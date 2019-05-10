---
title: 「 微信开发 」深入分析微信小程序安全与管控
date: 2019-04-24 18:27:12
tags: 微信开发
---
#### 思考

写了那么多小程序，你思考过小程序的安全问题么？

为什么小程序支持跨域？

小程序中有xss，crsf攻击吗？

为什么小程序只支持request cookie，不支持response set cookie呢？

为什么小程序不支持动态加载js？什么叫动态加载js？eval和new Function有什么可怕之处？

真的没有绕过小程序审核热更新的方法吗？

小程序的登录设计有哪些安全考虑？

可以肉眼区分一个页面是小程序还是web-view吗？

今天我们就来深入理解小程序的安全设计和管控。

先限定下范围，虽然头条小程序，支付宝小程序，QQ小程序层出不穷，但本文主要从微信小程序说起。

小程序中也支持web-view，内嵌网页这类不在本文讨论范围之中，因为和浏览器是一样的，还是不支持跨域，会有XSS等。
<!-- more -->

#### 分析

**为什么小程序支持跨域？**

我通常和后端小伙伴这样解释，我们可以理解小程序是类似native的，所以不需要考虑跨域问题。

再解释清楚一点就是：小程序的网络请求是通过native的能力来中转后发出的，而不是直接在webview中发出的。

那我们再思考下，**为什么native支持跨域而前端限制那么多？**

个人感觉原因如下：

1.因为前端代码是裸奔状态，用户（或者坏人）输入网址后F12就可以一览无余你的源码，而客户端的代码是编译好的，没那么容易被反编译，看清源码，所以现在webpack打包会生成source map，上传时最好删掉source map文件，否则坏人轻轻松松就能get到你的源码。

2.了解源码之后，前端代码存在各种被注入的可能性，一不小心就location.href到别的页面，一不小心就document.cookie获取走了cookie，或者给你做个假的登录页窃取你的密码，可以参照文章：万物皆可hook

3.浏览器tab页的存在，用户身份通常通过cookie来存储，即使tab页之间没有关系，坏人页面依然有办法窃取你的cookie。攻击形式太多样了，非常需要同源策略（跨域）的限制。

既然说到了cookie，小程序中有个重要的限制就是**request中可以支持cookie但是response中不支持cookie**，这又是什么考虑呢？

我们知道，在浏览器中发出请求时会自动携带该域名下的cookie，同域名的js，img，css请求都会带上cookie，但是因为小程序的网络请求是通过native来转发的，并不是之前常用的ajax，实际上我们在请求中必须手动在header中设置cookie。

```js
wx.request({
  method: 'GET',
  url: 'evacoder.com/users',
  data: {},
  header: {
    'content-type': 'application/x-www-form-urlencoded',
    'cookie': 'userId=12345'
  },
})
```

请求是native代为转发，那接受请求的实际是native，处理后再传递给webview，wx.request成功回调中没有返回cookie，也直接禁用了DOM，BOM相关的所有方法，document.cookie根本无法调用，没处理cookie微信爸爸不提供咱也没办法。

发送cookie必须手动设置，获取不到cookie，小程序中应该是**不存在CSRF**了。

当然还有更严格的限制，**要调用接口必须在该域名下放校验文件**，所以不经过同意别人的小程序没法调用你的接口，在你的小程序坏人也没法把窃取到的信息发到他的域名。

**小程序是否存在xss呢？**

xss即js代码注入，前端要做的是不信任任何用户的输入，将所有的特殊字符如&lt;&gt;/%都进行过滤，尽量不要使用eval，innerHTML。比如在博客留言中留下`<script>location.href='https://badman.com'</script>`后，这样每个访问这个页面的人都会被导流到坏人的页面（或者其他被窃取cookie等造成损失）。

博客评论及论坛帖子这种富文本编辑器一向是xss的重灾区，在小程序中呢？

我觉得总会有些小白不过滤吧，试了几个没啥用户的个人小程序，填写`<script>location.href='https://badman.com'</script>`果然可以提交成功，重新进入这个页面并没有得偿所愿。

本来认为小程序最终也是编译成了html，那获取到的数据中的&lt;script&gt;确确实实是存在的，难道有啥奇妙之处？

其实原因很简单，数据绑定模版语法大多类似，普通的`{{msg}}`反显时使用的应该是**innerText而不是innerHTML**，所以即使没有过滤，反显的时候也依然无法注入成功。

但是还是要注意，小程序中如果忘记过滤了，**可能在网页中接口返回后XSS就可以成功了**，也是一个攻击点，所以前后端都要好好过滤。

小程序中也提供了富文本 [rich-text组件](https://developers.weixin.qq.com/miniprogram/dev/component/rich-text.html)，但是明确规定了只接受受信任的标签，当然是没有`<script>`的。

但是极端一点如果小白把用户的输入eval了怎么办呢？

不用担心，小程序也**不支持eval和new Function动态加载**。

其实当然不是为了防止小白犯傻，更重要的是为了防止有人来进行热更新动态加载绕过审核，整个小程序是个壳，通过请求来返回非法内容把小程序改的面目全非。但是看到这篇 [懂编译原理真的可以为所欲为](https://zhuanlan.zhihu.com/p/34191831) 貌似还是有方法可以绕过。

就目前来看，常见的xss已经被封死，可能还有没想到的欢迎补充。

**有办法肉眼区分一个页面是小程序还是web-view吗?**

可能网页在性能差的手机上渲染的慢一点，不要提肉眼区分，服务端都没有好办法直接判断。因为iOS的ua不规范，根本无法区分是小程序还是微信的浏览器。（所以说一开始多花点时间讨论再开工，不然要累死客服啊），请大家补充。

**小程序编译完之后变成了什么？**

编写小程序是使用类似HTML，JS，CSS的WXML，WXSS，JS来编写的，因为RN不稳定等原因，小程序最终没有选择类似RN的采用客户端原生渲染，还是采用了Web技术渲染，加上原生接口来提供原生能力，所以小程序最终还是打包成HTML运行在浏览器中，浏览器不识别小程序语言，所以需要一个编译过程，大概就是有wxml-loader，wxss-loader通过webpack打包成pp-service.js后，插入在一个page-frame的html中。

我们可以抓包看到小程序的refer是[https://servicewechat.com/{APPID}/{VERSION}/page-frame.html。](https://servicewechat.com/{APPID}/{VERSION}/page-frame.html。)

小程序的写法和vue相似，但是vue中js，html，css可以写在一个vue文件中，但是小程序中必须拆分为三个文件，会不会有些多余呢？

实际上这正是小程序的安全设计的用心之处，**区分视图层和逻辑层，采用双线程的设计**。（其实也可以一个页面，编译时做拆分即可，不过三个页面也更灵活）。

官方提供的编译过程

```js
// 【WXML】编译后得到以下函数，根据路径和数据自动生成virtualDom，然后通过组件系统比对差异渲染页面
function $gwx(pagePath) {
  // ...
  return function(pageData) {
    // ...
    return {};
  }
}

// 根据页面路径获取页面结构生成函数
var generateFun = $gwx('name.wxml')

// 页面结构生成函数从【js】接受页面数据，得到描述页面结构的JSON
var virtualTree = generateFun({
  name: 'Eva',
  age: '6'
})

/* virtualTree == {
  tag: 'view'，
  children: [{
    tag: 'view',
    children: ['name:Eva', 'age:6']
  }]
}*/

// 小程序组件系统在虚拟树对比后将结果渲染到页面上
virtualDom.render(virtualTree)
```

视图层通过WebView浏览器环境，逻辑层通过JsCore来执行。

**视图层**包括WXML和WXSS，使用**WebView**加载page-frame.html，在确定页面路径之后，WXML和WXSS文件生成了JS函数generateFun，结合逻辑层的页面数据得到virtualTree，小程序组件系统在虚拟树对比后将结果渲染到页面上。

**逻辑层**中包括所有JS代码，主要是页面的数据，和点击事件等处理，所有JS编译成appService.js。

初次渲染：逻辑层将pageData和pagePath等通过Native传给视图层来渲染页面。

更新数据：逻辑层发送数据给视图层

用户事件：视图层收到用户事件根据绑定事件回调函数来反馈给逻辑层

为啥要区分逻辑层和视图层呢？

> 基于Web 技术来渲染小程序是存在一些不可控因素和安全风险的。这是因为Web技术是非常开放灵活的，我们可以利用JavaScript 脚本随意地跳转网页或者改变界面上的任意内容。

开发者可以做的事太多了，比如现在用户授权必须用户手动点按钮，但是如果可以DOM操作开发者直接就能跳过这一步获取用户敏感信息了。而且会存在前面提到的很多的安全隐患。

逻辑层使用的JsCore只是一个单纯的脚本解析器，浏览器中的BOM对象无法使用，正好可以完美解决这个问题。开发者工具并不是使用了真正的JSCore，而是直接用一个webview，有个很巧妙的设计，将开发者的代码包裹在define域的时候，**将浏览器的BOM对象局部变量化**，从而使得在开发阶段就能发现问题。

如果在渲染层写了script代码，在webpack处理时因为不符合规范应该会直接被过滤，或者会报错无法执行。

如果在逻辑层写了操作DOM的方法，因为JSCore环境下不识别，所以也会报错。

#### 官方论证

其实上面的问题官方都公布了答案：[深入理解小程序架构](https://developers.weixin.qq.com/ebook?action=get_post_info&token=935589521&volumn=1&lang=zh_CN&book=miniprogram&docid=000e84889907c00b0086b0a2f5b40a)。

这里真的有很多宝藏，快去探索吧。

通过上文的分析，小程序通过这些限制来保证了安全和管控：

1.逻辑层和渲染层的双线程架构

2.不支持eval和new Function动态加载

这篇文章的篇幅已经很长了，关于小程序的登录安全设计，我们将在下篇文章中介绍。
