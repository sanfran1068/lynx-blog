layout: post
title: "JavaScript高级程序设计笔记整理"
date: 2017-05-09 12:00:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories:
- Document
tags:
- JavaScript
- 基础知识
- 读书笔记
---

#### 第三章 基本概念

#### 语法：

*   **区分大小写**：ECMAScript中的一切都区分大小写
*   **标识符**：变量、函数、属性、函数参数等的名字，由字母，下划线，一个美元符号开头
*   **注释**：//单行注释，/* */多行注释（尽量少用该种注释方法）
*   **严格模式**：在脚本顶部或者某函数内顶部添加”use strict”,支持IE10+,Firefox4+，Safari5.1+，Opera12+，Chrome；

<!-- more -->

#### 语句

由分号结尾，多条语句以花括号包围形成代码块，有以下几种语句类型：

*   _if_ 语句：与Java中的条件判断语句语法相同
*   _do-while_ 语句：相同条件下会比while循环语句少执行一次
*   _while_ 语句：与Java中的条件判断语句语法相同
*   _for_ 语句：与Java中的条件判断语句语法相同
*   _for-in_ 语句：支持任何有迭代属性的对象（Object、Array、Set、Map等）进行枚举循环操作
*   _label_ 语句：添加标签一般与循环语句配合使用，使用break或continue语句引用

    ```javascript
    var temp=0;  
    start:  
    for(var i=0; i<5; i++) {  
      for(var m=0; m<5; m++) {  
          if(m==1) {  
              break start;  //输出1，若去掉start将输出5
          }  
          temp++;  
      }  
    }  
    alert(temp);
    ```

*   _break_ 和 _continue_ 语句：后面可引用label

*   _with_ 语句：将代码作用域设置到一个特定对象中，避免多次编写同一个对象的工作

    ```javascript
    with(someObject) {
      var qs = search.substring(1);   //如果没有使用with语句，这个表达式应该是var qs = someObject.search.substring(1);
      var hostName = hostname;
      var url = href;
    }
    ```

*   _switch_ 语句:与Java中的条件判断语句语法相同，注意使用**break**语句

#### 数据类型和变量

*   **关键字和保留字**：
    表达式和代码块中出现关键字和保留字一定要留意是否可用

*   **变量**：

    *   ECMAScript中变量为松散类型，var定义局部变量，可以不赋值（此时为undefined类型）
    *   无var关键词时定义的是全局变量（但是严格模式下会抛出错误）
*   **数据类型**：
    由typeof操作符可以显示给定变量的数据类型，有Undefined、Boolean、String、Number、Object（Array、Null）、Function六中数据类型；

    *   Undefined：未声量变量使用typeof操作符时显示undefined，而且对未声明变量使用alert()方法会报错；
    *   Boolean：true&false，注意各种数据类型对应boolean值得转换规则；
    *   Number：八进制0开头，十六进制0x开头；NaN不等于任何值（包括NaN本身）；非数值转换为数值的三个函数：number(),parseInt(“10”,进制基数)，parseFloat()；数值转换（true:1，false:0，null:0，undefined:NaN）；字符串转换规则（只包含有效数字转化为非0开头数值，空字符串转化为0，字符串包含除有效数字之外的转换为NaN）；
    *   String：字符串不可变，改变时会将原来的变量值销毁再重新赋值；数值、布尔值、对象和字符串值都有toString()方法；
    *   Object：都有Constructor、hasOwnproperty(propertyName)、isPrototypeOf(object)、propertyIsEnumerable(propertyName)、toLocaleString()、toString()、valueOf();
    *   Null：对值为null的变量使用typeof时返回object，因为null表示一个空对象指针；null==undefined总是返回true；
*   **操作符**：

    *   一元：++、–、+、-、~位、!逻辑、
    *   二元：&位、|位、^、<<、>>、>>>、&&逻辑、||逻辑、+、-、*、/、%、>、<、>=、<=、==、!=、===、!==、
    *   三元：boolean_expression？true_value:false_value;
    *   赋值：=、+=、-=、*=、/=、%=、<<=、>>=、>>>=

        **运算符优先级**：.、[]、()先于delete、new、typeof、+（一元）、—（一元）、！、~先于*、/、%先于+、-先于>=、<=、>、<先于===、!==先于&&先于||先于?:

*   **函数**：使用function关键词声明函数，参数类似数组并可省略，参数（和js中的基本类型）都是传值而非传引用（例如string对象）；名字相同的函数只有后定义的函数可以使用；

### 第四章 变量、作用域和内存问题

#### 基本类型和引用类型：

*   除Object之外的基本类型都是按值访问，对象则是通过访问内存中对象的引用；
*   基本类型无法添加和改变属性；
*   复制基本类型值会产生一个独立副本，而复制引用类型值时两者是指向同一个对象的指针；
*   检测基本类型值时使用typeof，**检测引用类型值** 时使用instanceof。另外，除了instanceof之外还可以使用return somevar.**proto**.constructor==sometype或者Object.prototype.toString.call(somevar)。

#### 执行环境和作用域：

*   局部执行环境（某个函数、let绑定代码块等）：某个执行环境中的所有代码执行完后，该环境被销毁，其中的所有变量和函数定义也随之消失；
*   全局执行环境：Web浏览器中，全局执行环境被认为是window对象；
*   每个函数也都有自己的执行环境，执行流（ECMAScript）进入函数，该函数环境被推入一个环境栈；
*   代码在一个环境中执行，会创建一个作用域链，用途是保证对执行环境有权访问的所有变量和函数有序访问；
*   延长作用域链：try-catch语句中的catch以及with语句
*   js中 **没有块级作用域**（任何花括号中的代码块），例如在if和for中的变量可以在if和for之外访问：
    1.  声明变量：使用var声明的变量自动被添加到最接近的环境中，没有用var声明会被添加到全局环境；
    2.  查询标识符：某环境中出现一个标识符时，要通过搜索确定该标识符代表什么，从作用域链前端开始，向上逐级查询，匹配到的第一个停止搜索，变量就绪；若全局中都没找到，则算未声明；

#### 垃圾回收：js具有自动垃圾收集机制

*   **标记清除**：变量进入某环境，标记该变量，离开环境时进行离开标记，释放内存；
*   **引用计数**：声明某变量并将引用类型值赋给该变量，该值引用次数为1，每被赋给一次就加1；若包含对这个值引用的变量取得另外一个值，则减1，该值变成0时就释放内存空间；
*   **管理内存**：为了占用最少内存达到更好性能，要将不再使用的变量赋值为null来释放其引用；

### 第五章 引用类型

#### Object类型

*   **创建对象时首选字面量创建方法**：

    ```javascript
    var person = {
      "name" : "Nicholas",
      "age" : 29,
      5 : true
    };
    ```

#### [](#Array类型 "Array类型")Array类型

*   **定义**：`var color = new Array(3);`

*   **访问**：采用数组下标[]的方式进行访问，使用.length得到数组长度；

*   **检测**：value instanceof Array(不推荐),_Array.isArray_(somearrayvalue)(推荐)；

*   **转换**：toLocaleString()/toString()/valueOf(),均生成 **逗号分隔** 的字符串其中，toLocaleString()方法转换时每一项都是使用toLocaleString()转换的，toString()也是Array.join(someJointSymbol)，默认用逗号连接；

*   **插入/删除**：除了直接用下标添加删除，最好用栈方法LIFO-push(任意项参数)在尾部增加任意数量参数，pop()删除最后一项；同时还有队列方法FIFO,在栈方法基础上增加shift()可删除第一项，unshift()可以在前端添加任意项；

*   **排序**：sort()升序排列（转化为字符串进行比较，数字比较会出错，需要在参数中提供比较函数compare()）；reverse()反转数组顺序；

*   **操作方法**

    *   **连接**：somearray.concat(otherarrays)返回连接所有数组的副本；

    *   **删除**：somearray.splice(起始位置，删除项数，插入项)-splice(0,2)删除前两项，splice(2,0,”red”,”green”)在当前数组位置2插入两项，splice(2,1,”red”,”green”)删除第二项并插入；

    *   **位置方法**：获取数组中元素的位置，indexOf()和lastIndexOf(),返回数组下表对应元素，否则返回-1；

    *   **迭代方法**：有5个迭代方法，参数都是两个，每一项上运行的函数和运行该函数的作用域对象every(someFunction)；对每一项进行函数操作，每一项返回true则最后返回true；filter(someFunction):对每一项运行给定函数，返回每次调用该函数值为true的项的数组forEach(someFunction):对每一项运行给定函数，无返回值map(someFunction):对每一项运行给定函数，返回每次调用该函数结果的数组some(someFunction):对每一项运行给定函数，只要有一项使函数返回true，则整个返回true；

    *   **缩小方法**：reduce(给每项调用的函数(可选)，缩小基础初始值)/reduceRight(给每项调用的函数(可选)，缩小基础初始值)，其中，参数中的函数需要传入4个参数，分别是前一项，当前项，项索引，数组对象；reduceRight是相反方向；

    *   **禁忌**：[， ， ， ，]不可以出现数组最后一项是，的情况

#### Date类型

*   **定义**：`var now = new Date();

*   **特定时间**：`var someDate = new Date(Date.parse(“May 25, 2004”));`但是不用date.parse也会默认转换；

*   **继承的方法**

    *   toLocaleString()按照与浏览器设置的地区相适应的格式返回日期和时间
    *   toString()返回带有时区信息的日期和时间
    *   valueOf()返回日期的毫秒表示
*   **日期格式化方法**

    *   toDateString():以特定于实现的格式返回日年月日星期
    *   toTimeString():以特定于实现的格式返回时区、时分秒
    *   toLocaleDateString():以特定于地区的格式返回年月日星期
    *   toLocaleTimeString():以特定于实现的格式显示时分秒
    *   toUTCString():以特定于实现的格式完整的UTC日期
*   日期时间组件方法
    ￼ ￼

#### RegExp类型：

*   **定义**：

```javascript
var pattern = new RegExp(“pattern”, “flags”);
var expression = /pattern/ flags;
```

其中flags有一下几种模式：

*   _g_:全剧模式，模式应用于所有字符串而非在发现第一个匹配项时立即停止；
*   _i_:不区分大小写模式；
*   _m_:多行模式，到达一行末尾还会查找下一行是否有匹配项

    *元字符转义：( { [ \ ^ $ | ) ? * + . ] }

    e.g.匹配第一个”[bc]at”:`var patter = /\[bc\]at/i;`

*   实例属性

    *   .global:bool是否设置g标志
    *   .ignoreCase:bool是否设置了i标志
    *   .lastIndex:整数，表示开始搜索下一个匹配项的自负位置，0算起
    *   .multiline:bool，表示是否设置了m标志
    *   .source:该正则的字符串表示
*   实例方法：.exec()

    *   构造函数属性：
    *   模式的局限性：

#### Function类型

函数是对象，因此函数名实际上也是一个指向函数对象的指针，可以使用多个指针指向同一个函数。

*   **声明**：

    ```javascript
    function sum(num1, num2){
      return num1 + num2;
    }
    ```

*   **特征**

    *   **没有重载**:因为JS中的函数名就像指针一样，所以当为一个函数名（就是一个变量）赋予新的函数时，会覆盖之前的函数；

    *   **声明位置和表达式**:

        JS中有函数声明提升过程，所以声明和调用不强调先后顺序。
        **但是！如果把函数放进一个初始化语句中而非函数声明时,会出现先后问题的错误；**

    *   **作为值的函数**:函数名可作为变量像值一样传入另一个函数；函数内也可以返回一个函数的计算结果等等；

    *   **函数属性和方法**:

        *   函数作为对象有两个属性:

            *   length（表示函数希望接受的命名参数的个数）
            *   prototype（保存引用类型的所有实例的方法，是不可枚举的）
        *   函数作为对象有两个方法:

            *   apply()：接受两个参数，一个是其中运行函数的作用域，另一个是参数数组（可以使arguments对象或者是真实的一个数组）
            *   call()：接受this参数以及来自函数的每一个参数都要写进去而非传入数组），这两个函数都是在特定的作用域中调用函数，相当于设置函数体内this对象的值
*   **函数内部对象及环境的属性**

    *   两个特殊对象

        *   arguments：包含传入函数的所有参数，并且有一个callee的属性指向拥有这个arguments对象的函数，其实就是这个函数，另一个caller的属性返回引用本函数的函数

        *   this：引用的是函数执行的环境对象，网页的全局就是window

#### 基本包装类型

String、Boolean、Number（Object、Null、Undefined、Array）

*   String：

    *   方法：

        *   下标相关
            *   所指的单字符串charAt()
            *   所指字符的编码charCodeAte()
            *   字符串位置indexOf(substr)
            *   最后一个字符串位置lastIndexOf(substr)
        *   字符串操作

            *   连接concat(String str1, String str2)
            *   截取slice(startPos(, endPos))
            *   截取**substr**(startPos(, length))
            *   截取substring(startPos(, endPos))，当输入负数的index时，slice将负数和字符串长度相加，substr将第一个负参数加字符串长度第二个负参数转换为0，substring将所有负参数转化为0；
        *   格式化字符串方法trim(string)创建一个副本并删除前后所有空格

        *   大小写相关

            *   转小写toLowerCase()
            *   转本地语言toLocaleLowerCase()
            *   toUpperCase()
            *   toLocaleUpperCase()
        *   字符串模式匹配方法match()详见正则表达式，search()返回第一个匹配项索引否则返回-1

        *   **replace(新字符串，需要替换的字符串)**

        *   split(simbol)，由simbol分隔字符串，返回一个字符串数组

        *   string1.localeCompare(string2)返回：string1在string2之前-1，相等0，之后1

        *   fromCharCode()静态方法，接收一或多个字符编码，然后转换成一个字符串

#### 单体内置对象

*   Global对象

    在js中，不属于任何其他对象的属性和方法，都是它的属性和方法。所有在全局作用域中定义的属性和函数都是它的属性和函数。例如isNaN()/isFinite()/parseInt()/parseFloat()/encodeURI()只编码空格/encodeURIComponet()编码所有非字母数字字符/decodeURI()/decodeURICompoment()/eval()接收一个js语句字符串，若是函数则不会被提升，需要按照声明顺序进行调用，严格模式下外部不能访问该函数中的函数或变量

### 第六章 面向对象的程序设计

对象就是无序属性的集合，其属性可以包含基本值、对象或者函数。创建对象的基础方法：

*   创建Object实例：

    ```javascript
    var person = new Object();
    person.name = "Nicholas";
    person.age = 29;
    person.sayName = function(){
      alert("this.name")
    }
    ```

*   对象字面量语法：

    ```javascript
    var person = {
      person.name = "Nicholas";
      person.age = 29;
      person.sayName = function(){
        alert("this.name")
      }
    }
    ```

#### 理解对象

*   属性类型：

ECMAScript中有两种属性：**数据属性** 和 **访问器属性**。

*   **数据属性**：包含一个数据值的位置，在此位置可读取和写入值，有以下特性：

    *   [[Configurable]]默认值为true，表示能否通过delete删除属性
    *   [[Enumerable]]默认值为true，表示能否通过for-in循环返回属性
    *   [[Writable]]默认值为true，表示能否修改属性的值
    *   [[Value]]默认值为undefined，包含这个属性的数据值

        要修改上述属性，必须通过_Object.defineProperty_(属性所在对象，属性名，描述符对象)方法：

        ```javascript
        var person = {};
        Object.defineProperty(person, "name", {
          writable: false,
          value: "Nicholas"
        })
        ```

*   **访问器属性**：不包含数据值，只包含getter和setter函数（这两个函数并非必需），访问器属性具有如下特征：

    *   [[Configurable]]默认值为true，表示能否通过delete删除属性
    *   [[Enumerable]]默认值为true，表示能否通过for-in循环返回属性
    *   [[Get]]默认值为undefined，读取属性时调用的函数
    *   [[Set]]默认值为undefined，写入属性时调用的函数
    *   访问器属性也必须使用Object.defineProperty()方法来进行定义：

        ```javascript
        var book = {
          _year: 2004,
          edition: 1
        };
        Object.defineProperty(book, "year", {
          get: function(){
            return this._year;
          },
          set: function(newValue){
            if(newValue > 2004){
              this._year = newValue;
              this.edition = newValue - 2004;
            }
          }
        });
        //若想定义多个属性或对应的方法时，可以这样写：
        Object.defineProperty(book, {
          //这里定义了两个数据属性
          _year: {
            value: 2004
          },
          edition: {
            value: 1
          },
          //这里定义了一个访问器属性
          year: {
            get: function(){
              return this._year;
            },
            set: function(newValue){
              if(newValue > 2004){
                this._year = newValue;
                this.edition = newValue - 2004;
              }
            }
          }
        });
        ```
        
    *   如果想读取属性的特征，则要通过Object.getOwnPropertyDescriptor()函数来访问：

        ```javascript
        var descriptor = Object.getOwnPropertyDescriptor(book, "_year");
        alert(descriptor.value);
        alert(typeof descriptor.get); //"function"
        ```

#### 创建对象

创建类和对象名要使用大写字母开头，构造函数也是，而非构造函数则由小写字母开头。

*   **工厂模式**：抽象了创建具体对象的过程，用函数来封装以特定接口创建对象的细节：

    ```javascript
    function createPerson(name, age, job){
      var o = new Object();
      o.name = name;
      o.age = age;
      o.job = job;
      o.sayName = function(){
        alert(this.name);
      };
      return o;
    }
    var person1 = createPerson("Nicholas", 29, "Software Engineer");
    ```

    其实就是把一个完整的由Object实例来创建对象的过程放在一个函数中；

*   **构造函数模式**：

    ```javascript
    function Person(name, age, job){
      this.name = name;
      this.age = age;
      this.job = job;
      this.sayName = function(){
        alert(this.name);
      };
    }
    var person1 = new Person("Nicholas", 29, "Software Engineer");
    ```

    可以看到，构造函数模式中 **没有显式地创建对象**，**直接将属性和方法赋给了this对象**，**没有return语句** 。

    构造函数模式相较于工厂模式的一个优点是，它创建的构造函数可以将它的实例标识为一种特定的类型，即对象也是Object也是Person的实例。

    将构造函数当做函数： 任何函数，只要通过new操作符来调用，就可以作为构造函数。

*   **原型模式**

    JavaScript中并没有提供类的实现，虽然在ES2015/ES6之中引入了class关键字，但是JavaScript仍然是基于原型的。而JavaScript中创建对象的方法有：new Object()方法、字面量方法、工厂模式、构造函数方法和**原型模式**。其中，使用原型模式创建对象可以令所有实例共享方法，减少内存消耗，有利于对象继承。

    JavaScript中的**继承**则是体现在一种结构上——对象，所有的对象都是由Object衍生的对象，所有的对象都继承了Object.prototype的方法和属性（也有可能被覆盖）。每一个对象都有一个内部链接到另一个对象，这个对象成为它的原型（prototype）。而且，该原型对象也有自己的原型，直到追溯到一个以null为原型的对象，因为null是没有原型的，所以可以作为这个**原型链（prototype chain)** 中的最终链接。

    只要创建了一个新函数，就会为该函数创建一个prototype属性，这个属性指向函数的原型对象。所有原型对象都自动获得一个constructor属性，这个属性包含一个指向prototype属性所在函数的指针。

    *   原型对象的创建：

        ```javascript
        function Person(name, age, job) {
            this.name = name;
            this.age = age;
            this.job = job;
        }
        Person.prototype.sayName = function() {
            alert(this.name);
        };
        var person1 = new Person("Nicholas", 29, "Lawyer");
        var person2 = new Person("Katie", 30 "Account");
        var person3 = new Person("Nicholas", 29, "Lawyer");
        person1.sayName();     //"Nicholas"
        person2.sayName();     //"Katie"
        alert(person1.sayName == person3.sayName);    //true
        ```

    *   更加简单的原型对象创建语法：

        ```javascript
        function Person() {
        }
        Person.prototype = {
          name : "Nicholas",
          age : 29,
          job : "Software Engineer",
          sayName : function() {
            alert(this.name)
          }
        };
        ```

*   **寄生构造函数模式**

    与工厂模式没有本质上的区别，只是在创建新的实例的时候，工厂模式不需要new，而寄生构造函数需要加new。这种模式不能够以来instanceof操作符来确定对象类型，所以建议在能够使用其他模式创建对象时尽量不要使用这种模式。

*   **稳妥构造函数模式**

    JavaScript中的稳妥对象是指，没有公共属性，而且其方法也不引用this的对象。稳妥对象最适合在一些禁止使用this和new的安全环境中使用，防止被Mashup等的应用程序改动。

    稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：一是新创建的对象的实例方法不引用this；二是不适用new操作符调用构造函数。

    ```javascript
    function Person(name, age, job) {
      var o = new Object();
      o.sayName = function() {
        alert(name);
      };
      return o;
    }
    ```

#### 继承

##### 原型链

*   代码搜索某属性的路径（原型链）：

    上面代码中出现了Person.prototype,通过该形式可以获得原型对象，并且可以为其添加属性和方法。这段代码也展示了对象属性和方法搜索的路径：每当代码读取某个对象的某个属性时，进行目标为该属性的名字，搜索首先从实例（person1）本身开始，若有该名字的属性则返回该属性，若没有则继续搜索该实例中原型指针指向的原型对象，在原型对象中查找该名字的属性。所以说**实例是共享原型中保存的属性和方法**。

*   实例的属性，原型的属性？

    一个对象的所有实例尽管可以共享该原型对象的所有属性和方法，但是却无法重写原型对象中的属性和方法，只能通过利用同名属性和方法来覆盖和屏蔽原型对象中的属性和方法；若想把实例中覆盖的属性还原，可以通过delete操作。使用hasOwnProperty()方法可以检测一个属性是存在于实例中还是存在于原型中。

*   怎样获得原型对象

    下面的代码展示了Object.getPrototypeOf(obj)方法和**proto**属性的使用：

    ```javascript
    Object.getPrototypeOf(person1) === Person.prototype;    //true
    person1.__proto__ === Person.prototype;    //true
    ```

    可以看到，Object.getPrototypeOf(obj)是获取obj对象的原型对象的方法，这个方法将在**利用原型实现继承的情况中发挥非常重要的作用**。obj.**proto** 也是如此，是每一个对象都拥有的属性，但是**proto**并不是一个规范的属性（当使用Object.create()方法创建对象时，**proto** 并不能指向该对象的原型对象），其对应的标准属性应当是[[Prototype]]。

*   原型对象和构造函数

    对于对象Person来说，它的构造函数是Person()，它的原型对象为Person.prototype；而Person.prototype.constructor又会指回Person。而对象Person的实例person1和person2都包含有一个属性[[Prototype]]（也就是上面提到的**proto**）它们都指向Person.prototype；同时，person1和person2也可以通过isprototypeOf(）方法来确定是否与确定对象之间有这种关系：

    ```javascript
    alert(Person.prototype.isPrototypeOf(person1));    //true
    ```

*   原型与in操作符

    *   in操作符单独使用的情况

        in操作符单独使用会在通过对象能够访问给定属性时返回true，无论该属性存在于实例中还是在原型中：

        ```javascript
        //接上面的代码
        var person4 = new Person();
        alert(person4.hasOwnproperty("name"));    //由于person4实例没有覆盖原型中的name所以返回false
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
    var keys2 = Object.getOwnPropertyNames(Person.prototype);    
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
    alert(person1.friends);    //此时两个Person的实例拥有相同的朋友，很可能发生错误
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
     constructor : Person,
     sayName : function() {
       alert(this.name);
     }
    }
    ```

### 第七章 函数表达式

函数的定义有两种方法：一种是函数声明，另一种是**函数表达式** 。

*   函数声明：

```javascript
function functionName(arg0, arg1, arg2) {
  //函数体\
}
```

Firefox、Safari、Chrome和Opera都给函数定义了一个非标准的name属性，可以访问到给函数指定的名字alert(functionName.name);同时我们还需要注意的是，函数有函数声明提升的重要特征，所以函数声明可以放在函数调用语句之后。

*   函数表达式：

```javascript
var functionName = function(arg0, arg1, arg2) {
  //函数体
};
```

这种情况下创建的函数叫做**匿名函数**f，所以函数的name属性是空字符串，而且根据赋值语句的特性，调用一定要在函数赋值语句之后。既然函数可以赋值给变量，同时函数也可以作为其他函数的返回值。

#### 递归

*   概念

    递归函数是在一个函数通过名字调用自身的情况下构成的。

    ```javascript
    function factorial(num) {
      if (num <= 1) {
        return 1;
      }else{
        return num * factorial(num - 1);
      }
    }
    ```

    这是一个经典递归阶乘函数，但是存在以下的问题：

    ```javascript
    var anotherFactorial = factorial;
    factorial = null;
    alert(anotherFactorial(4));   //出错
    ```

    上面调用出错的原因是，当factorial的引用被替换之后，在anotherFactorial调用时还是会用到factorial，这时候就会发生错误。这种情况可以使用arguments.callee可以解决这个问题。

    arguments.callee是一个指向正在执行的函数的指针。所以用这个指针对递归调用能够解决这个问题，例如：

    ```javascript
    function factorial(num) {
      if(num <= 1) {
        return 1;
      } else {
        return num * arguments.callee(num - 1);
      }
    }
    ```

    如果是严格模式下，callee模式会导致错误，最好使用命名函数表达式来达到相同的目的：

    ```javascript
    var factorial = (function f(num) {
      if(num <= 1) {
        return 1;
      } else {
        return num * f(num - 1);
      }
    });
    ```

#### 闭包

*   概念：闭包是指有权访问另一个函数作用域中的变量的函数。

*   创建闭包：常见方式是在一个函数内部创建另一个函数。

*   理解闭包

    要理解闭包的细节，必须理解如何创建作用域链以及作用域链的作用以及理解函数第一次被调用的时候发生了什么。

    当某函数第一次被调用时，会创建一个执行环境以及相应的作用域链，并把作用域链赋值给一个特殊的内部属性（[[Scope]])，然后可以使用this、arguments和其他命名参数的值来初始化函数的活动对象。在作用域链中，外部函数的活动对象始终处于第二位，外部函数的外部函数活动对象处于第三位，直至作为作用域链重点的全局执行环境。

    所以说，当函数执行完毕之后，据不活动对象（this、arguments）就会被销毁，内存中也仅保存全局作用域。而引入闭包之后情况就会好转：

    在另一个函数内部定义的函数会将包含函数（外部函数）的活动对象添加到它的作用域链中。当调用该函数时，函数执行完毕之后，其活动对象也不会被销毁，因为其中的匿名函数的作用域链仍然在引用这个活动对象，除非使用null来销毁该函数解除对匿名函数的引用，释放内存。

*   闭包与变量

    作用域链的机制有一个副作用：闭包只能取得包含函数中任何变量的最后一个值。闭包所保存的是整个变量的对象，而不是某个特殊的变量。

    这是因为匿名函数始终都在内存中保存着作用域链中该函数的活动对象，所以其实所有新建的函数都使用着同一个变量。除非在该匿名函数中再写一个匿名函数才能强制让闭包的行为符合预期。

*   this对象

    在闭包中使用this对象也会出现问题：**匿名函数的执行环境具有全局性**，因此匿名函数的this对象通常指向window（全局变量）。这是因为内部匿名函数在搜索变量时，根据作用域链进行搜索，所以存在在对象属性的变量是无法访问到的（只有函数才有作用域，还是因为外部函数没有显示声明this对象？），内部函数只能找到全局，所以如果想要闭包的函数中访问到外部函数的this对象，必须要将this对象保存在外部函数中。

*   内存泄漏

    IE9之前的版本中，如果闭包的作用域链中保存着一个HTML元素，那么该元素无法被销毁。

    避免发生”循环引用“的情况。

#### 模仿块级作用域

JavaScript中没有块级作用域的概念，所有的块语句中定义的变量都存在于函数中的作用域。所以以下代码是可以正常访问的：

```javascript
function outputNumbers(count){
  for(var i = 0;i < count;i++){
    alert(i);  
  }
  alert(i);   //返回count计数
}
```

就算在最后一个alert(i)之前重新定义声明变量i也不会改变i的值。所以JS从来不会提醒是否多次声明了同一个变量，有可能会导致问题的产生。所以这时候需要匿名函数来模仿块级作用域来避免这个问题:

```javascript
(function(){
  //这里就是块级作用域
})();
```

以上代码定义并立即调用了一个匿名函数，前一个括号是函数表达式声明，后一个括号是指立即调用这个函数。需要注意的是，函数表达式后面可以跟圆括号，而函数声明后面是不可以加圆括号的。

#### 私有变量

JS中没有私有成员的概念，对象属性和函数都是共有的，但是有一个私有变量的概念。**任何在函数中定义的变量，都可以认为是私有变量**。所以利用函数的闭包就可以创建用于访问私有变量的公有方法。该方法成为”特权方法”，创建方式有以下两种：

*   基本模式

```javascript
function MyObject(){
  var privateVariable = 10;
  function privateFunction(){
    return false;
  }
  this.publicMethod = function(){
    privateVariable++;
    return privateFunction();
  };
}
```

##### 静态私有变量

在构造函数中定义特权方法也有一个缺点，那就是必须使用构造函数模式来达到该目的，可是构造函数模式的缺点就是每个实例都会创建同样一组新方法。而使用静态私有变量来实现特权方法就可以避免这个问题。

通过在私有作用域中定义私有变量或函数，同样也可以创建特权方法：

```javascript
(function(){
  var privateVariable = 10;
  function privateFunction(){
    return false;
  }
  MyObject = function(){
  };    //严格模式下未经声明的变量赋值会导致错误
  MyObject.prototype.publicMethod = function(){
    privateVariable++;
    return privateFunction();
  };
})();
```

##### 模块模式

**单例** 是指只有一个实例的对象，通常JavaScript是用对象字面量的方式来创建单例对象。

而模块模式正是专门为单例创建私有变量和特权方法的方式：

```javascript
var singleton = function(){
  var privateVariaable = 10;
  function privateFunction(){
    return false;
  }
  return {
    publicProperty: true,
    publicMethod: function(){
      privateVariable++;
      return privateFunction();
    }
  };
}();
```

这种模式在需要对单例进行某些初始化，同时又需要维护其私有变量时时非常有用的。单例经常被用在Web应用程序中来管理应用程序级的信息。

另外还有 **增强的模块模式**

### 第八章 BOM

如果在Web中使用JavaScript，那么BOM才是真正的核心。

#### window对象

BOM的核心对象是window，它表示浏览器的一个实例。在浏览器中，window对象不仅是通过JavaScript访问浏览器窗口的一个接口，又是ECMAScript规定的Global对象。

*   全局作用域

在JavaScript中，全局作用域中声明的变量、函数都会变成window对象的属性和方法。然而定义全局变量和在window对象上直接定义属性仍有一点区别：全局变量无法通过delete操作符删除（IE<9抛错，其他浏览器返回false），而window对象上的属性可以。

*   窗口关系及框架

如果页面中包含frame，则每一个框架都拥有自己的window对象，并保存在frames集合中。

*   窗口位置

用来确定和修改window对象位置的属性和方法：

IE、Safari、Opera和Chrome都提供screenLeft和screenTop属性；Chrome和Safari以及Firefox则支持screenX和screenY属性：

```javascript
var leftPos = (typeof window.screenLeft == "number") ? window.screenLeft : window.screenX;
var topPos = (typeof window.screenTop == "number") ? window.screenTop : window.screenY;
```

上述代码对浏览器进行选择并获取窗口的位置。默认返回左上角原点（0，0）。

通过winow.moveBy(横坐标长度，纵坐标长度)，window.moveTo(横坐标，纵坐标)，可以移动窗口，但是可能会被浏览器禁用。

*   窗口大小

IE9+/Firefox/Safari/Opera/Chrome均提供4个属性：innerWidth/innerHeight/outerWidth/outerHeight。其中在IE9+/Safari/Firefox中，outerwidth和outerHeight返回浏览器窗口本身吃葱，Opera中则是表示页面视图容器的大小。而innerWidth、innerHeight是表示该容器中页面视图去的大小。Chrome中内外两对属性返回相同值-viewport大小。

另外，还可以使用window.resizeTo(width,height)/window.resizeBy(widthtoformer,lengthtoformer)

*   导航和打开窗口

window.open()方法可以导航到特定URL或打开新浏览器窗口。

```javascript
window.open("http://www.wrox.com/",topFrame,"someString","1")
//四个参数分别是网站的url、窗口目标（或者_self/_parent/_top/_blank），特性字符串，是否取代当前页面的bool
```

如果该方法中第二个参数不存在，就会根据在第三个参数为止传入的字符串创建一个新窗口，有“fullscreen=yes、height=100”等参数，该参数字符串中不允许出现空格。该方法可以按照下面的方式赋值给变量并进行一些操作：

```javascript
var wroxWin = window.open("http://www.wrox.com/","wroxWindow","height=400,width=400,top=10,left=10,resizable=yes");
alert(wroxWin.opener == window);    //true
wroxWin.resizeTo(500,500);
wroxWin.moveTo(20,20);
wroxWin.close;
alert(wroxWin.closed);    //true
```

鉴于有很多网站有很多弹窗广告的窗口，所以为了屏蔽这些窗口，网站的代码中可以

*   间歇调用和超时调用

JS是单线程语言，但是允许通过设置超时值和间歇时间值（毫秒）来调度代码在特定时刻执行。其中，超时值是在指定时间过后执行代码，后者是每隔指定时间就执行一次代码。

```javascript
setTimeout(function() {
  alert("Hello world!");
}, 1000);
//第一个参数可以是字符串格式的代码，但是不推荐使用，会导致性能损失
setInterval(function(){
  alert("Hello World!");
},10000);
//每10000毫秒执行一次代码
```

该方法也可以赋值给变量，然后通过clearTimeout(timeoutVar)/clearinterval(intervalVaar)

*   系统对话框

浏览器通过alert(),confirm(),prompt()方法可以调用系统对话框向用户显示消息。其中confirm()方法有OK和cancel两个按钮，prompt()会提示用户输入一个参数。另外还有window.print()打印对话框和window.find()查找对话框。

#### location对象

location是最有用的BOM对象之一，提供了与当前窗口中加载的文档有关的信息和导航功能，还将URL解析为独立片段来通过不同属性访问这些片段。

location既是window对象的一个属性，又是document对象的属性，即window.location=document.location。下面是location的所有属性：

hash（#contents）、host（www.wrox.com:80）、hostname（www.wrox.com）、href（[http://www.wrox.com）、pathname（/WileyCDA）、port（8080）、protocol（http:）、search（?q=javascript）](http://www.wrox.com）、pathname（/WileyCDA）、port（8080）、protocol（http:）、search（?q=javascript）);

*   查询字符串参数（对search属性进行字符串解析）

*   位置操作

    通过`location.assign("http://www.wrox.com");`可以打开新的URL，与`window.location="http://www.wrox.com";`以及`location.href="http://www.wrox.com";`相同

    如果要禁止将页面存入浏览记录（防止后退到前一个页面），可以使用location.replace(URL);

#### navigator对象

*   检测插件

    navigator.plugins是一个数组，其中每一项都包括name, description, filename, length

    但是IE中检测插件需要使用ActiveXObject类型：

    ```javascript
    function hasIEPlugin(name) {
        try{
            new ActivXOBject(name);
            return true;
        } catch(e) {
            return false;
        }
    }
    alert(hasIEPlugin("shockwaveFlash.ShockwaveFlash"));

    ```

*   注册处理程序

    navigator对象有registerContentHandler()和registerProtocolHandler()方法；

    registerContentHandler()接受三个参数：要处理的协议（mailto或ftp），处理该协议的页面URL和应用程序的名称：

    ```javascript
    navigator.registerProtocolHandler("mailto", "http://www.somemailclient.com?cmd=%s", "Some Mail Client");
    ```

#### screen对象

对于变成来说作用不大，用来标明客户端的能力，包括浏览器窗口外部的显示其信息等；

#### history对象

使用history.go()接受参数整数，一个url（跳到最近的该url页面）
另外也可以使用history.back()/.forward();

### 第九章 客户端检测

#### 能力检测

目标是识别浏览器的能力，

*   基本模式如下：

    ```javascript
    if(object.propertyInQuestion) {
        //使用object.propertyInQuestion
    }
    ```

    使用方案：先检测达成目的的最常用的特性（提升效率），必须测试实际要用到的特性（判断条件要够详细）

    如果是想要判断对象是否有某个属性或者支持某种方法，尽量使用typeof

#### 怪癖（bug）检测

目标是识别浏览器的特殊行为。

#### 用户代理检测

通过检测用户代理字符串来确定实际使用的浏览器。用户代理字符串作为响应头部发送，也可以通过navigator.userAgent属性访问

用户代理字符串检测技术：

```javascript
var client = function() {
    var engine = {
        ie: 0,
        gecko: 0,
        webkit: 0,
        khtml: 0,
        opera: 0,
        ver: null
    };
    var browser = {
        ie: 0,
        firefox: 0,
        safari: 0,
        konq: 0,
        opera: 0,
        chrome: 0,
        ver: null
    };
    var system = {
        win: false,
        mac: false,
        x11: false，
        iPhone：false,
        ipod: false,
        ipad: false,
        ios: false,
        android: false,
        nokiaN: false,
        winMobile: false
    }; //navigator.platFORM
    //再次检测呈现引擎、平台和设备
    var ua = navigator.userAgent;
    if(window.opera) {
        engine.ver = window.opera.version();
        engine.opera = parseFloat(engine.ver);
    } else if(/AppleWebKit\/(\S+)/.test(ua)) {
        engine.ver = RegExp["$1"];
        engine.webkit = parseFloat(engine.ver);
    } else if(（/KHTML\/(\S+).test(ua)) {
        engine.ver = RegExp["$1"];
        engine.khtml = parseFloat(enging.ver);
    } else if(/rv:([^\)]+)\) Gecho\/d{8}/.test(ua)) {
        engine.ver = RegExp["$1"];
        engine.gecko = parseFloat(engine.ver);
    } else if(/MSIE ([^;]+)/.test(ua)) {
        engine.ver = RegExp["$1"];
        engine.ie = parseFloat(engine.ver);
    }
    return {
        engine：engine,
        browser: browser,
        system: system
    };
}();

```

可以检测用户代理、浏览器的类型和版本、系统类型和版本、移动设备

### 第十章 DOM

#### 节点层次

HTML页面中，文档元素始终都是元素。

#### 节点类型

DOM1级定义了Node借口，由DOM中的所有节点类型实现。在JS中作为Node类型实现，**除了IE**，其他所有浏览器中都可以访问到这个类型。节点类型一共有12种。

使用方法：

```javascript
if(someNode.nodeType == Node.ELEMENT_NODE) {
    alert("Node is an element.");   //不适用于IE
}
if(someNode.nodeType == 1) {
    alert("Node is an element.");   //全部适用
}
```

*   nodeName和nodeValue属性

    对于元素节点，nodeName保存元素的标签名，nodeValue中则为null

*   节点关系

    可以使用.childNode[]数组属性、item()方法来访问子节点：

    ```javascript
    var firstChild = someNode.childNodes[0];
    var secondChild = someNode.childNodes.item(1);
    var count = someNode.childeNodes.length;
    ```

    可以使用parentNode属性访问父节点，使用previousSibling和nextSibling属性访问兄弟节点，但是**列表中第一个节点的previousSibling属性值为null，最后一个节点的nextSibling属性值也为null**

    可以使用firstChild和lastChild属性访问childNodes列表中第一个和最后一个节点。

*   操作节点

    appendChild()方法新增节点：`someNode.appendChild(newNode)`，该方法返回新增的节点，若传入的参数节点已经存在，则该方法对该节点进行转移

    insertBefore()方法插入特定位置，接受两个参数，一个是要插入的节点，第二个是参照节点

    replaceChild()方法替换节点

    removeChild()方法移除节点

    cloneNode(一个bool参数表示是否执行深复制)、normalize()支持所有类型节点

#### Document类型

JS中通过Document类型表示文档，document对象是HTMLDocument（继承自Document类型）的一个实例，表示整个HTML页面，也是window对象的一个属性。Document节点具有下列特征：

*   nodeType值为9
*   nodeName的值”document“
*   parentNode的值为null
*   nodeValue的值为null
*   ownerDocument的值为null

##### 文档的子节点

可以通过**documentElement属性**（始终指向\<html\>元素）和childNodes列表访问文档元素。</html\>

可以通过**document.body来访问\<body\>元素，这两个属性所有浏览器均支持；</body\>

可以通过**document.doctype访问\<!DOCTYPE>的引用，但是IE8之前不支持，而其他浏览器这个属性不存在在childNodes中

\<html\>标签之外的注释也算文档的子节点，但是IE8/SAFARI/OPERA/CHROME均只为第一条注释创建节点</html\>

##### 文档信息

##### 查找元素（最最常用）

可以通过getElementById(“”)访问指定元素,ID大小写在IE7之前有要求。如果多个相同ID的元素出现，只返回第一个元素（IE7会在name等于指定id时且在这个id元素之前出现，返回有该name的元素）

可以通过getElementsByTagName(“”)访问指定元素列表，返回HTMLCollection对象，可以使用数组的下标访问，也可以通过item()方法进行访问，**也可以通过该对象的namedItem(name)方法返回相同tag元素列表中指定的name元素**。

可以通过getElementsByName()访问

##### 特殊集合

*   document.anchors
*   document.applets
*   document.forms
*   document.images
*   document.links

##### DOM一致性检测

##### 文档写入

write()/writeln()/open()/close()是document对象的IO功能，这些函数是在加载过程当中写入的，所以加载完再运行write会导致整个页面消失：

`document.write("<script type=\"text/javascript\" src=\"file.js\">" + "<\/script>");`

#### Element类型

要访问元素的标签名，最好使用tagName属性，但是注意这里tagName返回的是**大写**！

*   HTML元素（取得特性）

    由HTMLElement类型表示，继承自Element类型，拥有id/title/lang/dir/className等属性

*   操作特性

    操作特性的DOM方法主要有：div.getAttribute(“属性名除了style和onclick”)/div.setAttribute(“属性名”,”属性值”)/div.removeAttribute(“属性名”)

*   attributes属性

*   创建元素

    使用document.createElement()方法创建新元素，只接受一个参数为要创建的标签名

#### Text类型

### 第十三章 事件

#### 事件流

事件流描述的是从页面中接收事件的顺序。

*   事件冒泡

    IE的事件流叫做事件冒泡：时间开始时由最具体的元素（嵌套层次最深的那个节点）接收，然后逐级向上传播直到文档根节点（window对象）。所有现代浏览器都支持事件冒泡。

*   事件捕获

    从不太具体的节点开始接收事件，逐级到最具体的节点最后接收到事件。很少使用事件捕获。

*   DOM事件流

    包括三个阶段：事件捕获阶段、处于目标阶段、事件冒泡阶段。

#### 事件处理程序

*   HTML事件处理程序

    ```html
    <form method="post">
        <input type="text" name="username" value="">
        <input type="button" value="Echo Username" onclick="alert(username.value)">
    </form>
    ```

    缺点：如果调用的函数在引用之后定义，页面解析该函数之前就进行了操作会引发错误（时差问题）；不同JS引擎遵循的标识符解析规则略有差异，很可能会在访问非限定对象成员时出错。

*   DOM0级事件处理程序

    ```javascript
    var btn = document.getElementById("myBtn");
    btn.onclick = function() {
        alert("Clicked");
    }
    ```

*   DOM2级事件处理程序

    定义了addEventListener()和removeEventListener()用于添加和删除监听器的函数。都接受三个参数：处理事件名，事件处理函数，和一个bool值（该值为true捕获阶段调用，false冒泡阶段调用）。但是add方法添加的程序 只能用remove来删除，所以当add一个匿名函数时就无法被删除。

    ```javascript
    var btn = document.getElementById("myBtn");
    btn.addEventListener("click", function() {
            alert(this.id);
        }, false);
    ```

    可添加多个事件，按顺序出发。

*   IE事件处理程序

    IE中使用attachEvent()，事件处理程序会在**全局作用域**中进行，所以函数中出现的this等于window，而DOM0中的函数中出现的this就是局部作用于内运行。也有detachEvent()移除函数，所以道理与DOM2级事件处理程序一样。

    ```javascript
    var btn = document.getElementById("myBtn");
    btn.attachEvent("onclick", function() {
        alert("Clicked");
        });
    ```

*   跨浏览器的事件处理程序

    1.  创建addHandler()方法，接受三个参数：要操作的元素、事件名称和事件处理程序函数；目的是判断情况应该使用DOM0/DOM2/IE的哪种方法，该方法属于EventUtil对象。

#### 事件对象

*   DOM中的事件对象

    兼容DOM的浏览器会将一个event对象传入到事件处理程序中。

    ```javascript
    var btn = document.getElementById("myBtn");
    btn.onclick = function(event) {
        alert(event.type);      //"click"
    };
    btn.addEventListener("click", function(event) {
        alert(event.type);      //"click"
        }, false)
    ```

    事件处理程序内部的this永远都是currentTarget，也就是现在事件作用的目标（按钮、window等）；

    一个函数处理多个事件，可以使用switch(event.type)函数；阻止特定时间的默认行为，可以使用preventDefault()方法；停止事件在DOM层次中传播停止事件的捕获和冒泡，可以调用stopPropagation()：

    ```javascript
    var btn = document.getElementById("myBtn");
    btn.onclick = function(event) {
        alert("Clicked");
        event.stopPropagation();    //避免了下面body上的事件触发
    };
    document.body.onclick = function(event) {
        alert("Body clicked");
    };
    ```

    事件对象的eventPhase属性可以用来确定事件当前正位于事件流的哪个阶段。捕获阶段调用程序为1，程序正在目标对象上为2，在冒泡阶段调用程序为3

*   IE中的事件对象

    ```javascript
    var btn = document.getElementById("myBtn");
    btn.onclick = function() {
        var event = window.event;
        alert(event.type);      //"click"
    }
    ```

    由于事件处理程序的作用域在IE下是全局，在DOM下是局部作用域，所以提供event.srcElement来判断this的指向：

    ```javascript
    var btn = document.getElementById("myBtn");
    btn.onclick = function() {
        alert(window.event.srcElement === this);        //true
    };
    btn.attachEvent("onclick", function(event) {
        alert(event.srcElement === this);               //false
        });
    ```

    若想要取消时间的默认行为，可以通过设置`window.event.returnValue = false`进行；

    若想要阻止时间的冒泡（IE只支持事件冒泡），可以通过设置`window.event.cancelBubble = true;`

*   跨浏览器的事件对象

#### 事件类型

*   UI事件

    *   **load**(html中可以给标签添加onload
    *   unload
    *   abort
    *   error
    *   select
    *   resize
    *   scroll
*   焦点事件

    *   **blur**
    *   **focus**
    *   focusin
    *   focusout
*   鼠标与滚轮事件

    *   click
    *   dbclick
    *   mousedown
    *   mouseenter
    *   mouseleave
    *   mousemove
    *   mouseout
    *   mouseover
    *   mouseup
*   键盘与文本事件

    *   keydown
    *   keypress
    *   keyup
*   复合事件

    *   compositionstart
    *   compositionupdate
    *   compositionend
*   HTML5事件

    *   contextmenu
    *   beforeunload
    *   DOMContentLoaded
    *   readystatechange
    *   pageshow/pagehide
    *   hashchange