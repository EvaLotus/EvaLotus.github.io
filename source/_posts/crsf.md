---
title: 「 安全 」这次，完全弄明白CSRF
date: 2018-12-20 20:11:15
tags: Security
---
今天听小伙伴分享了下CSRF攻击的原理，这次真的弄明白了CSRF。
**CSRF\(**Cross Site Request Forgery\) 是什么？跨站请求伪造。

通过CSRF攻击，坏人可以冒用你的身份（登录态）来做任何事情。因为登录态一般都是通过cookie来存储在浏览器中，要知道即使只是发起一个图片的请求，也会带上这个域下的所有cookie，可以做到用户完全无感知的情况下发起用户不想发出的请求。
<!-- more -->

#### 为啥钱被转走了

假设mi有个付钱的接口是 `api.mi.com/pay?money=10000&x=xx`

要知道一般登录态（serviceToken）都是存在浏览器的cookie中，有的有效期还很长，如果有一个坏人的页面`badman.com`，页面中加载一个看不见的小图片

```html
<img src=wwww.mi.com/pay?money=10000 style="display:none"/>
```

只要我们访问这个页面，就会在我们不知情的情况下自动发请求 `api.mi.com/pay?money=10000`出去，虽然只是一个图片，一样会带上存在你浏览器中的 `mi.com`下的所有的cookie，如果这时候你已经登录了mi网，这个请求就轻松的得到了你的登录态，向服务器发出了付款的请求。坏人setInterval每秒发个请求，你的钱就哗哗的全被转走了。

一般作恶的核心是**跨域**，所以浏览器对于跨域有很多限制。

但是我们都知道这种通过ping img 图片跨域的方式发起的是**get请求**，只管请求发出去，而不管响应。
get请求一般只是读取操作，像付款这种写操作如果还使用get请求，第一步就被人吃干抹净，所以在接口设计中一定要注意。
现在我们将 `api.mi.com/pay?money=10000` 升级为post请求，如果有坏人访问就统统405，现在就可以高枕无忧了吗？

**post请求可以跨域吗？**

ajax有跨域限制，但是通过form表单的方式，post请求一样可以跨域，再加上放在iframe里，还可以不跳转页面就发送表单post请求，用户毫无感知钱就被转走了。

```html
<iframe style="display:none">
　<form method="POST"　action="http://api.mi.com">
　　　　<input type="hidden" name="other" value="XXX">
　　　　<input type="hidden" name="money" value="10000">
　</form>
</iframe>
```

#### 防御CRSF

* 删改查操作使用post method是基础。
* 一般的做法是随着登录态下发一个 `crsf_token`，在之后所有post操作中都要求带上这个 `crsf_token`，axios有这个功能，不需要我们手动每次添加。 这种做法的原理是在坏人的页面中，虽然可以自动带上被攻击网站的cookie，但是他的代码在 `badman.com` 下，因为cookie的同源策略，他通过js是无法获取在mi.com域下的crsf\_token cookie放在post 请求中的，crsf\_token是个随机字符串都可以达到这个效果。

* 也可以在敏感操作中通过后端校验refer，通过refer白名单的方式来杜绝来自坏人网站的攻击。

* 在非常敏感的操作中可以（最好不要）牺牲一些体验要求验证码验证，要求用户必须有交互。

* 最好严格下发子域的cookie，避免全站通用的cookie。

#### 思考

为啥只在post请求下使用 `crsf_token`呢？

之前回答post和get的区别的时候都回答的很浅，但是在这个场景下，安全无小事，你就能理解两者的区别了。

get请求你要加 `crsf_token`其实也没问题吧，只不过一般get请求是读取操作，不会修改服务器数据，一般没啥影响，不加验证也还好，毕竟加token会影响性能。一个合格的后端开发应该不会用get请求来敏感操作的，如果有，你可以教训教训他。