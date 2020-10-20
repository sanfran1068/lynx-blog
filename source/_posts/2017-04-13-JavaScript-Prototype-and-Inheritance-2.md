layout: post
title: "JavaScript中的原型与继承-继承篇"
date: 2017-04-13 12:00:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories: 
- Document
tags:
- JavaScript
- 原型
- 基础知识
- 继承
- 读书笔记
---

#### 原型链

原型链是实现继承的主要方法，基本思想是**利用原型让一个引用类型继承另一个引用类型的属性和方法**。回顾构造函数、原型和实例之间的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型的内部指针。

此时我们**令原型对象成为另一个对象的实例，该原型对象将包含一个指向另一个原型对象的指针**，若是这样层层递进，就会构成一条实例和原型的链条，这就是原型链的基本概念。

<!-- more -->

![](http://oqcytejyk.bkt.clouddn.com/image2.jpg)

由上图可以看出，原型链中，每个对象都有一个内部指针指向原型对象，而原型对象内部也会有指针指向另一个原型对象。图中新创建的Fruit和继承Fruit的Apple和Banana这样的对象的原型对象，首先会追溯到Object，而通过JavaScript的控制台使用Object.getPrototypeOf(obj)的方法得出Object的原型是function，而function的原型又会指向到Object，最后Object的原型最终会是null，到此为止null再也不会有原型对象。

这幅图也能很形象地展示出之前讲到过的原型搜索机制，即，当以读取模式访问一个实例属性时，1）首先搜索实例；2）搜索该实例的原型对象；3）搜索该实例的原型对象的原型对象；以此类推，直到找到null。需要注意的是：

*   所有引用类型都默认继承了Object；
*   通过instanceof操作符来确定原型和实例之间的关系；
*   由于子类继承过程中需要重写父类中的某些方法或者添加不存在的方法，**给原型添加方法的代码一定要放在原型继承的语句之后**；
*   在通过原型链实现继承时，不能使用对象字面量创建原型方法；
*   由于原型的实例都共享原型中的属性和方法，所以会导致原型中可能包含的引用类型（Function、Array）也被迫同步给每一个实例，所以需要一些解决办法。

### 继承

#### 继承方式

##### 经典继承

```javascript
function SuperType() {
  this.colors = ["red", "blue", "green"];
}
function SubType() {
  //继承了SuperType
  SuperType.call(this);
}
var instance1 = new SubType();
instance1.colors.push("blaack");
alert(instance1.colors);    //"red,blue,green,black"
var instance2 = new SubType();
alert(instance2.colors);    //"red,blue,green"
```

该方法，子类中使用call函数调用了父类中的函数，当然也可以向其中传入相应的参数。然而这个方法需要将所有的方法都在构造函数中定义，造成了函数复用无法实现。

##### 组合继承（原型链+构造函数）

参考上一篇文章中的“原型对象的缺点”以及上一点“经典继承”，结合起来就是组合继承的方法。

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
};
function SubType(name, age) {
  SuperType.call(this.name);
  this.age = age;
}
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;  //这一步是必须的吗？
SubType.prototype.sayAge = function() {
  alert(this.age);
};
```

然而，组合继承也有一定的缺陷，即，无论何种情况下进行继承，都会调用量词父类的构造函数（一次是创建子类原型，其次是子类构造函数内部）

##### 原型式继承

原理是使用一个函数将一个对象进行复制，具体代码如下：

```javascript
//将传入的对象o进行复制，副本传给一个新的构造函数
function object(o) {
  function F(){}
  F.prototype = 0;
  return new F();
}
```

而ES5中新增了Object.create()方法来规范原型式继承，该方法接受两个参数，第一个是所要复制的对象，第二个是属性描述符定义的属性和属性值。（不支持IE8及以下浏览器）

同样的，原型式继承也存在新创建的实例都必须默认共享原型对象的属性和方法，对于引用类型的属性是存在缺陷的。

##### 寄生式继承

寄生式继承的思路与寄生构造函数和工厂模式类似，即，创建一个仅用于封装继承过程的函数，该函数返回集成之后的对象。

```javascript
function createAnother(originalObj) {
  var cloneObj = object(originalObj);
  clone.sayHi = function() {
    alert("Hi");
  };
  return cloneObj;
}
```

##### 寄生组合式继承

为了解决组合继承中必须调用两次父类对象的构造函数这一问题，提出了寄生组合式继承。寄生组合式继承的思路是，**通过借用构造函数来几成熟型，通过原型链的混成形式来继承方法**。本质上，是使用寄生式继承来继承超类型的原型，再将结果指定给子类型的原型。其基本模式如下：

```javascript
function inheritPrototype(subType, superType) {
  var prototype = Object.create(superType.prototype);    //创建父类副本
  prototype.constructor = subType;    //加强副本对象
  subType.prototype = prototype;    //完成原型继承父类副本
}
```

该方式之调用一次SuperType的构造函数，与此同时，原型链仍保持不变（仍能够使用instanceof和isPrototypeOf()方法确定继承关系），所以该方法目前来说是引用类型最理想的继承方式。

### 关于原型与继承的总结

在JavaScript的编程过程中，需要考虑创建的对象是否需要被继承，如果有这样的需要，我们应当及时地使用组合模式（构造函数创建包含引用类型的属性，原型来创建该对象的函数方法）来创建该对象（构造函数）；在继承的过程中，最好采用寄生组合式继承的方法来进行对象的继承，以达到提高效率且保护原型链的目的。

在考虑原型和继承的过程中，需要牢记实例、对象（构造函数）和原型三者之间的关系。