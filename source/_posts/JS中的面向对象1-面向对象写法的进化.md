---
title: JS中的面向对象1-面向对象写法的进化
date: 2018-06-18 18:00:12
tags: javascript
---
本系列是学习整理js中的面向对象的第一篇，面向对象写法的进化
<!-- more -->

```javascript
// ===========================
// 1.最原始的创建对象
var person = new Object();
person.name = 'Eva';
person.age = 12;
person.sayName = function() {
console.log(this.name);
}
// ===========================
// 2.使用对象字面量来创建对象
var person = {
name: 'Eva',
age: 12,
sayName: function() {
console.log(this.name);
}
}
// ===========================
// 3.工厂方法来创建对象
function createPerson(name, age) {
var o = new Object();
o.name = name;
o.age = age;
o.sayName = function() {
console.log(this.name);
}
return o;
}
// 缺点是：无法知道对象的类型

// ===========================
// 4.构造函数模式
function Person(age, name) {
this.age = age; // 为实例添加属性和方法
this.name = name;
this.sayName = function() {
console.log(this.name);
};
// 本质是如下:每个方法在每个实例上都需要重新创建一遍
this.sayName=new Function('console.log(this.name)');
// this指向：如果是无new调用，this就会指向window || global
// new 调用：new做了什么
}
var p1 = new Person(11, 'Eve');
var p2 = new Person(12, 'Tom');
p1.constructor === p2.constructor === Person;
p1 instanceof Person; // true
p2 instanceof Person; // true
p1.sayName == p2.sayName // false 每个方法在每个实例上都需要重新创建一遍
// 相比前者：可以将实例标志成某种特定的类型
// 缺点是：每个方法在每个实例上都需要重新创建一遍,直接写成全局函数又会污染全局变量，无封装性可言

// ===========================
// 5.原型模式
function Person(age,name) {

};
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
p1.sayName == p2.sayName // true

// 简写
// 缺点：constructor会指向Object构造函数
Person.prototype = {
constructor: Person,// 手动指定
name: 'Eva',
age: 18,
sayName: function() {
console.log(this.name);
}
}

// ===========================
// 6.构造函数和原型模式组合使用，解决引用类型属性的共享问题：最常用的写法
Person.prototype.toys = ['toy1','toy2'];
p1.toys.push('toy3');
p2.toys.length == 3// true
// 改为
function Person(age, name, toys) {
this.age = age; // 为实例添加属性和方法
this.name = name;
this.toys = toys;
}
Person.prototype={
constructor:Person,
sayName:function() {
console.log(this.name);
}
}

// ===========================
// 7.动态原型模式
function Person(age, name) {
this.age = age;
this.name = name;
if(typeof this.sayName!='function'){
Person.prototype.sayName = function() {
console.log(this.name);
};
}
}
// 不能再使用对象字面量重写Person.prototype,在已经创建了实例之后再重写原型，就会切断实例与新原型的关系

// ===========================

// 8.寄生构造函数模式
function Person(age, name) {
var o = new Object();
o.name = name;
o.age = age;
o.sayName = function() {
console.log(this.name);
}
return o;
}
// 如何把Object换成Array就更好理解了，创建的新对象是基于Array的，所以是寄生模式。
// 因为创造出的对象并不是Person类，而是还是Object类，所以没办法使用instanceof来判断类型，不建议使用这种方式

// ===========================
// 9.稳妥构造函数模式
function Person(age, name) {
var o = new Object();
o.name = name;
o.age = age;
o.sayName = function() {
console.log(name);
}
return o;
}
var p=Person(11,'Eva');
// 与上面的区别在于，1.不引用this，2.无new构造
// 同上：创建对象与构造函数没什么关系，也不能使用instanceof
// 重点在于安全性：除了通过sayName无法访问name属性
```


