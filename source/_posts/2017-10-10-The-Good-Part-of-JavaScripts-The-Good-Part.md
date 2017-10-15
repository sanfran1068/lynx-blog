layout: post
title: "JS语言精粹中的精粹"
date: 2017-10-10 12:00:00
banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg
comments: true
categories: 
- Document
tags:
- JavaScript
- 基础知识
- 读书笔记
---

《JavaScript语言精粹》这本书可谓是JavaScript这门语言的几乎所有知识点的总结，相当于考试之前要画的重点。尽管书中对于各个知识点的介绍比较少，但是知识点覆盖很全，可以进行反复阅读和积累。下面就对这本书中的知识点进行一个整理和总结。

<!-- more -->

#### 语法

这一部分主要总结JavaScript中的语言结构和相关语法知识点。

- 空白：空格一般是可以省略，但是对于`var that = this;`这种语句var和that之间的空格是必须保留的，而且为了代码美观，也要适当使用空格。
- 注释：使用`/* */`这种风格的注释在遇到正则表达式中出现*/时很容易出错，所以**建议使用`//`的注释方式**。
- 标识符：只能使用字母、$、_开头，其中也只能使用字母、数字或下划线。**避免使用保留字作为标识符**。
- 数字：JS中只有一个数字类型，内部表示为64位浮点数，一定程度可以避免溢出问题；**NaN是一个数值，不等于任何值（包括自己）**，有isNaN(number)方法；Infinity表示无限大；数字拥有方法，也可以使用Math对象的方法对数字进行操作。
- 字符串：由单引号/双引号包括，\开头是转义字符。在ES5中，JS中所有字符都是**16位**Unicode；字符串有.length属性，不可变，但是可以通过一些方法对其进行操作。
- 语句：switch、while、for、do可有前置标签label，配合break语句使用

    - if语句中可以作为假值的条件有：false、null、undefined、''、0、NaN，其他都作为真值条件；
    - for语句中可以使用for in语句对对象或者类对象的属性/键名进行枚举，但是要确保该属性是否来自该对象还是原型链，使用`if(object.hasOwnProperty(variable))`进行确定；
    
- 表达式：注意运算符的优先级；typeof运算符可产生六个值（字符串）number、string、boolean、undefined、function和object；
- 字面量：
- 函数：

#### 对象

JavaScript中的简单数据类型有Number、String、Bool、Null、undefined，这些类型都不可变（但是数字、字符串和布尔值有一些方法），其他的数据类型都属于对象Object。

- 对象字面量：

    ```javascript
    var stooge = {
      "first-name": "Jerome", //不合法的属性名要用""包括
      last_name: "Howard"
      }
    ```
    
- 检索：对象的属性检索方式有两种，一种是