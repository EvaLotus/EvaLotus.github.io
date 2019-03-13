---
title: 「 puppeteer 」从puppeteer谈前端爬虫检测和绕过
date: 2019-02-14 18:20:35
tags: puppeteer
---
检测Headless Chrome/ webdriver/ selenium/ puppeteer

爬虫检测是一个攻防的过程，js是裸露在外的，在坏人手里的，我们只能想办法提高作恶的成本。

各处搜集整理信息了很久，后来发现 [这篇文章](https://intoli.com/blog/not-possible-to-block-chrome-headless/) 讲的非常清楚，本地化一下，加上一些自己的理解。

phantom，webdriver，puppeteer等爬虫都有设置的方法，本文主要从puppeteer来分析。

<!-- more -->
#### 1.navigator.userAgent
防：`/HeadlessChrome/.test(window.navigator.userAgent)`一般判断ua中有headless字样都没跑了，肯定是爬虫。
攻：puppeteer可以设置ua，甚至直接设置device。

```js
await page.setViewport(conf.viewport);
await page.setUserAgent(conf.ua);
await page.emulate(conf.device);
```

#### 2.检测webdriver标志字段

防：检测window object中是否有任何`selenium/webdriver/$cdc_/$wdc_`等字样。

以下是搜索到的字段标志，只测试了在正常访问状态下并没有这些字段。

```js
// webdriver keywords
function detectWebDriver() {
	const r = [];
	const w = ['webdriver', '__driver_evaluate', '__webdriver_evaluate',
	' __selenium_evaluate', '__fxdriver_evaluate', '__driver_unwrapped',
	'__webdriver_unwrapped', '__selenium_unwrapped', '__fxdriver_unwrapped',
	'_Selenium_IDE_Recorder', '_selenium', 'calledSelenium',
	'_WEBDRIVER_ELEM_CACHE', 'ChromeDriverw', 'driver-evaluate',
	'webdriver-evaluate', 'selenium-evaluate', 'webdriverCommand',
  'webdriver-evaluate-response','__webdriverFunc', '__webdriver_script_fn',
  '__$webdriverAsyncExecutor', '__lastWatirAlert',
	'__lastWatirConfirm', '__lastWatirPrompt', '$chrome_asyncScriptInfo',
  '$cdc_asdjflasutopfhvcZLmcfl_', '_phantom', '_phantomas'];
	w.forEach((t) => {
		if (!!window[t] || !!window.document.documentElement.getAttribute(t) || !!navigator[t]) {
			r.push(t);
		}
	});
	return r;
}
```

人力限制不一个个测试了，不放心的话前期先记录下命中词，之后可以只要遇到命中词就拉黑拒绝提供服务。

攻：使用puppeteer会检测到webdriver，navigator.webdriver为true。

试着加上了`navigator.webdriver=false`实际上并不会生效，console出来依然是false。

`const isAutomated = navigator.webdriver`

这个字段就是标志了一切自动化。

`navigator.userAgent='test'`设置了再console也是无效的。

实际上可以使用Object.defineProperty来修改原生对象

```js
await page.evaluate(async () => {
  Object.defineProperty(navigator, 'webdriver', { get: ()=> false });
});
```

但是对于反爬方来说，一般在你进入页面的时候就已经开始检测是否是爬虫，立即停止服务了，在puppeteer中篡改已经晚了。

这种时候可以通过**代理工具**，fiddler，charles都可以完成js的注入。

* 在检测爬虫的js前加上以上的篡改js。
* 或者先在正常状态下分析下源码具体检测爬虫的代码，使用代理工具，去掉识别webdriver的js（或者直接返回不是爬虫）之后再继续爬，当然这很难。

前面说了js是裸奔的，顶多代码混淆下，但是也是在坏人手里的，想怎么改怎么改，只能说提高坏人作恶的成本。

防御方只能进行更大强度的代码混淆（其实也没啥用）。

#### 3.window.chrome

防：`!window.chrome||!window.chrome.runtime`如果是自动化的话，window.chrome会是undefined

攻：经测试puppeteer设置headless:true时，window.chrome是有值的，虽然会慢一点。或者反正也能篡改js。

```js
window.navigator.chrome = {
	runtime: {},
};
```

还有其他的检测plugins，language以及permissions，实际都可以被篡改。

看完是不是觉得非常悲观，怎么也拦不住！

还是那句话，能拦多少拦多少呗！

搞一些骚气一点的走位，其实你也发现不了我在哪里检测了

基本检测方式如下，当然还有很多别的指标来判断，我才不会告诉你。

```js
function detectAuto() {
  if (/HeadlessChrome/.test(navigator.userAgent) || // ua test
    !window.chrome || // headless test
    navigator.plugins.length === 0 || //plugins test
    // languages test
    !navigator.languages || navigator.languages.length === 0) {
    return true;
  }
  const permissionStatus = await navigator.permissions.query({ name: 'notifications' });
  if (Notification.permission === 'denied' && permissionStatus.state === 'prompt') {
    return true;
  }
  const r = [];
  const w = ['webdriver', '__driver_evaluate', '__webdriver_evaluate',
    ' __selenium_evaluate', '__fxdriver_evaluate', '__driver_unwrapped',
    '__webdriver_unwrapped', '__selenium_unwrapped', '__fxdriver_unwrapped',
    '_Selenium_IDE_Recorder', '_selenium', 'calledSelenium',
    '_WEBDRIVER_ELEM_CACHE', 'ChromeDriverw', 'driver-evaluate',
    'webdriver-evaluate', 'selenium-evaluate', 'webdriverCommand', 'webdriver-evaluate-response',
    '__webdriverFunc', '__webdriver_script_fn', '__$webdriverAsyncExecutor', '__lastWatirAlert',
    '__lastWatirConfirm', '__lastWatirPrompt', '$chrome_asyncScriptInfo', '$cdc_asdjflasutopfhvcZLmcfl_',
    '_phantom', '_phantomas',
  ];
  w.forEach((t) => {
    if (!!window[t] || !!window.document.documentElement.getAttribute(t) || !!navigator[t]) {
      r.push(t);
    }
  });
  return r.length > 0;
}
```

绕过js已整理好，但是还是不分享给你们这群坏人了。

参考：

https://intoli.com/blog/not-possible-to-block-chrome-headless/

https://intoli.com/blog/making-chrome-headless-undetectable/

https://www.zhihu.com/question/50738719