---
layout: post
title: "JavaScript 中的值"
date: 2021-04-21 22:00:00
comments: true
categories: 
- Document
tags:
- JavaScript
- 基础知识
---

### JavaScript 中的值的分类

JavaScript 中的值可以分为**原始类型**和**复杂类型（对象）**：

- 原始类型
    - Boolean - `true` 或 `false`
    - null - 用 `type of` 检验 `null` 数据类型时为 `Object` ，但它不是对象，这是JS的一个bug
    - undefined
    - number - JavaScript中的所有数字都是浮点数，没有整数
    - string
    - symbol (ES6)
- 复杂类型（对象）
    - 原始类型的包装对象（Boolean、Number、String）
    - 内置对象（Date）
    - 用户创建对象

<!-- more -->

原始类型和复杂类型之间有一些不同点：

- 复杂类型可变，但是原始类型不可变（这里指的是**值的可变性**）：
    
    ```jsx
    // 复杂类型
    var obj = { arr: [1, 2, 3] };
    obj.arr.push(4);
    
    // 原始类型，可以改变变量的值，但是不能改变这个值的值
    var str = 'some string';
    ```
    
- 原始类型作为值保存（存放在栈中），复杂类型作为引用保存（存放在堆中，其引用存放在栈中）

原始类型中的 null 和 undefined 是两个比较特殊的类型。两种类型的本质区别是：null 是**程序级正常或预期的值缺失**（例如我们给一个变量赋为空值时就使用null），而 undefined 是系统级的意外和错误的值缺失（没逾期到的情况，比如一个未声明的变量或者命名空间不存在的对象属性）。还有其他一些区别：

- typeof null === 'object'，这个现象是原始 JavaScript 语言中的错误（JavaScript 的底层中二进制前三位如果都是 0 的话就被判断为 object 类型，但是 null 二进制表示是全0）
- null == undefined 返回 true，但是 null === undefined 返回 false
- null 可以进行递增递减（null 相当于一个介于0和1之间的值）

### 怎样判断值的类型

#### typeof 运算符

- 使用方法
    
    ```jsx
    console.log(typeof 'someString')  // 'string'
    typeof('someString')              // return 'string'
    ```
    
- 返回值都是字符串类型，有以下几种返回值
    - 'undefined'，所有未定义的变量
    - 'boolean'，布尔类型
    - 'string'，字符串类型
    - 'number'，数字类型
    - 'function'，函数类型
    - 'object'，对象类型（包括一些内置对象、数组、正则表达式等，还有 **null**）
- 原理：不同的对象在底层都表示为二进制，在 Javascript 中二进制前（低）三位存储其类型信息，typeof 根据这三位进行判断
- **注意点：不能单独判断数组、正则表达式和一些内置对象的类型，null 的类型也会返回 'object'**

#### instanceof 运算符

- 使用方法： `val instanceof Object`
    
    ```jsx
    console.log([] instanceof Array)      // true
    console.log(new Date instanceof Date) // true
    ```
    
- 返回值是布尔值，表示一个值的类型是不是某个对象
- 原理： `a instanceof A` 检测 a 的原型链上是否有 A.prototype
    
    ```jsx
    var a = function(){}
    console.log(a.__proto__ === Function.prototype) // true
    ```
    
- 注意点只能已知值的类型（原型对象）然后判断是否相等，无法直接返回值的类型

#### Object.prototype.toString.call()

- 使用方法
    
    ```jsx
    // 比较适用于复杂类型的判断
    var arr=[1, 2];
    Object.prototype.toString.call(arr); // "[object Array]"
    Object.prototype.toString.call(arr).indexOf('Array') === 8; // true 
    ```
    
- 返回值是字符串类型，格式为 `'[object Array]'` ，后面的是其原型
- 原理：JavaScript 中的对象都继承自 Object，而且原始类型也被包装过成为了对象，所以可以使用 Object.prototype.toString.call() 来判断。
- 注意点：继承过后的对象的 toString 方法已经将最顶端的方法重写，所以普通的 toString 方法是不能返回对象类型的