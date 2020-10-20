layout: post
title: "LESS官方文档学习笔记"
date: 2017-08-02 12:00:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-css%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories:
- Document
tags:
- JavaScript
- 基础知识
- 读书笔记
---

LESS是一种动态样式语言。
LESS将CSS赋予了动态语言的特性，如变量，继承，运算，函数。LESS既可以在客户端上运行 (支持IE 6+，Webkit， Firefox)，也可以借助Node.js或者Rhino在服务端运行。

<!-- more -->

### 使用方式

*   客户端使用

    在页面顶端加入下面代码，在服务器环境下可以使用less文件：

    ```html
    <link rel="stylesheet/less" type="text/css" href="styles.less">
    <script src="less.js" type="text/javascript"></script>
    ```

注意，LESS文件引入时rel的值为*stylesheet/less*；less.js一定要在LESS文件引用之后的位置。

*   在服务器端使用

    使用npm工具管理器在服务器端安装LESS，之后可以在Node中调用编译器，类似于：

    ```javascript
    var less = require('less');
    less.render('.class { width: 1 + 1 }', function (e, css) {
    	    console.log(css);
    });
    ```

    可以通过向解析器传递参数对LESS进行配置：

    ```javascript
    var parser = new(less.Parser)({
        paths: ['.', './lib'], // Specify search paths for @import directives
        filename: 'style.less' // Specify a filename, for better error messages
    });
    parser.parse('.class { width: 1 + 1 }', function (e, tree) {
        tree.toCSS({ compress: true }); // Minify CSS output
    });
    ```
    
    也可以通过命令行的方式调用LESS解析器对LESS文件进行解析或编译：

    ```javascript
    $ lessc styles.less
    $ lessc styles.less > styles.css
    ```

### 语法

#### 注释

完全支持CSS中的注释方法。

#### 变量

LESS中的变量是完全的“常量”，由@符号引导，且只能够定义**一次**：

```css
@nice-blue: #5B83AD;
@light-blue: @nice-blue + #111;
	
@fnord: "I am fnord.";
@var: 'fnord';
.text{
	color: light-blue;
	content: @@var;
}
```

#### [](#混合 "混合")混合

LESS中定义好通用的class属性集之后，可以在另一个属性集中调用：

```css
.bordered {
  border-top: dotted 1px black;
  border-bottom: solid 2px black;
}
.post a {
  color: red;
  .bordered;
}
```

**带参数混合**

可以像函数的参数一样为样式属性集合名称中添加带**默认值的参数**（参数可设置多个，使用逗号进行分割，可以使用@arguments代替所有参数出现在样式属性集合体之中），该混合方法最适合隐藏某属性集合但需要引用该属性集合时使用。

```css
/*带有参数的混合方法*/
.border-radius (@radius: 5px) {
  border-radius: @radius;
  -moz-border-radius: @radius;
  -webkit-border-radius: @radius;
}
#header {
  .border-radius(4px);
}
/*
  使用@arguments代替所有参数进行表达
  其中传参个数可以少于参数数量，但是参
  数的位置必须对应，中间参数位置不可空
*/
.box-shadow (@x: 0, @y: 0, @blur: 1px, @color: #000) {
  box-shadow: @arguments;
  -moz-box-shadow: @arguments;
  -webkit-box-shadow: @arguments;
}
.special-box-shadow {
	.box-shadow(2px, 5px);
}
```

#### [](#模式匹配和导引表达是 "模式匹配和导引表达是")模式匹配和导引表达是

*   模式匹配

    如果想根据传入的参数来改变混合的默认呈现，可以使用模式匹配方式（相当于将第一个参数设置为可变参数，该参数可以设置switch的选项来启动相应的模式）：

    ```css
    .mixin (dark, @color) {
      color: darken(@color, 10%);
    }
    .mixin (light, @color) {
      color: lighten(@color, 10%);
    }
    /*接受任意值，相当于默认值*/
    .mixin (@_, @color) {
      display: block;
    }
    @switch: light;
    	
    .someClass {
    	.mixin(@switch, #888);
    }
    ```

*   引导

    当我们想根据表达式进行匹配，而非根据值和参数匹配时，导引就显得非常有用。如果你对函数式编程非常熟悉，那么你很可能已经使用过导引。

    导引中可以用的比较运算符有：>、>=、=（这里相当于==）、=<、<
    
    ```css
    /*
      使用when语句判断条件，lightness()是一个
         判断颜色亮度的函数。
    */
    .mixin (@a) when (lightness(@a) >= 50%) {
        background-color: black;
    }
    .mixin (@a) when (lightness(@a) < 50%) {
        background-color: white;
    }
    .mixin (@a) {
        color: @a;
    }
    ```
    
    when后面所跟的条件，值为布尔true时才会执行解析，而LESS中的除去上面的**比较结果返回值**以及**关键词true**以外的值都视为布尔false：

    ```css
    /*下面两个语句含义相同*/
    .truth (@a) when (@a) { color: black; }
    .truth (@a) when (@a = true) { color: black; }
    /*下面一句是不会匹配任何条件，也不执行*/
    .someClass { .truth(255); }
    ```
    
    导引之后可以有**多个条件**，一般使用`(condition1),(condition2),...`的形式来表示。符合其中的任意一个条件，匹配就可以成功，属性集才会被执行，这个形式相当于使用关键词*or*来连接条件实现或门。另外，可以使用关键词*and*实现与门，使用*when not*关键词实现非门。
    
    导引语句when之后的判断条件中可以是**不含参数**的判断条件，也可以是对多个参数的比较等：
    
    ```css
    .mixin (@a) when (@media = mobile) { ... }
    .mixin (@a) when (@media = desktop) { ... }
    .max (@a, @b) when (@a > @b) { width: @a }
    .max (@a, @b) when (@a < @b) { width: @b }
    ```
    
    其中判断语句中可以使用一些判断函数，如常见的类型判断：`iscolor()`，`isnumber()`，`isstring()`，`iskeyword()`，`isurl()`；常见的单位量判断：`ispixel()`，`ispercentage()`，`isem()`。

#### 嵌套规则

此处的嵌套，指的是对上级重复的派生选择器进行嵌套方式的简写，例如：

```css
#header { color: black; }
#header .navigation {
  font-size: 12px;
}
#header .logo { 
  width: 300px; 
}
#header .logo:hover {
  text-decoration: none;
}
```

可以简写为：

```css
#header { 
	color: black; 
	.navigation {
  		font-size: 12px;
	}
	.logo { 
  		width: 300px; 
  		&:hover: text-decoration: none;
	}
}
```

这里需要注意的是，对于`:hover`和`:focus`这样的伪类等，需要使用串联选择器而非后代选择器时（串联选择器之间无空格，后代选择器有）应该使用_&_符号置于前面：

```css
.bordered {
  &.float {
    float: left; 
  }					/*输出的是.bordered.float选择器*/
  .top {
    margin: 5px; 
  }					/*输出的是.bordered .top选择器*/
}
```

#### 运算

数字、颜色或者变量都可以参与运算，且支持并向下兼容带有单位的数值：

```css
@base: 5%;
@filler: @base * 2;				/*返回10%*/
@other: @base + @filler;		/*返回15%*/
color: #888 / 4;
background-color: @base-color + #111;
height: 100% / 2 + @filler;
@var: 1px + 5;					/*输出结果是6px*/
width: (@var + 5) * 2;
```

#### [](#函数 "函数")函数

LESS中提供了非常丰富且方便使用的函数，包括有color函数、字符串函数、数学函数等等，下面主要介绍一下color函数，其余的函数可以在官方的API中查询使用[^1](http://less.bootcss.com/functions/#functions-overview)

##### [](#Color函数 "Color函数")Color函数

LESS提供的颜色运算函数，本质是先把RGB格式转化成HSL（就是色调、饱和度和亮度三维空间）色彩空间，然后再通道级别进行操作：

```css
hue(@color);        		/* 返回该颜色的hue通道值 */
saturation(@color);  		/* 返回该颜色的饱和度值 */
lightness(@color);  		/* 返回颜色的亮度值 */

lighten(@color, 10%);     /* 颜色亮度提高10% */
darken(@color, 10%);      /* 颜色亮度降低10% */

saturate(@color, 10%);    /* 颜色饱和度提高10% */
desaturate(@color, 10%);  /* 颜色饱和度降低10% */

fadein(@color, 10%);      /* 颜色透明度降低10% */
fadeout(@color, 10%);     /* 颜色透明度提高10% */
fade(@color, 50%);        /* 颜色透明度变为50% */

spin(@color, 10);         /* 在色轮上顺时针旋转10°
spin(@color, -10);        /* 在色轮上逆时针旋转10°

mix(@color1, @color2);    /* 两种颜色的混色
```

#### [](#命名空间 "命名空间")命名空间

为了对CSS进行更好的封装，可以将一些变量或混合模式打包起来，如：

```css
#bundle {
  .button () {
    display: block;
    border: 1px solid black;
    background-color: grey;
    &:hover { background-color: white }
  }
  .tab { ... }
  .citation { ... }
}
/* 使用>符号来进行引入 */
#header a {
  color: orange;
  #bundle > .button;
}
```

#### 作用域

LESS中的作用域类似于JS中原型链，首先会从本地查找变量或者混合模块，如果没找到的话会去父级作用域中查找，直到找到为止.

#### importing

可以通过@import “less-file-name.less”，其中.less的后缀是可选的。

#### 字符串插值

变量可以通过使用@{someName}的方式来将变量放入字符串中，例如：

```css
@base-url: "http://assets.fnord.com";
background-image: url("@{base-url}/images/bg.png");
```

#### 避免编译

当我们需要让一些语句失去作用，避免编译这些语句时，可以在语句字符串前面加上~，例如：

```css
.class {
  filter: ~"ms:alwaysHasItsOwnSyntax.For.Stuff()";
}
```

#### 在LESS中使用JavaScript

通过反单引号``的形式包围：

```css
@str: "hello";
@var: ~`"@{str}".toUpperCase() + '!'`;

```