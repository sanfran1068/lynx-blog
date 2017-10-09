layout: post
title: "ES6入门必须了解的知识（上）"
date: 2017-08-02 12:00:00
banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg
comments: true
categories: 
- Document
tags:
- ES6
- JavaScript
- 基础知识
---

最近一段时间的实习期间，团队进行了一次ESLint规范的调整和文档改进，里面涉及到了非常多ES6的内容，而在之前热力图项目中也用到了像Promise对象、async函数等的知识，所以专门参考阮一峰写的[《ECMAScript6入门》](http://es6.ruanyifeng.com/)一书对ES6的一些重要知识点进行一次整理。
<!-- more -->

##### Babel转码器

Babel是可以将ES6代码转为ES5代码的转码器。

*   配置文件 **.babelrc**

    该配置文件是使用Babel工具和模块必须的，存放在项目的根目录下，用来设置转码**规则**和**插件**，基本格式如下

    ```javascript
    {
        "presets": [],
        "plugins": []
    }
    ```

    其中presets字段设定转码规则，官方提供以下的规则集，你可以根据需要安装。一般安装最新的的规则集 `npm install --save-dev babel-preset-latest` ，然后，配置文件应该如下

    ```javascript
    {
        "presets": [
          "latest",
          "react",
          "stage-2"
        ],
        "plugins": []
    }
    ```

*   命令行转码**babel-cli**

    1.  安装命令： `npm install --global babel-cli`
    2.  基本用法（全局环境下）：
        *   转码结果输出到标准输出： `babel example.js`
        *   转码结果写入一个文件： `babel example.js --out-file compiled.js`
        *   整个目录转码： `babel src --out-dir lib`
*   建议将babel-cli安装在项目目录下

    1.  安装命令： `npm install --save-dev babel-cli`;
    2.  改写package.json
    
        ```javascript
        {
        	"devDependencies": {
        		"babel-cli": "^6.0.0"
        	},
        	"scripts": {
        		"build": "babel src -d lib"
        	}
        }
        ```

    3. 转码命令（在项目目录下）： `npm run build`;

*   ES6的REPL环境**babel-node**：

    babel-node随babel-cli工具安装，执行 `babel-node` 进入REPL环境可以直接运行命令了

*   babel-register

    babel-register木块改写require命令，可以使文档中使用require加载的.js/.jsx/.es/.es6后缀的文件先进行babel转码：

    1.  局部安装： `npm install --save-dev babel-register`
    2.  使用（前必须先加载babel-register）：

        ```javascript
        require("babel-register");
        require("./index.js");
        ```

*   babel-core：当代码需要调用Babel的API进行转码时

    1.  安装： `npm install babel-core --save`
    2.  项目中调用：

        ```javascript
        var babel = require('babel-core');
           
        // 字符串转码
        babel.transform('code();', options);
        // => { code, map, ast }
           
        // 文件转码（异步）
        babel.transformFile('filename.js', options, function(err, result) {
         result; // => { code, map, ast }
        });
           
        // 文件转码（同步）
        babel.transformFileSync('filename.js', options);
        // => { code, map, ast }
           
        // Babel AST转码
        babel.transformFromAst(ast, code, options);
        // => { code, map, ast }
        ```

*   babel-polyfill

    Babel 默认只转换新的 JavaScript 句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。如果想让这个方法运行，必须使用**babel-polyfill**

*   Traceus转码器

    *   直接插入网页
    *   在线转换
    *   命令行转换

##### 常量和变量

除了ES5提出的var和function两种声明变量的方法，ES6提出了let和const命令，以及import命令和class命令。所以ES6中一共有六种声明变量的方法。

##### let和const命令

*   let命令

    ES6新增了let命令，用来声明变量。它的用法类似于var，但是所声明的变量，只在let命令所在的**代码块**内有效。

    *   基本用法：特别适合在for循环计数器中使用，而且循环体是循环语句的自作用域：

        ```javascript
        var a = [];
        for (let i = 0; i < 10; i++) {
        	a[i] = function () {
        	  console.log(i);
        	};
        }
        a[6](); // 可以避免闭包中只能读取最后一个值
        	
        for (let i = 0; i < 3; i++) {
        	let i = 'abc';
        	console.log(i);
        }		// abc
        ```

* 特点

    - 不存在变量提升（变量提升是指变量在声明之前可以使用**但是值为undefined**）
    - 暂时性死区：只要块中有let声明变量语句，该变量在该块中就必须服从let语句的规则（const也是如此），所以typeof也是不安全的
    - 不允许重复声明：包括在相同作用域内禁止重复声明同一个变量（只要出现let就不允许，包括var），以及函数内部重新声明参数

*   块级作用域

    由于内层变量可能会覆盖外层变量，用来计数的循环变量泄露成为全局变量。

    let实际上为JavaScript新增了块级作用域：

    ```javascript
    function f1() {
    	let n = 5;
    	if (true) {
    	  let n = 10;
    	}
    	console.log(n); // 5
    }
    ```

- 特点

    - 块级作用域可以任意嵌套，但是外层作用域无法读取内层作用域的变量
    - 内层作用域可以定义外层作用域的同名变量
    - 实际上让立即执行函数表达式不再必要（直接将需要立即执行的函数内部代码写在{}块级作用域中即可）

- 块级作用域与函数声明

    ES6明确允许在块级作用域中声明函数（在ES5中函数声明只能在函数作用域或全局作用域中进行），且块级作用域中的函数声明行为类似于let，，但必须在有大括号的情况下成立，且在块级作用域外不可引用。

    但是浏览器不遵守上述规定，有自己的行为方式：

    - 允许在块级作用域内声明函数
    - 函数声明类似于var，即会提升到全局作用域或函数作用域的头部
    - 同时，函数声明还会提升到所在的块级作用域的头部

    所以**尽量不要**在块级作用域内声明函数，若非要声明，也应该写成**函数表达式**而非函数声明语句。

- do表达式

    块级作用域内的let声明的变量无法在作用域外被访问，所以提出了do表达式，可以使块级作用域表现为表达式并有返回值：

    ```javascript
    let x = do {
    	let t = f();
    	t * t + 1;
    };
    ```

*   const命令

    声明一个只读的常量，声明之后该值不能改变。所以一旦声明必须赋值！

    特点：

    1.  不会被提升；
    2.  存在暂时性死区
    3.  将一个引用变量声明为常量需要小心（该常量指向引用变量的地址，但是里面的内容是可以修改的，所以对象常量不可以被重新赋值为新的引用变量）

        如果想让对象以及对象里的内容只读，那么需要使用 `const foo = Object.freeze({});`

*   顶层对象的属性

    顶层对象，在浏览器环境指的是**window对象**，在Node指的是**global对象**。ES5之中，顶层对象的属性与全局变量是等价的。但是这样存在很多问题：

    1.  没法在编译时报出变量未声明的错误（只能运行时报错）
    2.  容易莫名其妙创建全局变量
    3.  顶层对象可以随处读写，不利于模块化编程

        所以ES6提出了：

    4.  var命令和function命令声明的全局变量依旧是顶层对象的属性

    5.  let命令、const命令和class命令声明的全局变量不属于顶层对象的属性：

        ```javascript
        let b = 1;
        window.b // undefined
        ```

*   global对象

    ES5中的顶层对象，在浏览器中是window，浏览器和WebWorker里面self也指向顶层对象，在Node里面是global

    对于this变量来说，ES5全局环境this返回顶层对象；Node和ES6模块返回当前模块；函数作为对象的方法运行指向对象，否则指向顶层对象，严格模式下返回undefined；

#### 变量的解构赋值

*   数组的解构赋值

    用法： `let[a, b, c] = [1, 2, 3];`

    只要两边的数组结构模式相同，那么就可以使用这种数组解构赋值。

    两种特殊情况：

    *   左边模式变量多于右边数组元素变量，解构不成功，返回undefined
    *   左边模式变量少于右边数组元素变量，解构成功
    *   等号右边不是数组（严格讲，可遍历的数据结构），报错

        事实上，只要具有Iterator解构的数据结构，都可以采用数组的形式进行解构赋值，例如 `let [x, y, z] = Set(['a', 'b', 'c']);` ,

        **默认值**：解构赋值允许指定默认值，使用表达式来赋值，但是默认值对应的数组中的位置如果不是严格等于undefined，默认值不会生效：

        ```javascript
        let [x, y = 'b'] = ['a']; // x='a', y='b'
        let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
        ```

默认值是一个惰性求值的**表达式**，只有右边数组中没有或者undefined时才会求值

```javascript
function f() {
  console.log('aaa');
}
let [x = f()] = [1];    //此处f()不会求值，因为右边已经赋值成功
    
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError
```

*   对象的解构赋值

    按照对象的**同名**属性可以进行正确得解构赋值

    ```javascript
    let { bar, foo } = { foo: "aaa", bar: "bbb" };
    foo     // "aaa"
    bar    // "bbb"
        
    let { baz } = { foo: "aaa", bar: "bbb" };
    baz // undefined
    ```

其真正的机制为： `let { foo: foo, bar: bar } = { foo: "aaa", bar: "bbb" };` 注意冒号后面的才是真正的变量

所以 `let { foo: baz } = { foo: "aaa", bar: "bbb" };` 中baz才被赋值

这里需要注意的是由let进行声明，不可以重新声明，可以使用var替换，也可以试用一下的方式：

```javascript
let foo;
({foo} = {foo: 1}); // 成功，圆括号必须有，否则会认为是一个代码块而非赋值语句
let baz;
({bar: baz} = {bar: 1}); // 成功
// 错误的写法
let x;
{x} = {x: 1};
// SyntaxError: syntax error, {}会被理解成一个代码块
    
// 正确的写法
({x} = {x: 1});
```

解构赋值失败会返回undefined，与数组相同，对象也可以用于嵌套结构，牢记**冒号前的一般为模式**，要分清模式和真正的变量；对象也可以有默认值，默认值的表达式也是惰性求值（任何非undefined的值都会对默认值进行覆盖），常用例子： `let { log, sin, cos } = Math;`

数组本质是特殊的对象，所以对象解构赋值等号右边可以是一个数组。


*   字符串的解构赋值

    *   字符串分解为数组： `const [a, b, c, d, e] = 'hello';`
    *   字符串（是一个对象）属性解构赋值：`let {length : len} = 'hello';`
    *   数值和布尔值的解构赋值

        由于Number和Bool值都是值类型，所以解构赋值前需要转换为引用类型，比如可以使用toString方法：

        ```javascript
        let {toString: s} = 123;
        s === Number.prototype.toString // true
        ```

        由于undefined和null无法转为对象，所以无法进行解构赋值

*   函数参数的解构赋值

    函数参数的解构赋值就是在函数的参数位置进行解构赋值，可以使用上面提到的数组解构赋值、对象解构赋值、字符串解构赋值等，支持默认值（默认值是当解构失败会返回的值），也可以直接传入数组、对象

    ```javascript
    function move({x = 0, y = 0} = {}) {
      return [x, y];
    }   				//该处为x,y指定默认值
        
    move({x: 3, y: 8}); // [3, 8]
    move({x: 3}); 		// [3, 0]
    move({}); 			// [0, 0]
    move(); 			// [0, 0]
        
    function move({x, y} = { x: 0, y: 0 }) {
      return [x, y];
    }   				//该处为函数的参数指定默认值，这里要进行深入研究
        
    move({x: 3, y: 8}); // [3, 8]
    move({x: 3}); 		// [3, undefined]
    move({}); 			// [undefined, undefined]
    move(); 			// [0, 0]
     ```

*   圆括号问题：尽量不要再解构赋值的模式（等号左边的一堆）中放置圆括号

    **不能使用圆括号**的三种情况

    *   变量声明语句中模式不能带有圆括号
    *   函数参数中模式不能带有圆括号
    *   赋值语句中，不能在模式外/嵌套模式中的一层套圆括号

        **可以使用圆括号**的情况

    *   只有一种：赋值语句的非模式部分可以使用圆括号

*   解构赋值的用途

    *   交换变量的值

        ```javascript
        let x = 1;
        let y = 2;
        [x, y] = [y, x];
        ```

- 从函数返回多个值

    ```javascript
    // 返回一个数组
    function example() {
    return [1, 2, 3];
    }
    let [a, b, c] = example();
      
    // 返回一个对象
    function example() {
    return {
      foo: 1,
      bar: 2
    };
    }
    let { foo, bar } = example();
    ```
    
- 函数参数的定义

    ```javascript
    // 参数是一组有次序的值
    function f([x, y, z]) { /*...*/ }
    f([1, 2, 3]);
        
    // 参数是一组无次序的值
    function f({x, y, z}) { /*...*/ }
    f({z: 3, y: 2, x: 1});
    ```
    
- **提取JSON数据**（都支持嵌套结构）

    ```javascript
    let jsonData = {
      id: 42,
      status: "OK",
      data: [867, 5309]
    };
        
    let { id, status, data: number } = jsonData;
        
    console.log(id, status, number);
    // 42, "OK", [867, 5309]
    ```
    
- 函数参数默认值

    ```javascript
    jQuery.ajax = function (url, {
      async = true,
      beforeSend = function () {},
      cache = true,
      complete = function () {},
      crossDomain = false,
      global = true,
      // ... more config
    }) {
      // ... do stuff
    };
    ```
    
- 遍历Map结构

    ```javascript
    var map = new Map();
    map.set('first', 'hello');
    map.set('second', 'world');
        
    for (let [key, value] of map) {
      console.log(key + " is " + value);
    }
    // first is hello
    // second is world
      
    // 获取键名
    for (let [key] of map) {
    // ...
    }
      
    // 获取键值
    for (let [,value] of map) {
    // ...
    }
    ```
    
- 输入模块的指定方法：`const { SourceMapConsumer, SourceNode } = require("source-map");`


##### 字符串的扩展

*   字符的Unicode表示法

    ES5只限制采用码点在\u0000~\uFFFF之间的字符，ES6规定只要将码点放入大括号，就能正确解读（最后JS有z \z \172 \x7A \u007A \u{7A}六种表示字符方法）：

    ```javascript
    "\u{20BB7}"
    // "𠮷"
        
    "\u{41}\u{42}\u{43}"
    // "ABC"
        
    let hello = 123;
    hell\u{6F} // 123
        
    '\u{1F680}' === '\uD83D\uDE80'
    // true
    ```

*   codePointAt()

    JS内部字符以UTF-16的格式储存，每个字符固定为2个字节。大于2个字节的字符，charAt()和charCodeAt()都不能读取整个字符，所以引入了codePointAt()，返回十进制码点。想返回十六进制，可以.toString(16)

*   String.fromCodePoint()

    可识别大于0xFFFF的字符，与上面的codePointAt()方法正好相反

*   字符串遍历器接口

    ES6可以使用for a of b结构对字符串循环遍历：

    ```javascript
    var text = String.fromCodePoint(0x20BB7);
      
    for (let i = 0; i < text.length; i++) {
    console.log(text[i]);
    }
    // 无法识别0xFFFF的码点
      
    for (let i of text) {
    console.log(i);
    }
    // "𠮷"
    ```

*   .at()

    使用垫片库实现`'𠮷'.at(0)`对0xFFFF字符进行charAt()操作

*   .normalize()

    用来将字符的不同表示方法统一为同样的形式，这称为Unicode正规化。

    ```javascript
    '\u01D1'.normalize() === '\u004F\u030C'.normalize()
    // true
        
    /*该函数可以接收NFC(合成形式)/NFD(分解形式)/NFKC/NFKD作为参数*/
    ```

*   .includes(), .startsWith(), .endsWith()

    都是找字符串中是否是包含、以…开始、以…结束的，均返回bool值，接收两个参数，第一个参数为寻找的子串，第二个参数为开始（end是结束）寻找的父串的下标：

    ```javascript
    var s = 'Hello world!';
      
    s.startsWith('world', 6) // true，这里要注意
    s.endsWith('Hello', 5) // true
    s.includes('Hello', 6) // false
    ```

*   .repeat()

    接收一个参数作为重复次数n（n>-1），返回一个重复n次的新字符串：

*   .padStart(), .padEnd()

    接收两个参数，一个是指定字符串长度，第二个是用来补全的”补丁字符串”（若补丁字符串超出长度则截取，若第二个参数省略用空格补全，不想要的话使用.trim()）：

    ```javascript
    'x'.padStart(5, 'ab') // 'ababx'
    'x'.padEnd(4, 'ab') // 'xaba'
    ```

*   模板字符串

    ES6引入模板字符串，使用反引号（`）来标识（可嵌套），可以当做普通字符串，也可以用来定义多行字符串（多行字符串中的空格和缩进都会保留在输出之中）：

    ```javascript
    // 普通字符串
    `In JavaScript '\n' is a line-feed.`
      
    // 多行字符串
    `In JavaScript this is
    not legal.`
      
    console.log(`string text line 1
    string text line 2`);
      
    // 字符串中嵌入变量使用${表达式、对象属性甚至调用函数等}，牢记{}中最后返回的都是字符串
    var name = "Bob", time = "today";
    `Hello ${name}, how are you ${time}?`
        
    //字符串里出现`需要转义
    var greeting = `\`Yo\` World!`;
    ```
    
*   模板编译

*   标签模板

    模板字符串可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串。这被称为“标签模板”功能（tagged template）。“标签”指的就是**函数**，紧跟在后面的模板字符串就是它的参数。

    *   当模板字符串里有变量，会将模板字符串先处理成多个参数，再调用函数

        处理结果是(stringArray, value1, value2)，处理过程是将所有非变量的部分存在一个字符串数组中，剩下的变量依次一个个排在后面:

        ```javascript
        var total = 30;
        var msg = passthru`The total is ${total} (${total*1.05} with tax)`;
          
        function passthru(literals) {
        var result = '';
        var i = 0;
          
        while (i < literals.length) {
          result += literals[i++];
          if (i < arguments.length) {
            result += arguments[i];
          }
        }
        return result;
        }
          
        msg // "The total is 30 (31.5 with tax)"
        ```

"标签模板"重要应用：

1. 过滤HTML字符串，防止用户输入恶意内容
2. 多语言转换（国际化处理）
3. 嵌入其他语言

*   String.raw()

    用来充当模板字符串的处理函数，返回一个斜杠都被转义（斜杠前面再加一个斜杠）的字符串，对应于替换变量后的模板字符串，源码如下：

    ```javascript
    String.raw = function (strings, ...values) {
    	var output = "";
    	for (var index = 0; index < values.length; index++) {
    		output += strings.raw[index] + values[index];
    	}
    	  
    	output += strings.raw[index]
    	return output;
    }
    ```

    该函数接受第一个参数是有raw数组属性的的对象

*   模板字符串的限制：默认会将字符串转义，所以无法嵌入其他语言（例如LaTeX），但是标签模板可以

##### 正则的扩展

*   RegExp构造函数

    ES5中，RegExp构造函数如下：

    *   参数是字符串，这时第二个参数表示正则表达式的修饰符（flag）`var regex = new RegExp('xyz', 'i');`
    *   参数是一个正则表示式，这时会返回一个原有正则表达式的拷贝`var regex = new RegExp(/xyz/i);`

        ES6允许有第二个参数（flag）：`var regex = new RegExp(/xyz/ig, 'i');`，且第二个参数会覆盖表达式中的修饰符

*   字符串的正则方法：

    字符串对象共有4个方法，可以使用正则表达式：match()、replace()、search()和split()。ES6将这4个方法，在语言内部全部调用RegExp的实例方法，从而做到所有与正则相关的方法，全都定义在RegExp对象上。

    *   `String.prototype.match` 调用 `RegExp.prototype[Symbol.match]`，以此类推…
*   u修饰符

    含义为Unicode模式，处理大于\uFFFF的Unicode字符

    *   点字符：对于码点大于0xFFFF的Unicode字符，点字符不能识别，必须加上u修饰符

        ```javascript
        var s = '𠮷';
          
        /^.$/.test(s) // false
        /^.$/u.test(s) // true
        ```

    *   Unicode字符表示法：使用大括号表示的unicode字符需要加u修饰符`/\u{20BB7}/u.test('𠮷')`
    *   预定义模式
    *   i修饰符：添加u才能够识别非规范的字母
    
*   y修饰符

    粘连修饰符，y修饰符的作用与g修饰符类似，也是全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始。不同之处在于，g修饰符只要剩余位置中存在匹配就可，而y修饰符确保匹配必须从剩余的第一个位置开始，这也就是“粘连”的涵义。

    ```javascript
    var s = 'aaa_aa_a';
    var r1 = /a+/g;
    var r2 = /a+/y;
      
    r1.exec(s) // ["aaa"]
    r2.exec(s) // ["aaa"]
      
    r1.exec(s) // ["aa"]
    r2.exec(s) // null
    ```

*   sticky属性

*   flags属性

*   RegExp.escape()

*   s修饰符：dotAll模式

*   后行断言

*   Unicode属性类

##### 数值的扩展

*   二进制和八进制的表示法

    二进制0b/0B，八进制0o/0O

*   Number.isFinite(Number), Number.isNaN(Number)

    分别检查一个数值是否为有限的、是否为NaN

*   Number.parseInt(Number), Number.parseFloat(Number)

    分别将参数（字符串）转换为int和float数据类型

*   Number.isInteger()

    用来判断一个值是否为整数（3和3.0都可以返回true）

*   Number.EPSILON()

    一个极小的常量，用来设置一个误差范围来精确浮点数计算：

    ```javascript
    function withinErrorMargin (left, right) {
    	return Math.abs(left - right) < Number.EPSILON;
    }
    withinErrorMargin(0.1 + 0.2, 0.3)		// true
    withinErrorMargin(0.2 + 0.2, 0.3)		// false
    ```

*   安全证书和Number.isSafeInteger()

    JavaScript能够准确表示的整数范围在-2^53到2^53之间（不含两个端点），超过这个范围，无法精确表示这个值。Number.isSafeInteger()则是用来判断一个整数是否落在这个范围之内。**验证运算结果是否落在安全整数的范围内，不要只验证运算结果，而要同时验证参与运算的每个值。**

*   Math对象的扩展

    *   Math.trunc方法用于去除一个数的小数部分，返回整数部分,可以接收数字和数字字符串
    *   Math.sign方法用来判断一个数到底是正数、负数、还是零
    *   Math.cbrt方法用于计算一个数的立方根
    *   Math.clz32方法返回一个数的32位无符号整数形式有多少个前导0
    *   Math.imul方法返回两个数以32位带符号整数形式相乘的结果，返回的也是一个32位的带符号整数
    *   Math.fround方法返回一个数的单精度浮点数形式
    *   Math.hypot方法返回所有参数的平方和的平方根
    *   Math.expm1(x)返回ex - 1，即Math.exp(x) - 1
    *   Math.log1p(x)方法返回1 + x的自然对数，即Math.log(1 + x)
    *   Math.log10(x)返回以10为底的x的对数
    *   Math.log2(x)返回以2为底的x的对数。如果x小于0，则返回NaN

    *   ES6新增了6个三角函数方法：

        *   Math.sinh(x) 返回x的双曲正弦（hyperbolic sine）
        *   Math.cosh(x) 返回x的双曲余弦（hyperbolic cosine）
        *   Math.tanh(x) 返回x的双曲正切（hyperbolic tangent）
        *   Math.asinh(x) 返回x的反双曲正弦（inverse hyperbolic sine）
        *   Math.acosh(x) 返回x的反双曲余弦（inverse hyperbolic cosine）
        *   Math.atanh(x) 返回x的反双曲正切（inverse hyperbolic tangent）
*   Math.signbit()

    Math.signbit()方法判断一个数的符号位是否设置了

*   指数运算符

    两个乘号代表指数运算符

##### 数组的扩展

*   扩展运算符
    扩展运算符（spread）是三个点（…），可将一个数组转为用逗号分隔的参数序列。所以该运算符可以用来代替apply调用函数的方式,apply方法是将数组展开为函数的参数，扩展运算符也可以达到此目的。

    扩展运算符的应用主要有：

    1.  合并数组：`[...arr1, ...arr2, ...arr3]`
    2.  与解构赋值结合：`const [first, ...rest] = [1, 2, 3, 4, 5];`
    3.  函数返回多个值；
    4.  将字符串转化为数组：能够正确识别32位的Unicode字符，最好使用扩展运算符；
    5.  实现了Iterator接口的对象：
    6.  Map、Set解构和Generator函数都可以使用扩展运算符
    
*   Array.from()

    Array.from方法用于将两类对象转为真正的数组：类似数组的**对象**（DOM操作返回的NodeList集合以及函数内部arguments对象）和可遍历（iterable）的对象（包括ES6新增的数据结构**Set**和**Map**），第一个参数接受一个类数组对象，第二个参数接受一个类map方法对对象进行操作。

    ```javascript
    // arguments对象
    function foo() {
    	var args = Array.from(arguments);
    	// ...
    }
    ```    
       
       也可以用来进行对数组的复制`arrayCopy = Array.from([1,2,3]);`
       
       扩展运算符（...）也可以将某些数据结构转为数组（必须是部署了Symbol.iterator遍历器接口的对象）：
       
    ```javascript
    // arguments对象
    function foo() {
    	var args = [...arguments];
    }
      
    // NodeList对象
    [...document.querySelectorAll('div')]
    ```

*   Array.of()

    ES5中，Array方法没有参数、一个参数、三个参数时，返回结果都不一样。只有当参数个数不少于2个时，Array()才会返回由参数组成的新数组。参数个数只有一个时，实际上是指定数组的长度。

    Array.of方法用于将一组值，转换为数组。

    ```javascript
    Array.of() 			// []
    Array.of(undefined) // [undefined]
    Array.of(1) 		// [1]
    Array.of(1, 2) 		// [1, 2]
    ```

*   数组实例的copyWithin()

    它接受三个参数，start和end指定要复制的子串，target指定从哪里开始替换:`[1, 2, 3, 4, 5].copyWithin(0, 3) // [4, 5, 3, 4, 5]`

    *   target（必需）：从该位置开始替换数据。
    *   start（可选）：从该位置开始读取数据，默认为0。如果为负值，表示倒数（加数组长度即可）。
    *   end（可选）：到该位置**前**停止读取数据，默认等于数组长度。如果为负值，表示倒数。

*   数组实例的find()和findIndex()

    数组实例的find方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员。如果没有符合条件的成员，则返回undefined。

    ```javascript
    [1, 5, 10, 15].find(function(value, index, arr) {
    	return value > 9;
    }) // 10
    ```

```
findIndex()与find()用法相同，返回寻找到的成员下标

```

*   数组实例的fill()

    fill方法使用给定值，填充一个数组：

    ```javascript
    ['a', 'b', 'c'].fill(7)			// [7, 7, 7]
    ['a', 'b', 'c'].fill(7, 1, 2)	// ['a', 7, 'c']
    ```

*   数组实例的entries(), key()和values()

    ES6提供三个新的方法——entries()，keys()和values()——用于遍历数组。它们都返回一个遍历器对象（详见《Iterator》一章），可以用for…of循环进行遍历，唯一的区别是keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。

*   数组实例的includes()

    Array.prototype.includes方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似。该方法属于ES7，但Babel转码器已经支持。

*   数组的空位

    数组的空位指，数组的某一个位置没有任何值。比如，Array构造函数返回的数组都是空位。空位不是undefined，undefined是属于有值的！

##### 函数的扩展

*   函数参数的默认值

    ```javascript
    function Point(x = 0, y = 0) {
    	this.x = x;
    	this.y = y;
    }
      
    var p = new Point();
    p // { x: 0, y: 0 }
    ```

    如果参数默认值是变量，那么参数就不是传值的，而是每次都重新计算默认值表达式的值。也就是说，参数默认值是惰性求值的。

    ```javascript
    let x = 99;
    function foo(p = x + 1) {
    	console.log(p);
    }
      
    foo() // 100
      
    x = 100;
    foo() // 101
    ```

- 与解构赋值默认值结合使用

    ```javascript
    function foo({x, y = 5}) {
          console.log(x, y);
        }
    foo({x: 1}) 	// 1, 5
    ```
    
    **在某些情况下，传入undefined时才会触发参数默认值**
    
- 函数的length属性：指函数预期传入参数的个数，返回**没有指定默认值时**的参数的个数（指定了默认值后length属性失真，从指定默认值的那以为开始失真，而之前没有指定默认值的还是会计入length）：`(function (a, b = 1, c) {}).length 	// 返回1`
        
- 参数默认值和作用域
    	
    一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域（context）。等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的。
    
    ```javascript
    var x = 1;
    function f(x, y = x) {
      console.log(y);
    }
    
    f(2) 	// 由于参数初始化自己形成单独作用域，所以y=x=2，不考虑函数外的全局变量
    
    let a = 1;
    
    function f(b = a) {
    let a = 2;
    console.log(b);
    }
    
    f() // 在这个作用域里面，变量x本身没有定义，所以指向外层的全局变量x
    ```   
    
    这个参数默认值的作用域有一个非常不错的应用途径：指定某参数不可忽略，否则抛出错误提示：
    
    ```javascript
    function throwIfMissing() {
        throw new Error('Missing parameter');
    }
    
    function foo(mustBeProvided = throwIfMissing()) {
        return mustBeProvided;
    }
    
    foo()
    ```

*   rest参数

    rest参数使用…扩展运算符将多余参数放入数组中，形式为(a, …b),所以该函数可以传入任意数目的参数，该参数也可以使用所有数组特有的方法：

    ```javascript
    const sortNumbers = (...numbers) => numbers.sort();
    function push(array, ...items) {
    	items.forEach(function(item) {
    		array.push(item);
    		console.log(item);
    	});
    }
    var a = [];
    push(a, 1, 2, 3)
    ```

需要注意的是，rest参数之后不能再有其他的参数，函数的length属性也不适用于rest参数。


*   函数内部严格模式

    在ES5中，函数内部可以使用严格模式，在ES2016中则规定只要函数参数使用了**默认值**、**解构赋值**、**扩展运算符**，那么函数内部就不能显式设定严格模式（这是因为函数内部严格模式适用于函数体和函数参数，由于函数执行是先执行函数参数继而执行函数体，所以在函数参数执行的时候无法获知是否使用严格模式，导致报错）。

    如果非要在有默认值、解构赋值或扩展运算符在函数参数中出现并需要再函数内设定严格模式，那么可以使用：

    1.  全局性的严格模式
    2.  使用无参数的立即执行函数将所需函数包含进去，并在立即执行函数中使用严格模式
*   函数的name属性：返回该函数的函数名

    ```javascript
    var f = function () {};
    f.name 	// "f"
    const bar = function baz() {};
    bar.name 	// "baz"
    ```

上面代码中显示，函数表达式中如果赋值的是匿名函数，则函数名返回该变量的变量名，若赋值的是具名函数，则返回该具名函数的函数名。

需要注意两点：

1. Function构造函数返回的函数实例，name属性的值为anonymous，`(new Function).name // "anonymous"`
2. bind返回的函数，name属性值会加上bound前缀`(function(){}).bind({}).name // "bound "`

*   箭头函数：`var f = v => console.log(v);`

    箭头函数是定义函数的简单方法，相当于匿名函数，‘=>’符号左边是函数的参数（如果不需要参数或者需要多个参数，可以直接使用()代替参数），右边是函数体（多余一条语句的代码块需要用{}括起来，如果返回的是一个对象，则该对象需要使用()括起来）。

    ```javascript
    //将上面sort函数改进为可以按照大小排序的方法
    var result = values.sort((a, b) => a - b);
    ```

需要注意的是：

1\. 箭头函数体内this对象是定义时所在对象，而非使用时所在对象
2\. 箭头函数不可以当做构造函数，不可以使用new命令
3\. 不能使用arguments对象，但是可以使用rest参数代替
4\. 不能使用yield命令，因此箭头函数不能用作Generator函数

箭头函数还可以进行嵌套：

```javascript
const pipeline = (...funcs) => val => funcs.reduce((a, b) => b(a), val);
const plus1 = a => a + 1;
const mult2 = a => a * 2;
const addThenMult = pipeline(plus1, mult2);
addThenMult(5)
// 12
```

上述代码显示，箭头前的（输出）作为箭头后的输入

- 绑定this：该语法是ES7的一个提案，函数绑定运算符是并排的两个冒号::，左边为一个对象，右边为一个函数，功能是将函数绑定在左边对象的上下文环境中（当前this指向左边的对象）；由于绑定之后返回的仍然是对象，所以可以链式写法：

    ```javascript
    foo::bar;
    // 相当于
    bar.bind(foo);
    foo::bar(...arguments);
    // 等同于
    bar.apply(foo, arguments);
    ```	
	
- 尾调用优化：尾调用是指某个函数的最后一步是调用另一个函数`function f(x){return g(x);}`
- 尾递归优化：对于尾递归来说，只存在一个调用帧，永远不会发生“栈溢出”的错误，但是必须在严格模式下才可以生效：

    ```javascript
    //非尾递归：
    function Fibonacci (n) {
      if ( n <= 1 ) {return 1};
    
      return Fibonacci(n - 1) + Fibonacci(n - 2);
    }
    
    //尾递归：
    function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
      if( n <= 1 ) {return ac2};
    
      return Fibonacci2 (n - 1, ac2, ac1 + ac2);
    }
    ```