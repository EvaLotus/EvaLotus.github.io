---
title: 「 JS函数之美 」函数柯里化和this那些事
date: 2018-02-27 13:13:16
tags: javascript
---
### 函数柯里化

函数柯里化：创建一个已经设置好了一个或多个参数的函数。

<!-- more -->

```javascript
// 柯里化
function curry(fn) {
  // arguments=[fn,...args];
  var args = [].slice.call(arguments, 1);
  return function() {
    // 柯里化之后的函数的arguments转array
    var innerArgs = [].slice.call(arguments);
    var finalArgs = args.concat(innerArgs);
    return fn.apply(null, finalArgs);
  }
}

function add(a, b) {
  console.log(a + b);
}

curry(add, 2)(6); // 设置一个固定参数
curry(add, 1, 4)(); // 设置两个固定参数

// 函数绑定
// ES5有原生的bind，但是支持ie9+ polyfill
if (!Function.prototype.bind) {
  Function.prototype.bind = function(context) {
    // arguments是bind的
    var args = [].slice.call(arguments, 1);
    var fn = this;
    return function() {
      var innerArgs = [].slice.call(arguments);
      var finalArgs = args.concat(innerArgs);
      // 除了此处和curry非常相似
      fn.apply(context, finalArgs);
    };
  }
}

// 只获取到了绑定后的函数传入的参数
// 结合了柯里化的函数绑定
function bind(fn, context) {
  // arguments=[fn,context,...args];
  var args = [].slice.call(arguments, 2);
  return function() {
    var innerArgs = [].slice.call(arguments);
    var finalArgs = args.concat(innerArgs);
    return fn.apply(context, finalArgs);
  }
}
bind(a.addTwoNum, { num: 2 }, '1');


// 函数绑定主要就是为了改变this的指向
// 函数绑定和函数柯里化提供了强大的【动态函数创建】的能力，但是使用时会有额外的性能开销

// TODO
function partial(fn, ...args) {
  return function(...partialArgs) {
    var finalArgs = args.concat(partialArgs);
    return fn.apply(this, finalArgs);
  }
}
```

#### bind，apply和call的区别

```js
var a = {
  num: 1,
  addNum: function(b) {
    console.log(b + this.num);
    return b + this.num
  },
  addTwoNum: function(b, c) {
    return b + this.num + c
  }
}
a.addNum(2); // 1+2

a.addNum.bind({ num: 3 })(5); // 3+5 改变了this的指向
// bind得到的是一个this指向变化的【函数】，还是需要调用

// apply和call的区别在于调用方式不同，前者传入参数数组，后者传入参数列表
// bind，apply和call都是用来改变this的指向
a.addNum.apply({ num: 4 }, [6]); // 4+6 改变了this的指向 【调用方法】fn.apply(context,args);
a.addNum.call({ num: 5 }, 6); // 4+6 改变了this的指向 【调用方法】fn.apply(context,...args);
```

jQuery中$.proxy也是改变this的指向

```js
var obj = {
  name: "John",
  test: function() {
    alert(this.name);
    $("#test").unbind("click", obj.test);
  }
};

$("#test").click( jQuery.proxy( obj, "test" ) );
```
call,apply,bind的区别
<!-- more -->

#### 为什么要使用call和apply

```js
// 可以改变this的指向
[].slice.call(document.getElementsByClassName('btn'));
[].slice.call(arguments);
```

比如需要将常见的arrayLike：arguments，nodeList这种类array转为array。可以直接调用原型方法，将this指向这些arrayLike即可调用

#### call和apply的区别

```js
Math.max.call(null,1,2,3,4);
Math.max.apply(null,[1,2,3,4]);
```

区别：

call是直接参数，apply是参数列表

apply的参数list需要进一步解析，call的性能会更好

函数bind

也是改变this的指向

#### call，apply是直接调用对应函数，bind会生成新函数

```js
var m = { x: 1 };

function fn(y) {
  alert(this.x + y);
}
fn.apply(m, [5]);
fn.call(m, 5);
var fn1 = fn.bind(m, 5);
// fn.bind(conext,arguments);
fn1();

// bind实现
function bind(fn, context) {
  return function() {
    return fn.apply(context, arguments);
  }
}

// 常用方式
document.getElementById('btn').onclick = bind(handler.handlerFn, handler);
```


