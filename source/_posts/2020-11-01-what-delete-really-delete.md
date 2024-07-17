---
layout: post
title: "What delete really delete?"
date: 2020-11-01 21:03:00
comments: true
categories: 
- Document
tags:
- JavaScript
- 基础知识
---

为了引入本文要讨论的内容，先放出一个几乎不会遇到的问题：`delete 0` 会发生什么？
答案是控制台输出 `true`，其他什么都没发生。
那假如我们继续尝试delete ，执行下面的语句会分别发生什么呢？

```javascript 
/* example 1 */
delete 0
/* example 2 */
const a = {name: 'z'}
delete a.name
/* example 3 */
delete y
/* example 4 */
delete null
/* example 5 */
delete undefined
/* example 6 */
const x = 1
delete x
/* example 7 */
const map = new WeakMap();
delete map;
```

<!-- more -->

### 输出解析

1. 返回 true，实际上没做任何事情，只是根据 delete 的规则，没有发生异常就必须返回 true
2. 返回 true，删除了 [obj.name](http://obj.name) 表达式的值
3. 如果 y 根本不存在，delete y 什么也不做，返回 true；访问不存在的变量 y 报 ReferenceError 错误，其实是对 y 表达式的的 Result 引用做 getValue 的时候报的错误，然后为啥 typeof y 和 delete y 不报错，因为这两个操作没有求值。如果 y 的 configurable 为 false，delete object.y 不能删除掉y属性，返回 false；如果在严格模式下，会报错：TypeError: Cannot delete property 'c'
4. 返回 true，null是值，相当于delete 0
5. 返回 false
6. 返回 false
7. 返回 false
8. 同 7 的原理一样，都是在删除一个不可设置也就是只读的 global 上的属性，因此返回 false。

对于所有情况都是 true，除非属性是一个[自身的](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) [不可配置](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Errors/Cant_delete)的属性（configurable 为 false），在这种情况下，非严格模式返回 false。

```javascript
Object.getOwnPropertyDescriptor(obj, 'x');
```

### 分析

1、delete x 的三种可能：x是一个值、x是一个引用、x是global对象上声明的属性，等同于一个全局变量。

- 删除一个值：delete 0，不代表真的把0从语言当前的运行环境中删除，彷佛“啥也没发生”？
- 删除一个属性：delete obj.x ？
- 删除一个变量：delete x ？

后两者其实都可以可以理解成，删除对象的属性，x可以理解成global.x。

2、delete 这个操作的正式语法设计并不是“删除某个东西”，而是“删除一个表达式的结果”；

```jsx
delete expression
```

所以1当中的delete 0，JavaScript 将 0 视为一个表达式，并尝试删除它的求值结果。所以，现在这里的 0，其实不是值（Value）类型的数据，而是一个表达式运算的结果（Result）。而在进一步的删除操作之前，JavaScript 需要检测这个 Result 的类型：

- 如果它是值，则按照传统的 JavaScript 的约定返回 true；
- 如果它是一个引用，那么对该引用进行分析，以决定如何操作。

当 x 是全局对象 global 的属性时，所谓delete x其实只需要返回global.x这个引用就可以了。而当它不是全局对象 global 的属性时，那么就需要从当前环境中找到一个名为x的引用，删除其表达式的结果即可。

3、那么究竟如何去看是值还是引用？

语句的执行结果是一个值（value） 表达式的执行结果是一个result（可能是一个引用，也有可能是一个值），是引用还是值，取决于这是要一个左值还是右值。当你确定了一个Result用作lhs（左手端），那么它就是引用；如果确定它用作rhs（右手端），那么它就是值（将由引擎隐式地调用`GetValue()`）。

```jsx
 x = x; // x = GetValue(x)
// “把 值x 赋给 引用x”
```

4、“delete x”归根到底，是在删除一个表达式的、引用类型的结果（Result），而不是在删除 x 表达式，或者这个删除表达式的值（Value）。

**在 JavaScript 中的delete是一个很罕见的、能直接操作“引用”的语法元素。除了=，在Js中几乎没有其他操作符能直接去操作引用。**

### 总结

- delete 运算符尝试删除值数据时，会返回 true，用于表示没有错误（Error）。
- delete 0 的本质是删除一个表达式的值（Result）。
- delete x 与上述的区别只在于 Result 是一个引用（Reference）。
- delete 其实只能删除一种引用，即对象的成员（Property）。

![https://intranetproxy.alipay.com/skylark/lark/0/2022/png/54556479/1653465940596-f2186ba3-a104-4006-82d0-51875255d664.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/54556479/1653465940596-f2186ba3-a104-4006-82d0-51875255d664.png)

### delete undefined

在早期的JavaScript中，undefined是一个特殊值，是在运行期中通过void运算，或者不返回值的函数，又或者一个声明了但未赋值的变量，等等类似这样的情况来“计算得到”的。所以在JavaScript的早期版本中，你没有办法直接判断“undefined是undefined”，例如无法写出“x === undefined”这样的代码，而你只能写类似“typeof(x) ==='undefined'”这样的代码。

后来（其实也没有太久），规范就约定把undefined作为可以缺省访问的名字，类似于null。但是这个时候就带来了一个矛盾，因为这个undefined很重要，早期的绝大多数框架或引擎都把它作为一个“全局名字”给声明了。也就是说，ECMAScript现在既没有办法将它规范成一个keyword，也没有办法处理成保留字等等，它看起来像null，但又没有办法在规范层面强制它。所以……ECMAScript就搞了一个“奇招”：把undefined声明成全局的属性，怎么样？！！（又一个天坑）

嗯嗯，很好。所以，现在的引擎上面undefined看起来长得跟null值差不多，而且在ECMAScript规范中它们都还是平级的（是原始值），而且它们的作用也很接近，最后他们都还是从最初的JavaScript 1.x中就存在的概念，但是undefined/null两者却在实现上完全不同：undefined是一个全局属性，而null是一个关键字。

由于undefined是全局属性，所以`delete undefined`其实就是`delete global.undefined`，是删除引用，而不是删除值。而这个属性是只读的，所以就返回false了。

### 参考文献

- [delete 操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/delete)
- [delete 0：JavaScript中到底有什么是可以销毁的](https://time.geekbang.org/column/article/164312)（付费）