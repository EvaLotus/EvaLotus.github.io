---
title: 「 缓存 」cookie实践指南
date: 2018-12-20 20:11:15
tags: HTTP
---
HTTP是无状态的协议，cookie可以用来存储用户的信息方便追踪。
<!-- more -->

#### 封装cookie操作

```js
function getCookie(name) {
const cookieName = `${encodeURIComponent(name)}=`;
const cookieStart = document.cookie.indexOf(cookieName);
let cookieValue = null;
if (cookieStart > -1) {
let cookieEnd = document.cookie.indexOf(';', cookieStart);
if (cookieEnd === -1) {
cookieEnd = document.cookie.length;
}
cookieValue = document.cookie.substring(cookieStart + cookieName.length, cookieEnd);
cookieValue = decodeURIComponent(cookieValue);
}
return cookieValue;
}

function setCookie(name, value, expires, path, domain, secure) {
let cookieText = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`;
if (expires) {
cookieText += `;expires=${expires.toUTCString()}`;
}
if (path) {
cookieText += `;path=${path}`;
}
if (domain) {
cookieText += `;domain=${domain}`;
}
if (secure) {
cookieText += ';secure';
}
document.cookie = cookieText;
}

function delCookie(name, path, domain, secure) {
setCookie(name, '', new Date(0), path, domain, secure);
}
```

从setCookie来看，每个参数都暗藏玄机。

* name和value

name和value自不必多说，cookie是存储的一个个键值对，name和value都需要encode之后存储，如果没有encode会怎么样呢？

在chrome中试验`document.cookie='%ddd=2=%#&试试中文'`依然可以设置cookie成功，读取时也可以读取到，但是为了规范存取，还是需要encode之后再读写。

* expires

如果不设置expires，会得到**session级别的cookie**。一定要注意，session的定义是把**整个浏览器关闭**之后，这些cookie才会消失，而不是仅仅关闭所在的tab页。

* path

如果不设置path的话，得到的cookie的path会是当前页面的path哦。比如在`eva.com/blog/page/1.html`中设置cookie，不设置path，得到的cookie的path会是`/blog/page/`

* domain

仔细观察可以发现，cookie中的domain有的是带点的比如`.eva.com`，有的是不带点的`eva.com`。它们的区别是什么？又是怎么设置成功的呢？

`eva.com`和`.eva.com`的区别

cookie也符合**同源策略**，带点的表示子域如`account.eva.com`也可以访问，而不带点的要求严格，只能在本域名中访问。

如果不设置domain的话，就默认是当前不带点的域名。如果设置域名的话，会下发在带点的域名下，子域也可以共享cookie。

比如在`eva.com/blog/page/1.html`中设置cookie，不设置domain，得到的cookie的path会是eva.com，`setCookie('key','value',new Date(),'/','eva.com')`设置了domain的话，会自动加上.变成`.eva.com`。

后端下发的cookie也同理，看是否设置域名。

带不带点需要视情况而定，看是否允许子域获取到cookie。

#### cookie的使用场景

1.登录态的记录

sso登录中需要用cookie来记录用户的登录态

2.已阅读标识

对于只出现一次的用户协议，我们需要一个已阅读的标识，这个时候其实可以不需要后端，前端设置一个cookie来作为已经出现过的标识。

其他待补充

#### 遇到的那些问题

1.cookie大小的问题

有的时候后端为了全面追溯用户的状态下，发的cookie非常大，注意一般浏览器的cookie大小限制是4kb左右，保险起见最好不要超过4kb。过大的cookie会下发不成功，遇到问题不好定位。

有时候运维也会在ngnix对cookie设置大小限制，一般比4kb还小，这个时候问题就更难定位了，可以作为一个排查点。

另外过大的cookie还会造成严重的性能问题。一般没有特别做性能优化的网站js/css/img等静态资源都和接口共用一个域名，域名下如果有很多的cookie的话，每次获取静态资源时这些cookie也会带在了请求中，虽然有4kb的限制，但是还是会造成不必要的网络开销，如果条件允许，建议启动**和主站不同的域名来放置静态资源**。虽然采用不同的域名导致多次建立网络连接，会多耗费时间，可以权衡一下采用哪种方案。

和server端强调下最好不要下发没必要的cookie，注意cookie的大小。

2.cookie标志用户，用户切换时要注意避免cookie污染。

使用cookie进行交互时一定要注意各个用户切换时造成的问题。一定能覆盖成功才可以

3.熟练使用抓包工具避免推诿

chrome中network的追溯

一旦涉及到cookie的问题，特别是webview中的cookie，需要iOS，andriod，FE，server端一起来定位问题，这个时候非常需要一个工具来确定到底谁是罪魁祸首。熟练使用抓包工具会避免推诿，省心不少。

一定要看清楚到底是response cookie和request cookie，明确cookie是在哪个请求中下发的。

抓包工具的使用



