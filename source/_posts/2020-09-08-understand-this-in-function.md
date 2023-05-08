layout: post
title: "理解函数中的 this"
date: 2020-09-08 22:00:00
comments: true
categories: 
- Document
tags:
- JavaScript
- 基础知识
---

很多人都对 JavaScript 中 function 里的 this 产生疑惑。在我看来，这些疑惑在我们理解了核心的 function 的调用原理后，都会彻底理解。

<!-- more -->

### 核心原理

首先来看一下 function 的 call 方法。call 方法调用时：

- 可以传入一系列参数
- 第一个参数就是函数中的 this 值
- 方法调用时，this 会被置为第一个参数值，后面的参数才会被当做真正的参数使用

```
function hello(thing) {
  console.log(this + " says hello " + thing);
}

hello.call("Yehuda", "world") //=> Yehuda says hello world

```

所以，很容易理解，this 的值只有在 call 方法被调用时才能够确定，它不是一个不变的值。

### 简单方法调用

相比较 Function.call() 这样的调用方式，直接用方法名加括号的形式来调用。同时，这样子会将原有 call 方法的第一个参数的 this 直接设置成为 window：

```
function hello(thing) {
	console.log("Hello " + thing);
}

// equals to
hello.call(window, "world");

// 在 ES5 的严格模式下，window 会被替换为 undefined
hello.call(undefined, "world");

```

### 成员方法

如果函数存在于对象中，那么方法调用时的 this 会被指定为对象本身：

```
var person = {
  name: "Brendan Eich",
  hello: function(thing) {
    console.log(this + " says hello " + thing);
  }
}

// this:
person.hello("world")

// desugars to this:
person.hello.call(person, "world");

```

### bind

那么，如果我们需要一个方法有一个恒定不变的 this 值要怎么操作呢？

- 可以使用另一个方法 B 来封装方法 A，并在调用 A 方法的时候写死 this 的值
- 可以使用 bind 方法来进行封装：

```
	var B = someFun.bind(A);
	B(params);
```