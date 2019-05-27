---
title: JS中的面向对象1-面向对象写法的进化
date: 2018-06-18 18:00:12
tags: javascript
---
JS红宝书中关于JS面向对象写法讲的很详细也很理解，把整个发展过程简化整理了一下，方便复习。
本系列是js中的面向对象的第一篇，面向对象写法的进化。
<!-- more -->

1.最原始的创建对象方法
```js
var person = new Object();
person.name = 'Eva';
person.age = 12;
person.sayName = function() {
  console.log(this.name);
}
```


2.使用对象字面量来创建对象
```js
var person = {
  name: 'Eva',
  age: 12,
  sayName: function() {
    console.log(this.name);
  }
}
```

3.工厂方法来创建对象
```js
function createPerson(name, age) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.sayName = function() {
    console.log(this.name);
  }
  return o;
}
```
缺点是：无法知道对象的类型


4.构造函数模式
```js
function Person(age, name) {
  this.age = age; // 为实例添加属性和方法
  this.name = name;
  this.sayName = function() {
    console.log(this.name);
  }
};
// 本质是如下:每个方法在每个实例上都需要重新创建一遍
this.sayName = new Function('console.log(this.name)');
// this指向：如果是无new调用，this就会指向window || global
// new 调用：new做了什么参见下篇
}
var p1 = new Person(11, 'Eve');
var p2 = new Person(12, 'Tom');
p1.constructor === p2.constructor === Person;
p1 instanceof Person; // true
p2 instanceof Person; // true
p1.sayName == p2.sayName // false 每个方法在每个实例上都需要重新创建一遍
```
相比第三种：可以将实例标志成某种特定的类型
缺点是：每个方法在每个实例上都需要重新创建一遍,直接写成全局函数又会污染全局变量，无封装性可言

5.原型模式
```js
function Person(age, name) {};
Person.prototype.name = 'Eva';
Person.prototype.age = 8;
Person.prototype.sayName = function() {
  console.log(this.name);
};
var p1 = new Person(11, 'Eve');
var p2 = new Person(12, 'Tom');
p1.constructor === p2.constructor === Person;
p1 instanceof Person; // true
p2 instanceof Person; // true
p1.sayName === p2.sayName // true

// 可以简写为以下
// 缺点：constructor会指向Object构造函数
Person.prototype = {
  constructor: Person, // 在这里手动指定来解决
  name: 'Eva',
  age: 18,
  sayName: function() {
    console.log(this.name);
  }
}
```

6.构造函数和原型模式组合使用
```
Person.prototype.toys = ['toy1', 'toy2'];
p1.toys.push('toy3');
p2.toys.length === 3 // true

// 改为以下：解决引用类型属性的共享问题,*是最常用的写法*
function Person(age, name, toys) {
  this.age = age; // 为实例添加属性和方法
  this.name = name;
  this.toys = toys;
}
Person.prototype = {
  constructor: Person,
  sayName: function() {
    console.log(this.name);
  }
}
```

7.动态原型模式
其他语言开发者一般都会将原型和构造函数写在一起，所以出现了下面的模式，将所有信息都封装在构造函数中,更完美
```js
function Person(age, name) {
  this.age = age;
  this.name = name;
  if (typeof this.sayName !== 'function') {
    Person.prototype.sayName = function() {
      console.log(this.name);
    };
  }
}
```
需要注意的是不能再使用对象字面量重写Person.prototype，因为在已经创建了实例之后再重写原型，就会切断实例与新原型的关系

8.寄生构造函数模式
```js
function SpecialArray() {
  var vals = new Array();
  vals.push.apply(vals,arguments);
  vals.toPipedString=function(){
    return this.join('|');
  }
  return vals;
}
```
创建的新对象是基于Array的，所以叫寄生模式。
因为创造出的对象并还是Array类而不是SpecialArray类，所以没办法使用instanceof来判断类型，一般不建议使用这种方式



9.稳妥构造函数模式
```js
function Person(age, name) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.sayName = function() {
    console.log(name);
  }
  return o;
}
var p = Person(11, 'Eva');

```
与上面前7种的区别在于：1.不引用this，2.不通过new来创建
和第8种有相同的问题：创建对象与构造函数没什么关系，也不能使用instanceof
但是保证了局部变量的安全性，除了sayName函数，外部无法访问name属性

通过面向对象写法的进化过程可以发现，第7种方式是最常用最完美的方式，后面几种也在某些特定场景有特定的应用。