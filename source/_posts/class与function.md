---
title: class与function
date: 2021-12-13 20:00:00
tags:
  - js
---

## 实现继承

本篇博文想到哪就写到哪~

js 里需要用到继承的场景，一般是需要自己通过构造函数创建对象的时候，这个对象有自己特别的一些方法，并且可以被标识为特定的类型。这时候我们通常会使用构造函数创建对象，并且可以通过继承实现代码扩展。

## function 实现

在 ES5 中，我们一般通过原型链继承，一般会通过构造函数实现

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}
Person.prototype.sayHello = function () {
  console.log("my name is ", this.name);
};
let person1 = new Person("guan", "2");
let person2 = new Person("jie", "3");
```

在实例化的过程中，person1 和 person2 均拥有 sayHello 方法。一般构造函数，命名时首字母大写。

## class 实现

在 ES6 中，实现了 class 定义方式，可以简单理解为 class 就是之前 function 实现继承方式的语法糖：

```js
class Person = {
  constructor(name,age){
    this.name = name;
    this.age = age;
  }
  sayHello(){
    console.log("my name is ", this.name)
  }
}
let person1 = new Person("guan", "2");
let person2 = new Person("jie", "3");
```

## new 操作

new 本质上做了 4 件事
新建一个对象
把这个对象的原型指向 new 的那个构造函数或者类
将新建的对象作为 this 的上下文
返回一个构造函数的返回值，如果没有返回值，则返回 this

## call

call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。

call 最大的作用用来指定函数的上下文执行环境

```js
function greet() {
  var reply = [this.animal, "typically sleep between", this.sleepDuration].join(
    " "
  );
  console.log(reply);
}

var obj = {
  animal: "cats",
  sleepDuration: "12 and 16 hours",
};

greet.call(obj); // cats typically sleep between 12 and 16 hours
```

当第一个参数不传时默认指向 window 对象，严格模式下指向 undefined

第一个参数为一个对象。

可以认为“js 中一切皆对象”，对象是拥有属性和方法的数据。 js 共有 7 中基本类型的（number，string，boolean，null，undefined，symbol，object ），之所以它们可以像对象一样引用，只是因为在引用过程中发生了“临时包装”导致，实际上，“js 中一切皆对象”是不准确的。

symbol 也是一个基础类型，主要作用于生成一个唯一的值

比如在枚举场景中：

```js
const COLOR_GREEN = Symbol("green");
const COLOR_RED = Symbol("red");
Symbol("green") === Symbol("green"); // false
```

就算 green 和 red 为同一字符都不会影响后面的判断逻辑
