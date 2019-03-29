---
title: 「 基础设施 」前端异常监控系统
date: 2019-02-19 19:37:12
tags: 前端
---
遇上有人反馈bug你一般都怎么处理？

<!-- more -->
#### bug反馈

先看看内网下自己的设备是否有报错，定位是线上问题or内网问题，内网报错：回滚内网分支。

线上报错：查看是普遍问题还是兼容性问题

* 普遍问题着手fix
* 兼容性问题，询问用户设备信息，使用模拟器或者找测试同学使用相同测试机来复现，可以复现之后，如果是pc端可以f12来查看报错信息并定位（熟练使用devtool，最好有sourcemap来直接定位），移动端则使用远程调试（remote debugging 等在chrome或safari中进行复现，并定位问题））

从上面我们可以发现，测试虽然测了大部分机型，但是js裸奔在各式各样的设备上，你真的能放心吗？

反馈问题我们需要：1.设备信息，2.报错提示，3.复现流程

内部反馈还好，如果是小白用户，根本不知道如何F12，更不要提移动端的错误调试，用户反馈到的很有可能就是一句（点了没反应，白屏之类的没有太多用处的信息）。

最近抛弃了ie8，各种新技术都可以用起来了。虽然使用webpack，babel已经很成熟了，也封装的越来越好，

一行browserlist&gt;1%就可以保证所有的兼容性。但是难免还是有点心慌，antd圣诞彩蛋自己被辞了也没办法找他们要赔偿啊？

虽然有内网灰度，等着别人反馈问题？周期很长的，灰度3个月你受得了？

这个时候我们真的非常需要一个前端的异常监控系统，复现流程先不奢求，只要有设备信息，js报什么错，具体的行数，一般也可以定位到问题了。

#### 异常监控系统的原理

如果让你自己实现一个异常监控系统你会怎么做？

最核心的API就是window.onerror，触发window.onerror的时候，将所有获取到的错误信息都发送一个请求到server端。

```js
window.onerror = function (msg, url, lineNo, columnNo, error) {
  return false;
};
```
当代码中的 promise reject 的时候，onerror 是捕获不到异常的。对于 promise reject 的异常，除了对每个用到 promise 的地方都加上 catch 之外，我们还应该在全局环境下进行一个兜底。

我们可以监听全局 unhandledrejection 事件：
```js
window.addEventListener('unhandledrejection', (e) => {
  console.log(e)
})
```
如果要阻止异常输出到控制台上，可以加上 e.preventDefault()。
通过发送请求还可以获取到设备的UA信息，ip地址。

但是线上运行的代码都是打包压缩的，报错信息并不具体，不好定位，我们需要生成sourcemap文件便于debug。

TODO sourcemap的原理。

#### sentry的使用

一般公司也不会自建异常监控系统，现在有很成熟的方案，这里推荐使用sentry。

接入只需要：

```js
<script src="https://browser.sentry-cdn.com/4.6.4/bundle.min.js" crossorigin="anonymous"></script>
<script>
  Sentry.init({
    dsn: 'https://57b032907e35480d991114d212fcdda4@sentry.evacoder.com/2',
    ignoreErrors: [],
    release: "<%= htmlWebpackPlugin.options.sentryRelease%>",
  })
</script>
```

或者使用npm直接安装也行，使用类似。

不用担心sentry的兼容性，因为sentry也是hook了window.onerror，而window.onerror所有浏览器有标准实现。

咋hook你懂的，可以复习下我写的这篇 [万物皆可hook](https://www.evacoder.com/2019/02/15/iframe_security/)。

但是需要注意的是我们的老朋友跨域，js报错也是会跨域的=\_=\#，&lt;script&gt;引入的不同源js中的错误，window.onerror只能捕获到Script error，需要在script标签添加`crossorigin`属性，并配置CORS。

iframe，css，img这些标签捕获不到错误。

参考文档可以通过docker来部署sentry
```bash
git clone https://github.com/getsentry/onpremise.git
```
按照github上的步骤一步步部署

遇到其他问题去看[文档](https://docs.sentry.io)，不会看文档还做程序员是害人害己哈哈哈。
