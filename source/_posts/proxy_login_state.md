---
title: 「 效率 」前后端分离项目的本地代理及登录态绕过
date: 2019-01-05 12:44:27
tags: Tool
---
在开发新需求或者复现bug时，前端经常会通过代理工具将被本地修改的js，css代理到线上，这样可以不用一次一次的上线测试，在本地就可以随改随生效。

vue项目可以 `npm run serve` 本地跑起来，开发阶段接口调试时，最好能把所有localhost的本地接口全代理到【线上】，有很多方法。

<!-- more -->

#### 绕过跨域限制的方法
1. 使用vue-cli中默认带的http-proxy-middleware，具体配置参见[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware) 文档。

2. 使用代理工具，fiddler 或 Charles设置cors。

3. 使用chrome插件Allow-Control-Allow-Origin: \*

4. 让后端开启cors（尽量自给自足）。

#### 如何绕过登录

但是遇到登录这个拦路虎，localhost下因为没有登录态，每次请求全部401，不能本地调试，代理完全都没用了。

后来同事分享了一个鸡贼的方法，真是个小机灵鬼~

以evacoder.com为例，因为登录态通过cookie来标志的，而此cookie下发在`evacoder.com`域下

怎么才能让localhost共享到登录态呢？

比如我npm run serve在localhost:8080，此项目需要登录态，我们可以配置host

```
127.0.0.1 test.evacoder.com
```
如果登录态的cookie不带点，也就是说子域不可访问，我们可以手动的chrome dev tool=>Application=>cookies下给它加个点。

这样我们测试的时候在evacoder登录之后下发了登录态，因为有代理，访问localhost:8080可以改为访问test.evacoder.com:8080就可以共享到登录态了。

少年你忘了还有丑陋的8080端口号吗？端口号不一样也是跨域的，那咋办呢？

我们还有大招nginx反向代理，nginx用处多多，此处只说怎么完成我们的需求

```bash
server {
    listen       80;
    server_name  localhost;
    location / {
        proxy_pass    http://127.0.0.1:8080;
        root   html;
        index  index.html index.htm;
    }
    ...
}
```
将localhost:8080代理到localhost:80端口，80端口是默认端口，这样就可以不需要丑陋的端口直接访问了。

#### 简化
为啥要这么麻烦呢？平常调试时不也就是用fiddler把线上的文件直接代理到本地就可以了么？

因为 `npm run serve` 生成的文件放在内存中，并没有实际的路径，但是我们还是可以使用 fiddler 通过autoresponse中使用正则来完成批量代理

只需将 `REGEX:https://evacoder.com/(.*)$` 代理到 `http://localhost:8080/$1` 即可。当然我们需要只代理静态文件，剔除api的代理。

虽然这样webpack HMR就没办法直接本地即改即生效了，但是我们还有办法，sublime上的livereload插件配合chrome上的livereload，保存文件时无脑刷新，虽然傻办法，但是还是可以节省点时间。

这样在开发过程中也可以开心的代理到线上即改即生效了~虽然功能很简单但是探索的过程很有趣，社会的发展都是懒人推动的哈哈哈。