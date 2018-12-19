---
title: JS中的面向对象5-Object.create和new Object
date: 2018-07-18 14:02:40
tags: javascript
---
Object.create和new Object的区别
<!-- more -->
```
var Person = function(age) {
this.age=age;
this.say=function(){console.log('hello');}
};
var p=new Person(11);
var p1 = Object.create(p);
```

![](/assets/proto1.png)

![](/assets/proto2.png)

从console中可以发现，p成了p1的原型对象，p中的属性和方法都成了p1的原型方法。

Object.create的作用就是传入一个对象，给创建的新对象提供`__proto__`引用

多用在对象的继承中。

Object.create的实现方式

```
if (!Object.create) {
Object.create = function(proto, propertiesObject) {
if (typeof proto !== 'object' && typeof proto !== 'function') {
throw new TypeError('Object prototype may only be an Object:' + proto);
} else if (proto === null) {
throw new Error("This browser's implementation of Object.create is a shim and doesn't support 'null'");
}
if (typeof propertiesObject != 'undefined') {
throw new Error("This browser's implementation of Object.create is a shim and doesn't support a second argument");
}

function F() {};
F.prototype = proto;
return new F();
}
}
```

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global\_Objects/Object/create](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)