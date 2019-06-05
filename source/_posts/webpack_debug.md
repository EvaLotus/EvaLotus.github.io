---
title: 「 Problems 」页面引用不同版本webpack打包的js导致的问题
date: 2019-06-03 22:51:35
tags: Problems
---
一次页面引用不同版本webpack打包的js导致的问题。
<!-- more -->

业务反馈接入js sdk遇到报错
> cannot read property 'bind' of undefined

线上代码有压缩混淆，所以先本地代理debug下，`webpack -d`来打包非压缩有sourcemap的版本，在chrome devTool中定位到具体的行数，发现并不是我们自己的代码报错，而是webpack打包过程生成的代码报错：
```js
window.webpackJsonP = window.webpackJsonP || []
window.webpackJsonP.push.bind()...
```
`window.webpackJsonP.push`为undefined，对比sdk执行正常的页面webpackJsonP为[]，而报错业务的页面中webpackJsonP为function，所以是undefined的，怀疑是webpack版本的问题，询问后发现版本确实不一致，google发现有人遇到同样的问题https://github.com/webpack/webpack/issues/6985

#### 问题总结：

如果在一个页面中引用了不同版本webpack打包的js可能会出现莫名其妙的报错，原因是webpack4中`window.webpackJsonP`为数组，而老版本`window.webpackJsonP`为函数。

#### 解决方法：

配置webpack的output.jsonpFunction
```js
module.exports = {
  //...
  output: {
    jsonpFunction:'specialWebpackJsonp'
  }
};
```
我们作为sdk提供方，为了从源头上避免此问题，最好设置特别的output.jsonpFunction