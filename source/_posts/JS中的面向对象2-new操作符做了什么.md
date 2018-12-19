---
title: JS中的面向对象2-new操作符做了什么
date: 2018-06-22 18:00:51
tags: javascript
---
本系列是学习整理js中的面向对象的第2篇，new操作符做了什么
<!-- more -->

new运算符的作用是创建一个类的实例（类可以是我们自定义的对象类型Person，也可以是具有构造函数的内置对象（如Object，Array，Function））

```
function Person(age,name){
this.age = age;
this.name = name;
}
var instance =new Person();

var o1 = new Object();
```

实际经历4个步骤

```
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

#### 如果把随便一个函数当作构造函数，用new来调用

```
// 返回值类型
function say(name) {
console.log(name);
// return name; 加不加这个都返回空的say类型的obj:{},没有return 相当于return undefined，值类型，丢弃。
}
var instance = new say('Eva'); // 同上返回say类型的obj:{}

instance instanceof say; // true
instance.__proto__ === say.prototype; // true

function CreatePerson(age, name) {
var p = { age: age, name: name };
return p; // 返回引用类型，原有的实例被丢弃，因为原有的instance被丢弃
// 此处的实例和CreatPerson并无原型链的关系，只是普通的Object对象
// 所以可以理解上文中的寄生模式和稳妥模式都不再有原型链关系的原因啦。简言之，new无效则原型无效
}
var p = new CreatePerson(11, 'Eva'); // {age: 11, name: "Eva"} 但是注意此处不是CreatePerson类的对象了，就是普通的Object对象

p.__proto__ == CreatePerson.prototype // false;
```

#### new的实现

```
function New(fn) {

/*1*/
var instance = { '__proto__': fn.prototype }; // 每个对象天生自带__proto__属性，此处只是重新指定了

return function() {
/*2*/
var res = fn.apply(instance, arguments);
/*3*/
if (typeof res == 'object') {
return res;
} else {
return instance;
}
};
}

function Person(age, name) {
this.age = age;
this.name = name;
}

// 调用
New(Person)(11, 'Eva');
```

#### 无new调用

构造函数也是函数，可以直接调用，为了避免忘记通过new来调用构造函数，导致属性和方法添加在window对象上，污染全局变量，一般会在构造函数中检查当前作用域，这样即使不通过new来生成实例也可以得到正确的结果

```
function Person(age, name) {
if (this instanceof Person) {
this.age = age;
this.name = name;
} else {
return new Person(age, name);
}
}
```

理解了new的实际作用，就不难理解原型对象的实现了

参考文献：

[https://www.zhihu.com/question/36440948](https://www.zhihu.com/question/36440948)
