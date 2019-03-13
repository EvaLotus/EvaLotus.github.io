---
title: 「 微信开发 」前言
date: 2018-09-18 18:23:29
tags: 微信开发
---
现在经济不景气，小公司拿不到投资，连iOS开发和andriod开发都招不起，而微信凭借着众多的用户，成为了开发的热土，一个创业公司可以没有App，但一定会需要先在微信里试水，是公司获客的最佳渠道。

这个系列记录了微信开发遇到的问题，和产品狗相爱相杀的往事。

<!-- more -->

目录（也是产品狗的那些呐喊）：

0.开发前的了解

1.我想要在微信的H5页面里直接打开小程序

2.我想要做活动页分享给好友

3.我想小程序和webview的登录态互相传递

...

#### 关于H5的那些争执

H5，在我们大前端眼中实际是HTML5的缩写，其实HTML已经是缩写了，缩完再缩，就变成了他们口中的H5，实际上只有中国人才会这么称呼，要是跟老外说H5，H five，他们就懵逼了。

比如App，并不是任何词组的缩写，而是Application的简读，只有中国人才会读APP。还有那个UGG的鞋子，实际上应该念‘阿哥’，但是呢，你开心就好。嗯，啥时候出一个程序员英语装逼指南应该会更受欢迎吧。

说回H5，对于FE来说，HTML5只不过是一种语言而已。

对于PM还有其他非前端的开发人员来说，H5指的是手机端上的网页。

对于运营妹子来说，嗯，H5呀，我也会做哦。实际上他们说的是，可以拖拖拽拽，像做PPT一样生成的页面，便于在微信里快速传播。

虽然他们说的都不对，但他们开心就好，能听懂就好。

#### 关于微信公众平台和开放平台

公众平台：[https://mp.weixin.qq.com/](https://mp.weixin.qq.com/) 主要用在公众号，运营妹子登的多，我们需要开发微信公众号的一些活动页面时也需要在里面配置一些东西

开放平台：[https://open.weixin.qq.com/](https://open.weixin.qq.com/) 主要用在各种开发上，app，小程序，公众号都包括

#### 开发公众号页面的配置

有的公号后台都是运营或PM来管，都不让我们FE来登录。下面是我们开发必要的一些信息。

![](/images/微信公众平台-基本配置.png)

##### 首页=&gt;开发=&gt;基本配置

###### 公众号开发信息

AppId用在很多地方

AppSecret只能放在服务端，前端代码对攻击者来说完全是透明的，只要是跟AppSecret相关的接口都一定是后端调微信的接口。

需要添加你的IP在Ip白名单里，这样才可以获取acces\_token来测试

###### 服务器配置

不要配置这个，以免运营妹子的自动回复没了

###### 已绑定的微信开放平台帐号

有用到unionId的话会需要这部分

##### 首页=&gt;开发=&gt;开发者工具

![](/images/微信公众平台-开发者工具.png)

下载一个微信开发者工具

开发者工具相关信息：[https://mp.weixin.qq.com/wiki?t=resource/res\_main&id=mp1455784140](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1455784140)

关注公众平台安全助手

让管理者开通公号开发者权限

基本工作完成就可以用开发者工具调试公号页面啦

#### 完成需求的步骤

1.当PM抛出一个需求时，我们应该看看**当前的公众号是否有相应的权限**。

不要设计接口、讨论开发了半天才发现这个公号根本没有这个接口权限。

登录微信公众平台=&gt;开发=&gt;接口权限

或者综合的接口权限说明： [https://mp.weixin.qq.com/wiki?t=resource/res\_main&id=mp1433401084](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1433401084)

选择设置=&gt;系统代理时，fiddler代理并不能生效。可以选择手动设置代理，ipconfig获取本机ip，ip:8888。

##### 订阅号和服务号的区别：

简言之，订阅号会被收纳起来，服务号推送消息会直接出现的你的消息列表中

订阅号比如：咪蒙。。。这类写文章的。

服务号比如：小米商城，招行信用卡，京医通等等。

其他的区别参见：
[https://www.zhihu.com/question/21289814](https://www.zhihu.com/question/21289814)

2.思索一下在生活中有没有看到过类似的别人完成的需求。

PM也是人，他们设计肯定也是看别人发了啥他们也要，他见过的咱们也肯定见过啊，没有见过就让他给找例子。

3.在chrome中打开链接，F12看下源码

反正咱们前端都是小透明，如果运气好没压缩的话，很easy就可以看懂了。

其实最大的问题是刚开始因为刚上手，不太明确前端和后端的职责，看看别人页面的network就可以胸有成竹的指使后端我需要这个你得给我返回啦~

有套路，不迷路。

#### 经常需要查看的文档们，google不如看文档：

公众平台综合技术文档：[https://mp.weixin.qq.com/wiki?t=resource/res\_main&id=mp1472017492\_58YV5](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1472017492_58YV5)

JS-SDK：[https://mp.weixin.qq.com/wiki?t=resource/res\_main&id=mp1421141115](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)

JS接口签名校验工具：[https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign&token=⟨=zh\_CN](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign&token=&lang=zh_CN)

接口调试工具：[https://mp.weixin.qq.com/debug?token=1156200581⟨=zh\_CN](https://mp.weixin.qq.com/debug?token=1156200581&lang=zh_CN)

获取JSTicket：[https://api.weixin.qq.com/cgi-bin/ticket/getticket?access\_token={Access\_TOKEN}&type=jsapi](https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token={Access_TOKEN}&type=jsapi)
