---
title: 微信开发实用指南3-手机浏览器唤起微信app登录的可行性
date: 2018-10-12 18:26:01
tags: 微信开发
---
微信的地位越来越高，隔不了多久就有大佬问，为什么你们的手机端H5没有微信登录的入口，压力很大呀。

可能是安全问题或者其他考虑，微信在网页端只提供了扫码登录的入口，即使在手机端集成了微信登录，用户点开也会很奇怪，这个码只有PC端的样式，看起来很奇怪，而且怎么用我的手机微信来扫这个手机浏览器这个码呢？而且即使可以扫码成功也无法登录成功，所以看了大部分的公司都选择在手机浏览器隐藏了微信登录的入口。

但是细心的PM发现，京东居然有微信登录的入口，但是一般也只是在QQ浏览器或者系统浏览器中支持，可以直接打开手机上的微信app，但是在无痕模式或者其他的浏览器也无法成功。

前端无隐私，让我们来F12来看看他们是怎么做到的。

![](/assets/wap_wxlogin.png)

可以发现，京东的微信登录并没有什么特别的，也是各种302跳转最后到了 [https://open.weixin.qq.com/sns/explorer\_broker?appid=wx2f5d8f9715c59d10&redirect\_uri=https%3A%2F%2Fplogin.m.jd.com%2Fcgi-bin%2Fml%2Fwxcallback%3Flsid%3Dq9ibchzrnbivxypneji7r4sxusp24wrv&response\_type=code&scope=snsapi\_userinfo&state=smr1qm3p&connect\_redirect=1 ](https://open.weixin.qq.com/sns/explorer_broker?appid=wx2f5d8f9715c59d10&redirect_uri=https%3A%2F%2Fplogin.m.jd.com%2Fcgi-bin%2Fml%2Fwxcallback%3Flsid%3Dq9ibchzrnbivxypneji7r4sxusp24wrv&response_type=code&scope=snsapi_userinfo&state=smr1qm3p&connect_redirect=1的)这个页面。最重要的是这个是微信的域名，所以我们可以知道，这个跳转是微信主动触发的。

怪不得说京东是微信亲儿子嘛。让我们看看这个页面做了什么事。

![](/assets/wap_wxlogin2.png)前面都是一些环境的判断，而且这些都是微信方帮忙实现的。最重要的是`location.href = 'weixin://dl/business/?ticket=tb15388ef1a6e07db69a0d57a87345f93'`

前端是可以实现打开微信app的，但是如何打开某个特定的页面就需要app端来支持了，而微信端为京东做了特殊的处理，可以做到直接打开微信的app中京东的微信授权页面。

所以如果我们的业务也需要实现这个功能，就需要向微信申请了。

另外一个更激进的就是QQ了，公告说是为了一些不安全因素，在18年年初完全下掉了wap端网页登录的方式，现在wap端的QQ登录都采用直接打开QQ app的方式来授权登录。在某些非系统浏览器中因为无法打开app就悲剧了。还是觉得有些激进了些。