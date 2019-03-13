---
title: JS中的面向对象3-原型和构造函数的关系
date: 2018-06-30 18:02:09
tags: javascript
---
本系列是学习整理js中的面向对象的第3篇，原型和构造函数的关系，接下来就要更深入的探究其中的原理了。
<!-- more -->

#### 原型和构造函数到底有什么关系？

![](/images/proto3.png)

构造函数有原型对象，Person.prototype，此对象中有属性constructor，指向了构造函数Person，构造函数又有原型...

```js
Person.prototype.constructor===Person;//true
Person.prototype.constructor.prototype.constructor.prototype===Person.prototype; // true 两者相依相存
```

构造函数是为了方便创建对象。

构造函数通过prototype来存储要共享的属性和方法。

#### prototype和\_\_proto\_\_（\[\[Prototype\]\]）又有什么关系？

**类和构造函数**：比如Array你可以看做一个类，也可以看成是Array的构造函数。

`__proto__`:是**实例**的属性，指向构造函数的原型 constructor.prototype，每个对象都有。所以我们暂且称它为原型引用。

`prototype`:原型对象，只有函数才有prototype，所以**原型是构造函数（类）的原型**。

我们先来回忆下new操作符到底做了什么【参见js中的面向对象2】。

就是new操作符实现了实例的`__proto__`指向类的原型。

```js
// __proto__是new出来的实例内部包含的一个指针，指向constructor.prototype。每个对象都有
p.__proto__ === Person.prototype // true

p.prototype // undefined 只有函数才有prototype

Object.getPrototypeOf(p)=== p.__proto__===Person.prototype // Object.getPrototypeOf得到实例的__proto__
// 字面意思是获取实例的原型prototype，实际是获取原型的引用。
// 因为实例没有原型，只有原型引用__proto__，类（构造函数）才有原型prototype

Person.prototype.isPrototypeOf(p) // true isPrototypeOf 原型对象是构造函数的原型
```

#### prototype和实例又有什么关系呢？

prototype是用来保存类的公共属性和方法的，其实也是个特殊的对象，可以理解为和实例是一个level的。但是他们之间有什么关系和区别呢

```js
// 联系
p.constructor === Person.prototype.constructor === Person; // true 两者具有相同的构造函数

p.__proto__ === Person.prototype; // true 实例的原型引用指向了原型对象

// 区别
Person.prototype.__proto__===Person.prototype; // flase 原型的原型引用不再是原型对象啦
```

既然是一个level为啥不相等呢？因为原型肩负了继承的重要责任。

正是通过`Person.prototype.__proto__.__proto__.__proto__.__proto__.__proto__`来一层层向上查找父类，这就是所谓的**原型链**

所以继承的写法就是

```js
Person.prototype=new Parent();
```

结合前面new的作用，可以明白此处实际上的最重要的作用就是

```js
Person.prototype.__proto__=Parent.prototype;// 将原型引用指向父类的原型
// Person.__proto__总是指向父类
```

调用p.name时，会从p自身的属性中寻找是否有name，没有则到p对应的原型中寻找
`obj.__proto__.__proto__.__proto__`形成原型链，可以通过instanceof来验证obj是否是构造函数（类）的实例

继承的细节将在下节来详细解释

#### Object和Function的关系及特殊的原型对象

```js
var obj1= {x: 1};
var obj2= new Object();
obj1.__proto__ === Object.prototype // true 指向构造函数的原型
obj2.__proto__ === Object.prototype // true
obj1.toString === Object.prototype.toString === obj2.__proto__.toString // true 调用实例方法实际上是调用原型的方法

var fn1=function(){};
var fn2=function(){};

fn1.constructor===Function
fn1.__proto__===Function.prototype 
fn1__proto__===fn2.__proto__ // true 

fn1.prototype.constructor=fn1;
fn1.prototype.constructor.name // fn1
fn2.prototype.constructor.name // fn2
fn1.prototype===fn2.prototype // false 
fn1.prototype==={
    // fn1
    constructor:function(){

    }
} // 指向了Object.prototype


fn1.prototype.__proto__ === Object.prototype
// 所以Function 继承自Object
```

#### 原型的相关方法

```js
Object.getPrototypeOf(p) === Person.prototype; // true
Person.prototype.isPrototypeOf(p); // true
p.hasOwnProperty('name'); // false 是原型属性
p.salary = 25000;
p.hasOwnProperty('salary'); // true 是实例属性

// 通过in操作符和hasOwnProperty可以判断属性到底是在实例还是原型中
function hasPrototypeProperty(obj, name) { // 判断是否是原型属性
  return !obj.hasOwnProperty(name) && (name in obj);
}
// 因为in操作符包括原型属性，所以在遍历时eslint不建议使用for-in，如果必须使用也需要用hasOwnProperty来过滤下

p instanceof Person; // true 实例是否是类的实例
```

#### 基本类型判断

但是instanceof也有力不从心的时候，回忆下我们判断isArray，isDate等类型的方法，instanceof假定只有一个全局作用域，在页面中有多个frame时，存在两个以上的全局执行环境，一个框架向另一个框架传值时，会有不一样的构造函数。建议使用以下方式来判断。

```js
function isType(type) {
  return obj => Object.prototype.toString.call(obj) === `[object ${type}]`;
}

const isObject = isType('Object');
const isString = isType('String');
const isArray = Array.isArray || isType('Array');
const isFunction = isType('Function');

export { isObject, isString, isArray, isFunction };
// toString只能通过Object.prototype.toString.call来调用才能判断类型
```

#### 注意事项

```js
function Person(age, name, friends) {
  this.name = name;
  this.age = age;
  this.friends = friends;
  // 只在初始化时判断，不需要把每个方法都判断TODO
  if (typeof this.sayName != 'function') {
    Person.prototype.sayName = function() {
      console.log(this.name);
    }
    // 局部添加方法和属性会立即生效，重写整个原型对象会切断现有实例和原型的关系

    // 在已经创建实例的情况下，使用Person.prototype={}这种对象字面量，相当于重写原型
    // 会切断现有实例和原型之间的关联
  }
}
```

参考
[https://github.com/creeperyang/blog/issues/9](https://github.com/creeperyang/blog/issues/9)
[https://www.zhihu.com/question/34183746](https://www.zhihu.com/question/34183746)
