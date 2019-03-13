---
title: JS事件循环机制2-throttle和debounce
date: 2018-10-15 13:10:28
tags: javascript
---
函数的debounce和throttle
<!-- more -->

| | throttle | debounce |
| :--- | :--- | :--- |
| 释义 | 节流（整个事件所用的时间） | 去抖动（整个事件的单次循环之间的时间间隔） |
| | 水龙头打开本来是连续不断的出水，但是我们将水龙头不断拧紧，直到水龙头以一滴一滴的形式流出。使某个连续不断的动作在某个规定时间段只调用一次，叫做节流throttle | 从按下弹簧到弹簧恢复松弛状态需要一定的时间，每次按下弹簧必须要一定时间等其恢复到原状之后才可以进行下一次按压，这才是一次完整的循环。为两次循环动作设定最短的时间间隔，叫做防抖debounce |
| | 在某时间段只调用一次 | 两次触发时间间隔超过设定值才会调用 |
| 实例 | window.resize事件虽然一直触发，但是我们其实只需要某时间段内最后一次 | 图形验证码刷新，用户一直点击刷新，但是我们设定只有点击间隔大于500ms时才会触发 |
| | window.scroll事件滑动时一直触发，但是我们需要控制 | input中输入关键字查询，每次keydown输入关键字都触发ajax查询，我们可以设定两次请求的最小间隔 |
| | mouseMove事件 | 多次点击按钮放重复提交 |
| 对于时间的理解 | interval是对于整个事件所占用的时间 | interval是对于不断循环往复的事件之间的时间间隔 |
| 调用方式 | | |
| 差别 | 每3000ms只调用一次和两次调用时间间隔大于3000ms,差别在于前者两次调用时间可能很近 | 防抖用的多一点。还是要看具体的使用场景。其实很多场景差别并不那么大 |

#### 基本实现

#### debounce

eg:input中输入字符来搜索，如果用户连续keydown，造成大量ajax，正确的做法是设置一个门槛值，两次请求的最小间隔

debounce：返回一个新函数，两次触发的时间间隔大于事先设定的值才运行实际任务

```javascript
// ========================
// debounce
// 一般是一些用户操作
function debounce(fn, delay){
var timer = null;
return function(){
var context = this;
var args = arguments;
clearTimeout(timer);
timer = setTimeout(function(){
fn.apply(context, args);
}, delay);
};
}
// 得到fn的防抖版本
var lazyLayout = debounce(calLayout, 3000);
$(window).rezise(lazyLayout);
```

#### Throttle

```javascript
// ========================
// throttle
// 一般是对于一些延续性的动作
function throttle(fn,interval) {
var last;
return function() {
var curr = new Date().valueOf();
if (curr - last > interval) {
fn.apply(this, arguments);
last = curr;
}
}
}
var throttle = throttle(updatePosition, 3000);

// 一个延续性动作的停止时间
// TODO 注意延续性动作的停止时间和debounce也是差不多的

function end(func) {
var timer;
return function(event) {
if (timer) clearTimeout(timer);
timer = setTimeout(func, 100, event);
};
}
```

不要设置太多setTimeout，因为其特别耗费CPU，比较理想的做法是，将要推迟执行的代码都放在一个函数里，然后只对这个函数使用setTimeout或setInterval。

可以用于模块的生命周期
\_.defer:延迟调用fn直到当前调用栈清空，类似setTimeout\(fn,0\)。对于执行开销大的计算和无阻塞UI线程的HTML渲染时候非常有用

由于事件频繁被触发，因而频繁执行DOM操作、资源加载等重行为，导致UI停顿甚至浏览器崩溃

比如：

1. window对象的resize、scroll事件

2. 拖拽时的mousemove事件

3. 射击游戏中的mousedown、keydown事件

4. 文字输入、自动完成的keyup事件

实际上对于window的resize事件，实际需求大多为停止改变大小n毫秒后执行后续处理；而其他事件大多的需求是以一定的频率执行后续处理。针对这两种需求就出现了debounce和throttle两种解决办法。

这两者本质都是函数调用的频度控制器。

参考文献：

[http://www.cnblogs.com/fsjohnhuang/p/4147810.html](http://www.cnblogs.com/fsjohnhuang/p/4147810.html)

[https://zhuanlan.zhihu.com/p/26054718](https://zhuanlan.zhihu.com/p/26054718)

用于测试js的性能

[https://jsperf.com/](https://jsperf.com/)
[http://www.alloyteam.com/2012/11/javascript-throttle/](http://www.alloyteam.com/2012/11/javascript-throttle/)