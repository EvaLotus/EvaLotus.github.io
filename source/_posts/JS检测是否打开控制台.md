---
title: JS检测是否打开了控制台（devtool）
date: 2018-12-28 20:11:15
tags: javascript
---
JS并没有直接提供判断检测用户是否打开devtool的API。
思考下，打开控制台有哪些特征呢？
<!-- more -->

1.按键类，用户按下了F12，cmd+i+u,右键选择审查元素等

但是打开devtool并不能用一种操作模式就能完全涵盖，不同操作系统下的键盘特征也不一样。

2.窗口宽度变化类，内部宽度变窄

Github上搜到这个 devtools-detect 只通过宽度检测加上直观的demo就可以拿到800+的stars哦。

核心代码

```js
let threshold = 160;
setInterval(function () {
    var widthThreshold = window.outerWidth - window.innerWidth > threshold;
    var heightThreshold = window.outerHeight - window.innerHeight > threshold;
    var orientation = widthThreshold ? 'vertical' : 'horizontal';

    if (!(heightThreshold && widthThreshold) &&
      ((window.Firebug && window.Firebug.chrome && window.Firebug.chrome.isInitialized) || widthThreshold || 
      heightThreshold)) {
        if (!devtools.open || devtools.orientation !== orientation) {
            emitEvent(true, orientation);
        }

        devtools.open = true;
        devtools.orientation = orientation;
    } else {
        if (devtools.open) {
            emitEvent(false, null);
        }

        devtools.open = false;
        devtools.orientation = null;
    }
}, 500);
```
主要思路是：一直500ms的间隔来监控是否出现内外宽高度差>160的情况，基本的场景都可以覆盖，除了将调试窗口从页面中拖出来的情况。虽然思路很简单，但是应该是测试工作在的。

不得不说确实很有效，但我们可以升级下，将setInterval改为window.onresize，性能上应该会好很多

3.特殊特征类
```js
(function() {
  var re = /x/;
  var i = 0;
  console.log(re);

  re.toString = function() {
    // console.log(i);
    return '第 ' + (++i) + ' 次打开控制台';
  };
})();


if (window.console && window.console.log) {
  console.log("打开了控制台")
}
```
参考 https://stackoverflow.com/questions/7798748/find-out-whether-chrome-console-is-open

https://github.com/sindresorhus/devtools-detect