---
title: 「 缓存 」localStorage实践指南
date: 2019-2-20 20:11:15
tags: JS
---
记录localStorage在实践中遇到的具体问题。

#### localStorage和cookie的区别
1.大小：
localStorage可以存储大约5M的数据，cookie一般只能存储4kb。
两者都有大小限制，不同浏览器单次设置的大小和总条数会有差异，需要做好超过限制之后的处理

2.兼容性：
cookie由来已久，几乎所有浏览器都支持，兼容性更好
localStorage支持>=IE8，还有些特别的情况下兼容性不好：
ie11下用户保护模式下调用`window.localStorage`直接报错
禁止cookie时调用`window.localStorage`直接报错，但`document.cookie`不会报错
safari无痕模式下cookie是可用的，localStorage存在但是setItem时会报错(chrome的无痕模式localStorage和cookie都可用，只不过关闭浏览器之后会清除)


3.特性：
cookie在每次请求中都会携带，服务端可以CRUD cookie，但是localStorage只是种存储方式而已
cookie必须url encode
cookie可以设置如下属性：
- HTTPOnly：避免被js操作保证安全
- expire：有过期时间
- secure：必须在https下传输
- path：不同页面可以设置不同path避免多个tab之间数据污染
localStorage没有这些功能，只有key，value设置，需要约定好避免存储的key，避免数据污染


4.持久性：
浏览器缓存中小白用户可能也知道怎么清除cookie，但是localStorage被清除的概率会低一些，所以可以使用LS来存储设备指纹

5.跨域限制
sessionStorage不能共享，localStorage在同源document间共享，cookie在同源且符合path规则的文档间共享


#### localStorage的使用注意
1.安全问题
一旦有xss漏洞，有坏人可能将恶意代码注入到localStorage中，导致即便修复了xss恶意代码也存在的问题。 所以遇到这种情况一定要注意发新版清除用户端的localStorage。

2.为何静态资源JS/CSS不存在localStorage中
- SEO：之前本站在robots.txt中禁止抓取assets目录（包括CSS和JS等），导致google搜索中出现页面不适合移动端访问的提示，影响了SEO。如果将JS/CSS存储在localStorage中可能会导致爬虫抓到的页面错误。
- FOUC:如果先输出HTML然后再从JS中读取CSS加载，会造成页面闪烁，体验很差
- PC和移动端的考虑：PC端网速比较快，浏览器兼容性没那么好，使用的意义不大
- 真的有必要吗：如果JS和CSS等静态资源设置合理的缓存和过期时间，localStorage读取并执行可能并不如304协商缓存快

3.不兼容情况下的处理

safari无痕模式下localStorage和sessionStorage本身是存在的，只是setItem会报错

有些浏览器不支持storage或者用户设置禁止了localStorage，调用`window.localStorage`时都会直接报错

Sentry上大量抓到这样的错误
>Failed to read the 'localStorage' property from 'Window': Access is denied for this document.
兼容性还是要考虑好，最好使用try catch来判断
```
let isSupportStorage = false;
try {
  localStorage.setItem('testStorage', 1);
  isSupportStorage = true;
} catch (e) {
  // 无痕模式或者不支持storage
}

```
#### localStorage和sessionStorage的区别

sessionStorage存储数据只在本次会话有效，数据仅在当前窗口有效，即不能跨tab

