layout: post
title: "JavaScript中的原型与继承-原型篇"
date: 2017-04-12 12:00:00
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

#### 原型

构造函数模式下，对象中的方法作用域只在对象中，对象的不同实例的同名函数都是不相等的。为了解决这个问题，js中创建的每一个函数都有一个prototype属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所由实例共享的属性和方法。

<!-- more -->

#### 原型对象

JavaScript中并没有提供类的实现，虽然在ES2015/ES6之中引入了class关键字，但是JavaScript仍然是基于原型的。而JavaScript中创建对象的方法有：new Object()方法、字面量方法、工厂模式、构造函数方法和**原型模式**。其中，使用原型模式创建对象可以令所有实例共享方法，减少内存消耗，有利于对象继承。

JavaScript中的**继承**则是体现在一种结构上——对象，所有的对象都是由Object衍生的对象，所有的对象都继承了Object.prototype的方法和属性（也有可能被覆盖）。每一个对象都有一个内部链接到另一个对象，这个对象成为它的原型（prototype）。而且，该原型对象也有自己的原型，直到追溯到一个以null为原型的对象，因为null是没有原型的，所以可以作为这个**原型链（prototype chain**中的最终链接。

只要创建了一个新函数，就会为该函数创建一个prototype属性，这个属性指向函数的原型对象。所有原型对象都自动获得一个constructor属性，这个属性包含一个指向prototype属性所在函数的指针。

![](http://oqcytejyk.bkt.clouddn.com/image1.jpg)

##### 原型对象的创建

```javascript
function Person(name, age, job) { 
     this.name = name; 
     this.age = age; 
     this.job = job; 
 } 
 Person.prototype.sayName = function() { 
     alert(this.name); 
 } 
 var person1 = new Person("Nicholas", 29, "Lawyer"); 
 var person2 = new Person("Katie", 30 "Account"); 
 var person3 = new Person("Nicholas", 29, "Lawyer"); 
 person1.sayName();     //"Nicholas" 
 person2.sayName();     //"Katie"
 alert(person1.sayName == person3.sayName);    //true 
```

更加简单的原型对象创建语法：

```javascript
function Person(){}
Person.prototype = { 
   name : "Nicholas", 
   age : 29, 
   job : "Software Engineer", 
   sayName : function() { 
     alert(this.name) 
   } 
}; 
```

* 代码搜索某属性的路径：

    上面代码中出现了Person.prototype,通过该形式可以获得原型对象，并且可以为其添加属性和方法。这段代码也展示了对象属性和方法搜索的路径：每当代码读取某个对象的某个属性时，进行目标为该属性的名字，搜索首先从实例（person1）本身开始，若有该名字的属性则返回该属性，若没有则继续搜索该实例中原型指针指向的原型对象，在原型对象中查找该名字的属性。所以说**实例是共享原型中保存的属性和方法**。

* 实例的属性，原型的属性？

    一个对象的所有实例尽管可以共享该原型对象的所有属性和方法，但是却无法重写原型对象中的属性和方法，只能通过利用同名属性和方法来覆盖和屏蔽原型对象中的属性和方法；若想把实例中覆盖的属性还原，可以通过delete操作。使用hasOwnProperty()方法可以检测一个属性是存在于实例中还是存在于原型中。

* 怎样获得原型对象:
    
    下面的代码展示了Object.getPrototypeOf(obj)方法和**proto**属性的使用：
    
    ```javascript
    Object.getPrototypeOf(person1) === Person.prototype;     //true 
    person1.__proto__ === Person.prototype;    //true 
    ```

可以看到，Object.getPrototypeOf(obj)是获取obj对象的原型对象的方法，这个方法将在**利用原型实现继承的情况中发挥非常重要的作用**。obj.**proto** 也是如此，是每一个对象都拥有的属性，但是**proto**并不是一个规范的属性（当使用Object.create()方法创建对象时，**proto** 并不能指向该对象的原型对象），其对应的标准属性应当是[[Prototype]]。

##### 原型对象和构造函数

对于对象Person来说，它的构造函数是Person()，它的原型对象为Person.prototype；而Person.prototype.constructor又会指回Person。而对象Person的实例person1和person2都包含有一个属性[[Prototype]]（也就是上面提到的**proto**）它们都指向Person.prototype；同时，person1和person2也可以通过isprototypeOf(）方法来确定是否与确定对象之间有这种关系：

<figure class="highlight javascript">

<pre>

<div class="line">alert(Person.prototype.isPrototypeOf(person1));    <span class="comment">//true</span></div>

</pre>

</figure>

##### 原型与in操作符

*   in操作符单独使用的情况

    in操作符单独使用会在通过对象能够访问给定属性时返回true，无论该属性存在于实例中还是在原型中：

    ```javascript
    //接上面Person对象代码 
    var person4 = new Person(); 
    alert(person4.hasOwnproperty("name"));     //由于person4实例没有覆盖原型中的name所以返回false 
    alert("name" in person4);    //true 
    ```

*   for-in循环

    该种方式是返回所有能够通过对象访问的、可枚举的属性，包括实例和原型中的属性，并且屏蔽了原型中不可枚举（[[Enumerable]]）的属性（仅在IE8及更早版本中有不可枚举的属性）：

    ```javascript
    var o = { 
       toString : function() { 
         return "my Object" 
       } 
     } 
     for (var prop in o) { 
       if (prop == "toString") { 
         alert("Found toString");      //IE中无法显示 
       } 
     }     
    ```

    若想获取对象中所有可枚举的实例属性，可以使用Object.keys(),该函数接受一个对象或实例作为参数（当然也可以是原型对象），返回一个包含所有可枚举属性的字符串数组。示例代码如下：

    ```javascript
    //接Person对象代码 
    var keys = Object.keys(person.prototype); 
    var person5 = new Person(); 
    person5.name = "Rob"; 
    person5.age = 31; 
    alert(keys); 
    alert(Object.keys(person5)); 
    var keys2 = Object .getOwnPropertyNames(Person.prototype);     
    alert(keys2); 
    ```

    上述代码中出现的Object.getOwnPropertyNames(Person.prototype)方法可以得到所有的可枚举/不可枚举的实例属性（列入constructor）。

##### 原型的动态性

由于从原型中查找值的过程试一次搜索，因此对原型对象所做的任何修改都能从实例上反映出来。即，可以先创建实例，再修改原型对象中的属性或函数，实例依旧可以访问该属性和函数：

```javascript
//接person对象代码 
 var friend = new Person(); 
 Person.prototype.sayHi = function() { 
   alert("hi!"); 
 }; 
 friend.sayHi();  //返回”hi” 
```

尽管像以上代码中所看到的可以为原型添加属性或方法，但是如果重写整个原型对象，就会切断实例的构造函数与最初原型对象之间的联系。这是因为实例中的指针仅指向原型，而非构造函数。

##### 原生对象的原型

不仅是自定义的对象，JavaScript中所有原生的引用类型，都是采用这种模式创建的原生对象。通过原生对象的原型可以取得所有默认方法的引用，也可以自己添加新的方法。

#### 原型对象的缺点

*   由于原型中所有属性都是可以被实例共享，而对于原型中含有引用类型值的属性来说就会发生问题：

    ```javascript
    //接person对象代码 
    Person.prototype.friends = ["Shelby", "Court"]; 
    var person6 = new Person(); 
    person6.friends.push("Van"); 
    alert(person6.friends); 
    alert(person1.friends);     //此时两个Person的实例拥有相同的朋友，很可能发生错误 
    ```

    可以看到，由于原型的属性可以被共享的这一特性，原型对象中包含的引用类型值很可能被修改之后导致实例的属性也发生错误，必须要在实例属性中覆盖才可以。所以为了解决这个问题，提出了以下组合构造函数模式和原型模式的方法来创建对象：

    ```javascript
     //改写上面的Person对象 
     //首先使用构造函数模式创建对象Person并加入容易被修改的属性name，age，job 
     function Person(name, age, job) { 
       this.name = name; 
       this.age = age; 
       this.job = job; 
       this.friends = ["Shelby", "Court"]; 
     } 
     //再使用原型模式创建构造函数和不容易被修改的属性和函数，这样可以发挥原型的共享机制并减少内存消耗 
     Person.prototype = { 
       constructor : Person,    //将构造函数放入原型中 
       sayName : function() { 
         alert(this.name); 
       } 
     }
    ```