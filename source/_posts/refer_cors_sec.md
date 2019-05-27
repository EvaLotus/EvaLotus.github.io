---
title: 「 安全 」从 referrer 白名单说到CORS攻击点
date: 2019-03-19 18:27:12
tags: Security
---
看到一个抢红包的活动页，准备把接口copy出来自动测试却发现403了，这种情况一般都是因为有 `referrer` 校验。我们也经常会通过 `referrer` 白名单来限制接口的盗用，今天来分析下可能存在的攻击点。

<!-- more -->
####  `referrer` 为空的情况

 `referrer` 本来的作用是判断当前页面的来源页面的地址，通常用来统计分析，日志记录，或者可以记录下帮别人导流数量来收钱，但是在页面跳转中也存在没有  `referrer`  的的几种情况：

1.来源页面采用的协议为表示本地文件的 `file` 或者 `data` URI

2.当前请求页面采用的是非安全协议，而来源页面采用的是安全协议\(HTTPS跳到HTTP\)

3.直接在浏览器地址输入地址进入的页面，或`location.reload()`刷新会没有 `referrer` ，`location.href`或者`location.replace()` 有 `referrer` 

4.代码设置禁止发送：`<a rel="noreferrer" href="https://evacoder.com/users">`，`<meta content="never" name="referrer">`

前端虽然可以通过 `document.referrer` 获取 `referrer` ，但为了安全JS是无法修改 `referrer` 的。

那是否还有可攻击点呢？

当然！如果要进行 `referrer` 校验，有的接口因为需要客户端通用，考虑到客户端请求不带 `referrer` ，server端有时会放过 `referrer` 为空的情况，这个时候坏人就可以利用上面的规则构造出 `referrer` 为空的情况。

```html
<html>
  <body>
    <iframe src="data:text/html;base64,PGZvcm0gbWV0aG9kPXBvc3QgYWN0aW9uPWh0dHA6Ly9hLmIuY29tL2Q+PGlucHV0IHR5cGU9dGV4dCBuYW1lPSdpZCcgdmFsdWU9JzEyMycvPjwvZm9ybT48c2NyaXB0PmRvY3VtZW50LmZvcm1zWzBdLnN1Ym1pdCgpOzwvc2NyaXB0Pg==">
  </body>
</html>
```

base64解码下

```js
atob('PGZvcm0gbWV0aG9kPXBvc3QgYWN0aW9uPWh0dHA6Ly9hLmIuY29tL2Q+PGlucHV0IHR5cGU9dGV4dCBuYW1lPSdpZCcgdmFsdWU9JzEyMycvPjwvZm9ybT48c2NyaXB0PmRvY3VtZW50LmZvcm1zWzBdLnN1Ym1pdCgpOzwvc2NyaXB0Pg==')

=>

"<form method=post action=http://a.b.com/d><input type=text name='id' value='123'/></form><script>document.forms[0].submit();</script>"
```

利用上文的第1条：协议为data开头，如果这个HTML页面向任何站点提交请求的话，这些请求的 `referrer` 都是空的。

或者利用第4条，强制设置不发送  `referrer` 

```html
<meta content="never" name="referrer">
<a rel="noreferrer" href="https://evacoder.com/users">
```

既然存在作恶的空间我们就最好不要放过 `referrer` 为空的请求，对于客户端通过UA来过滤放行。

####  `referrer` 的正则攻击点

那不允许 `referrer` 为空就不存在安全问题了吗？ `referrer` 的校验依然有很多可绕过的地方。因为无法看到后端的 `referrer` 校验规则，我们只能碰碰运气，常见的问题如下：

如果已知在evacoder.com/demo中是可以通过 `referrer` 校验的，我们试着在抓包工具中修改 `referrer` 为以下：

1.https://www.badmanhackevacoder.com/demo  （可能只校验了`/^.*evacoder.com/demo$/`）

2.https://www.evacoder.badman.com/evacoder.com/demo   (可能只校验了`/^.*evacoder.com/demo$/`,可以构造一个对应的文件夹来绕过）

3.https://badman.com?id=evacoder.com/demo （构造参数）

#### CORS中的类似攻击

我们常说AJAX有跨域限制，但是很多人没有注意到即使跨域，AJAX其实也已经将请求发出去，只不过因为浏览器限制，JS获取不到响应而已，类似ping img。为了解决AJAX跨域的问题，现代我们通常使用CORS来跨域资源共享。

IE10以上都支持CORS，放心使用，只需要后端配置允许CORS，前端啥也不用改就可以直接获取到跨域的请求，我们可以视情况在IE10以下使用JSONP来兼容。

node配置示例

```js
app.use('*',function (req, res, next) {
  // 任意域名都可访问此接口，不能携带cookie
  res.header('Access-Control-Allow-Origin', '*');
  // res.header('Access-Control-Allow-Origin', 'http://www.baidu.com'); // 只有www.baidu.com 可以访问。
  res.header('Access-Control-Allow-Headers', 'Content-Type, Content-Length, Authorization, Accept, X-Requested-With , yourHeaderFeild');
  res.header('Access-Control-Allow-Methods', 'PUT, POST, GET, DELETE, OPTIONS');// 允许的请求方法
  if (req.method == 'OPTIONS') {
    res.send(200); // 在正常的请求之前，会发送一个验证，是否可以请求。
  }
  else {
    next();
  }
});
```

上面说到 `referrer` 因为正则校验不严格，所以容易出现可以绕过的问题，在CORS中一样存在。

对于CORS为\*的，我们不必关心，它已经放弃了限制，你随便来调用吧，要是有笨到将敏感接口如此设置的，应该会被直接开除吧。

但是假设我们F12突然发现 [https://www.evacoder.com](https://www.evacoder.com) 中有个接口居然设置了CORS，

CORS请求成功之后，在response header中会有以下请求头，比如：

```
Response Header
Origin: https://www.evacoder.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Headers: Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,token
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Origin: https://www.evacoder.com
```

这里的CORS只在指定的某些域名可以使用，同样仅在前端是看不出来在后端设置的具体域名的。

我们来碰碰运气，在抓包工具中修改Origin为以下：

1.[https://www.badmanhackevacoder.com](https://www.badmanhackevacoder.com) （可能只校验了`/^.\*evacoder.com$/`）

2.[https://www.evacoder.badman.com](https://www.evacoder.badman.com) \(可能只校验了`/^.\*evacoder..\*.com$/`）

去测试吧，可能会碰到漏网之小白，再找到敏感接口，将 [https://www.badmanhackevacoder.com/testPage](https://www.badmanhackevacoder.com/testPage) 发给用户，用户可能已经登录过evacoder.com，坏人页请求敏感接口成功，就能获取到小白在evacoder的敏感信息了。

CORS可攻击点的范围比 `referrer` 要稍小一点，CORS只能构造假的domain， `referrer`  还可以构造参数，可发挥的空间更大。

虽然一般是server端referer和CORS白名单，但是作为对浏览器最了解的前端，为server端提供安全建议也是我们应该做的哦~

[CORS详细介绍](http://www.ruanyifeng.com/blog/2016/04/cors.html)
