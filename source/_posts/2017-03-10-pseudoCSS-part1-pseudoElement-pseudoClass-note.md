layout: post
title: "CSS中的伪类与伪元素"
date: 2017-03-10 12:00:00
comments: true
categories: 
- Document
tags:
- CSS3
- Selector
- 读书笔记
- 基础知识
---

#### 简介[^1]

样式表中的选择器是控制网页样式输出的非常重要的工具之一，也是css通过DOM树来控制其样式的最好的方式之一。而除了标签选择器、类选择器、ID选择器、属性选择器以及元素关系选择器（父子、兄弟等选择器），CSS3为我们提供了一种更为复杂同时也是更为方便设计者的选择方式-**伪类**（Pseudo-Classes）和**伪元素**（Pseudo-Elements）。

伪类和伪元素通常由**冒号**（colon）引导，而CSS3重新规定伪元素之前需要由两个冒号引导，但是为了使旧版本网站也能够统一使用和显示（IE8及以下浏览器无法理解双冒号的语法），浏览器则仍支持单冒号引导的伪元素方式（*除了少数特定的伪元素之外*）。

<!-- more -->
#### 伪类和伪元素的区别[^2]

CSS中的伪类根据元素的编码或动态状态等来选择HTML文档中特定部分，即选择HTML文档中的某元素（通过用户的干预或者随着时间的变化而产生动态改变状态），CSS2又将该特性扩展到了DOM树的某些“虚拟组件”和可推断部分（例如first-child），可以出现在选择器链条中的任何位置；

CSS中的伪元素是用来强调某些元素的子部分，允许选择并改变那些除DOM中指定的元素之外的部分的样式（对DOM树中未包含的逻辑元素进行选择和改变样式）(可以用来展示比如头字母大写等常见的排版效果)；伪元素有一些使用的限制条件：
- 不允许出现在<body>中的in-line样式（只能出现在外部样式表或位于<head><style></style></head>的样式中）；
- 在选择器的链条中只能出现在最后一位（即要位于任何类和ID选择之后）；
- 每个选择器中只能出现一个伪元素（若要在单个元素结构中城县多个伪元素的选择，必须使用多个选择器/样式声明）；

### 伪类

伪类就像是class一样，以冒号引导（e.g. :hover等），可以以**标签:伪类**和**类:伪类**的形式存在于CSS样式表中。

#### 动态伪类

##### 链接样式的动态伪类

针对链接，用户可以无任何操作、将光标移至链接上、点击、点击后放开光标，将光标从链接上移走等操作。

- a:link
  选择用户还未进行过任何操作的链接；

- a:visited
  选择用户已经点击过的链接；

- a:hover
  选择用户将鼠标移至上方的链接；同时，:hover伪类也可以用在除链接以外的元素上（e.g. <p>或是<div>）；

- a:active
  选择用户使用鼠标正在点击（按下）的链接；

样例代码：
```CSS
.example a:link {color:gray;} /*链接没被访问时为灰色*/
.example a:visited {color:maroon;} /*链接被访问过后前景色为红色*/
.example a:hover {color:blue;} /*鼠标悬浮在链接上时前景色为蓝色*/
.example a:active {color:darkblue;} /*鼠标按下链接前景色为深蓝色*/
```

这里需要特别注意的是，这四个动态伪类在样式表中定义的顺序是不可以任意排序的，而是要遵循唯一的一个顺序，即**L**O**V**E/**HA**TE原则，即以a:link/a:visited/a:hover/a:active的顺序来定义。这是因为在CSS样式表中针对同一个元素的样式定义要*从一般到特殊*。一个链接在用户未进行操作的时候可能同时拥有:link、:visited和:hover三种样式属性，而无论如何:hover都会覆盖前面两种样式，所以:hover要在前面两种样式之后定义，而:link又是最一般的样式，所以要放在第一位来进行定义；最后:active则能覆盖所有之前三种属样式，所以放在最后一位进行定义。一句话来总结这一特性，*越容易被覆盖的样式越先定义，覆盖越多的样式要越晚定义*。

##### 表单样式的动态伪类

常对于输入类的表单进行选择并展示或修改其样式的动态伪类；

- :focus
  与链接中的:hover的作用类似，是当用户将光标的注意力集中到某元素而改变其样式，多用于用户在网页上的表单进行输入的元素进行选择（e.g. \<input>或\<textarea>）。

- UI元素状态伪类(IE8及以下不支持该伪类)：
  - :enabled
    选择可以进行输入和选择等操作的表单元素进行样式的声明；

  - :disabled
    选择不能进行输入和选择等操作的表单元素进行样式的声明；

  - :checked
    选择已经选择的表单元素（type为radio或checkbox元素）进行样式声明；

  - :unchecked
    选择还未进行选择的表单元素进行样式声明；

#### 子选择器（:nth选择器）

CSS原本提供了后代选择器（descendant selector），但是针对于某些特定的场景，例如列表和表格制定行列的样式改变，CSS又提供了伪类选择器来为这些子类选择器提供服务。

- :first-child
  选择指定元素中第一个子元素；
- :last-child
  选择指定元素中最后一个子元素；
- :nth-child(parameter)
  选择某个元素的一个或多个特定的子元素，参数可以是自然数，也可以是像2n，3n+1，-n+3这样的公式进行有规律的子元素选择；
- :nth-last-child()
  与:nth-child()作用相同，但是是从选定元素的最后一个子元素开始算起；
- :nth-of-type()
  与:nth-child()作用相同，但是可以选择指定类别的子元素；
- :nth-last-of-type()
  与:nth-nth-of-type()作用相同，但是是从选定元素的最后一个子元素开始算起；
- :first-of-type
  与:first-child作用相同，只是可以选择子元素的类型；
- :last-of-type
  与:last-child作用相同，只是可以选择子元素的类型；
- :only-child
  当指定元素只有一个子元素时，选择该子元素；
- :only-of-type
  当指定元素只有一个指定类型的子元素时，选择该子元素；
- :empty
  当制定元素中没有任何内容时；


#### 特殊伪类选择器

- :target
  该伪类类似于HTML中的锚，指向文档中某个具体的元素，常用于点击出现某些指定元素样式的改变（点击按钮，某元素由display:none变为display:block;这样的出现效果）；

- :not
  也被称作negation pseudo-class，也就是否定取反。当使用类似于
  ```css
  p:not(.exampleStyle){color:green}
  ```
  时，意为选择所有不包含。exampleStyle类的p元素进行选择改变字体颜色为green。

### 伪元素

#### ::before和::after

- ::before
  可以在某给定元素上添加提前展示的内容：
  ```css
  .tip::before {content: "HOT TIP!" }
  ```
- ::after
可以在某给定元素上添加提前展示的内容：
  ```css
  .tip::after {content: "HOT TIP!" }
  ```

#### 文字排版常见的::first-line和::first-letter

- ::first-line
  选择制定元素的第一行（例如改变每个段落的第一行文本样式）；

- ::first-letter
  选择制定文本块的第一个字母进行样式设置，多用于进行段落排版；

#### 光标选中文本样式::selection

::selection用来改变浏览网页选中文本的默认效果；

[^1]: 《CSS, The Missing Manual, 4th Edition》
[^2]: [Difference between a pseudo-class and a pseudo-element]([http://www.d.umn.edu/~lcarlson/csswork/selectors/pseudo_dif.html)
