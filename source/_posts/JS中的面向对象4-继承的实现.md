---
title: JS中的面向对象4-继承的实现
date: 2018-07-15 13:05:02
tags: javascript
---

理解了基本发展历史和深层原理，接下来就是生生不息的原因，继承部分啦。
<!-- more -->

我们知道JS是单继承的，Object.prototype是原型链的顶端，所有对象从它继承了包括toString等方法和属性。

Object本身是类当然也是构造函数，构造函数嘛当然是继承自Function.prototype

而Function也是对象，继承自Object.prototype。

这里就有一个鸡和蛋的问题：到底是先有Object还是先有Function。

```js
Object instanceof Function // true
Function instanceof Object // true

Function.prototype===Function.__proto__ // true
Object.prototype.__proto__ === null // true 说明原型链到Object.prototype终止。
```

上篇中我们了解到原型链是实现继承的主要方法

```js
function SuperType() {
this.property = true;
}
SuperType.prototype = {
constructor: SubType,
getSuperVal: function() {
return this.property;
}
}

function SubType() {
this.subProperty = false;
}
SubType.prototype = new SuperType(); // 重点
SubType.property.getSubVal = function() {
return this.subProperty;
}

var sub1 = new SubType();
sub1.getSuperVal(); // true
sub1.getSubVal(); // false
sub1 instanceof SubType; // true
sub1 instanceof SuperType; // true
```

回顾下new的作用

```js
var instance=new Person();
// 实际经历了如下四步：

// 1.创建空对象
var instance = new Object();

// 2.设置原型链，指向类【构造函数】的原型
instance.__proto__ = Person.prototype;

// 3.让构造函数Person的this指向实例instance,执行构造函数Person的函数体
var p = Person.call(instance);

// 4.判断Person的返回值类型
// 值类型就不要了，还是返回instance
// 如果是引用类型，替换掉instance返回引用类型
if(typeof p =='object'){
return p
}else{
return instance;
}
```

所以上文中重点new实现了如下：

```js
SubType.prototype.__proto__ = SuperType.prototype;
SuperType.call(SubType.prototype);
```


