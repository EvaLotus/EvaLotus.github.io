---
title: JS事件循环机制1
date: 2018-10-13 14:09:20
tags: javascript
---
单线程和异步

<!-- more -->
#### js为啥是单线程的？

作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

单线程
优点：免去了频繁切换线程的开销，减少资源互抢问题。不用如多线程那样处处在意状态的同步，没有死锁的概念。
缺点：容易阻塞，无法利用多核CPU。

#### 为啥有异步？

异步编程的四种方式：
回调，事件监听，观察者模式（发布订阅），Promise（为异步编程提供统一的接口）

是**浏览器**提供了多线程的环境
网络请求，定时器，事件监听等任务是非常耗时的，都老老实实的排队等待执行的话，执行效率会非常的低，甚至导致页面的假死。所以，浏览器为这些耗时任务开辟了另外的线程，主要包括**http请求线程**，**浏览器定时器线程**，**浏览器事件触发线程**，**页面渲染线程**
浏览器主线程用来页面渲染？

浏览器中js和UI共用一个线程

主线程排队执行同步任务

主线程之外有个**任务队列**（callback queue），任务队列中都是异步任务，主线程空了就会读取任务队列里的任务，将其加入主线程执行，如`setTimeout(fn,0)`也是在主线程所有的代码都执行完毕之后才执行，意思是尽可能早的执行fn，而不是马上执行fn。

任务产生事件：
http请求线程：网络请求事件，ajax success failure的回调
浏览器定时器线程：setTimeout的回调
浏览器事件触发线程：用户交互事件，如click，keypress等事件的回调

这些callback都被加入callback queue中，**浏览器是多线程的，但是js是异步的**，将这些任务放在任务队列里等待执行而已

#### 无阻塞Never Blocking

Event
JS通过events和callbacks来处理I/O,所以在等待ajax返回时依然可以处理用户输入

Event loop事件循环：主线程不断从任务队列中读取任务的过程，是实现异步的一种方式。js宿主环境的机制

#### setTimeout运行机制

setTimeout和setInterval的运行机制：将指定的代码添加到任务队列中，等到下一轮EventLoop（主线程从任务队列中读取任务）时，再检查是否到了指定时间，如果到了就执行对应的代码，否则就等下一次Event Loop重新判断，setTimeout指定的代码，必须等到本次EventLoop执行完所有代码执行才执行

setTimeout\(\)只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。要是当前代码耗时很长，有可能要等很久，所以并没有办法保证，回调函数一定会在setTimeout\(\)指定的时间执行。

```
setTimeout(waitTask,1000)// 10000+100?还是1000
veryLongTask();// 耗时10000
```

setTimeout\(fn,0\)是立即执行吗

```
setTimeout(fn1,0);// fn2先执行
fn2();
```

必须等主线程中的同步任务和任务队列中已有事件全部处理完之后才执行setTimeout

setTimeout\(fn,0\)只能做到尽可能早的执行指定任务，H5新标准规定，setTimeout推迟执行的事件最少是4毫秒，小于4会自动增加到4，为了防止多个setTimeout\(fn,0\)连续执行造成性能问题

setTimeout\(fn,0\)加在你要延迟执行的函数上，或者需要页面渲染完毕才执行

```
document.getElementById('my-ok').onkeypress = function() {
var self = this;
setTimeout(function() {
self.value = self.value.toUpperCase();
}, 0);
}
```

js是单线程的，容易阻塞
将复杂的操作分片放在setTimeout\(fn,0\)中执行

setTimeout\(fn,1000\)返回一个int，可以clearTimeout来取消对应定时器

setTimeout和setInterval返回的整数值是连续的\(一定环境下，比如浏览器控制台，或者js执行环境等\)，也就是说，第二个setTimeout方法返回的整数值，将比第一个的整数值大1。利用这一点，可以写一个函数，取消当前所有的setTimeout。

```
(function() {
var gid = setInterval(clearAllTimeouts, 0);

function clearAllTimeouts() {
var id = setTimeout(function() {}, 0);
while (id > 0) {
if (id !== gid) {
clearTimeout(id);
}
id--;
}
}
})();
```

#### setTimeout和Promise

```
setTimeout(function() {
console.log(1)
}, 0);
new Promise(function executor(resolve) {
console.log(2);
for( var i=0 ; i<10000 ; i++ ) {
i == 9999 && resolve();
}
console.log(3);
}).then(function() {
console.log(4);
});
console.log(5);
// 2 3 5 4 1
```

promise虽然是异步操作但是setTimeout是等所有的同步操作加任务队列里所有的任务都执行完之后才执行的

#### nodejs

i/o密集和cpu密集型
适合io密集型而不适合cpu密集型操作

不必耗费过多的系统开销，把精力放在处理多线程
宿主环境和事件驱动机制使它实现了非阻塞IO

进程-&gt;线程-&gt;协程

很多让nodejs支持多线程的方法是使用C++的addon实现，在需要进行cpu密集型计算的地方，把js代码改写成c/c++代码，但是如果开发人员对c++不是很熟悉，一来开发效率会降低不少，二来也容易出bug，而且我们知道在addon中的c++代码除了编译出错外，是很难调试的，毕竟没有vs调试c++代码方便。

v8引擎是c++写的，解析js的

js处理并发就是排队

reactor模式：NIO，selector多路复用，

nodejs为啥要用js来写呢：没有历史包袱

线程process 进程thread

# 从setTimeout来理解事件循环

基于event loop（事件循环）的concurrency model（并发模型）

runtime（运行时）的概念：
由stack（栈），heap（堆），queue（队列）来组成

stack:函数执行
heap：Object 的allocated内存分配
queue：runtime有message queue，每个message对应处理这条message的function
event loop：在某些时间点，runtime开始处理queue上的message，message从queue中移除，其对应的function被调用，message会被作为参数传递进去，调用函数也会产生新的stack frame

一般主线程都空了之后就同步的等待messgage，来一条处理一条

```
while (queue.waitForMessage()) {
queue.processNextMessage();
}
```

不可以像C那样切换线程，如果处理一条messgae比较耗时，就无法处理用户交互了，可以将messgage分片处理
message queue空了之后event loop再次执行？？？

message和function对应，事件发生时就会产生message，如果click事件没有handler，事件就丢弃了，不产生message

理解：
普通的代码都是在stack中执行的，setTimeout是加在queue中的

感觉event loop并不是主线程读取queue，主线程即stack中的代码在js加载时就一行行执行完毕了，其他的如网络请求，事件点击等，都是用户交互之后通过事件和message加入在queue中，通过和message关联的function来进行处理的

因为你不可能在stack中新增function吗？

Queue：
Callback queue：任务队列
Single thread：单线程
Event Loop：事件循环
macroTask：

microTask：

callstack：
I/O
JS V8 engine

Stack，heap，message queue，runtimes（web worker 或 cross-origin iframe）

runtimes：不同的运行时环境有不同的Stack，heap，message queue。可以通过postMessage来传递消息

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)

参考文献：
[http://www.ruanyifeng.com/blog/2014/10/event-loop.html](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)

[关于setTimeout的面试](http://caibaojian.com/interesting-interview.html)

[关于setTimeout你不知道的事](http://caibaojian.com/about-settimeout.html)