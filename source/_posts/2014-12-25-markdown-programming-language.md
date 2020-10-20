title: Markdown
date: 2014-12-25 00:31:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/lynx.jpg -->
categories:
- Document
tags:
- Markdown
---
本篇文章是关于本站博客文章的Markdown格式的文档。
<!-- more -->

* [段落元素](#block-elements)
  * [p和br](#paragraphs-and-line-breaks)
  * [标题](#headers)
  * [段落引用](#blockquotes)
  * [列表](#lists)
  * [代码](#code-blocks)
  * [Horizontal Rules](#horizontal-rules)
  * [表格](#table)
* [段内元素](#span-elements)
  * [链接](#links)
  * [强调](#emphasis)
  * [代码](#code)
  * [图片](#images)
* [混杂模式](#miscellaneous)
  * [自动链接](#automatic-links)
  * [Backslash Escapes](#backslash-escapes)
* [内联HTML](#inline-html)

<!-- more -->
## Block Elements
### Paragraphs and Line Breaks
#### 段落
HTML标签: `<p>`

在一段后进行回车空出一行可以达到该效果。

代码:

	This will be
	inline.
	
	This is second paragraph.

预览:
***
This will be
inline.

This is second paragraph.
***

### Headers
Markdown支持两种方式的标题：Setext/atx.
#### Setext
HTML标签: `<h1>`, `<h2>`

使用等号‘=’在标题下划线代表`<h1>`， 使用连字符‘-’在标题下划线表示 `<h2>`，其中符号的数量可以不做限制.

代码:

    This is an H1
    =============
    This is an H2
    -------------
预览:
***
This is an H1
=============

This is an H2
-------------
***
#### atx
HTML标签: `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`

使用1到6个‘#’号来表示标题标签 `<h1>` - `<h6>`.

代码:

    # This is an H1
    ## This is an H2
    ###### This is an H6
预览:
***
# This is an H1
## This is an H2
###### This is an H6
***

atx格式的标题可以进行闭合如`### This is an H3 ######`，可见两端的‘#’号数量不需要相等。

### Blockquotes
HTML标签: `<blockquote>`

需要再每一个段落之前加‘>’符号来形成段落引用。

Code:

    > This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
    > > This is nested blockquote.
    > Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
    >
    > id sem consectetuer libero luctus adipiscing.
Preview:
***
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> >  This is nested blockquote.

> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
>
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
***


### Lists
Markdown支持有序和无序列表。
#### 无序列表
HTML标签: `<ul>`

无序列表可以使用‘*’、‘+’、‘-’三种符号开头并在符号后添加一个空格。

代码:

    *   Red
    +   Green
    —   Blue
预览:
***
*   Red
+   Green
-   Blue

#### Ordered
HTML标签: `<ol>`

有序列表通过数字和点符号‘.’以及空格开头来表示，前面的数字是几无所谓，会自动解析成从小到大的顺序。

代码:

    1.  Bird
    22.  McHale
    3.  Parish
    44\. Cat
    
预览:
***
1.  Bird
22.  McHale
3.  Parish
44\.  Cat
***

#### 缩进
段落级别的缩进使用tab添加在段落之前即可。

### Code Blocks
HTML标签: `<pre>`

代码的每一行都缩进至少‘4个空格’和‘1个tab’，代码结束是以第一个不缩进的段落为止。

代码:

    This is a normal paragraph:

        This is a code block.
        
代码:
***
This is a normal paragraph:

    This is a code block.
***

#### 包裹代码
可以使用`*3后跟语法名称来进行代码的包裹，从而避免每一行的缩进并可以进行语法高亮。

代码:

    Here's an example:

    ```javascript
    function test() {
      console.log("notice the blank line before this function?");
    }
    ```
预览:
***

```
function test() {
  console.log("notice the blank line before this function?");
}
```
***

### Horizontal Rules
HTML Tag: `<hr />`
分界线可以使用’*‘、’-‘、’_’

代码:

    * * *
    ***
    *****
    - - -
    ---------------------------------------
    ___
预览:
***
* * *
***
*****
- - -
---------------------------------------
___
***
### Table
HTML标签: `<table>`

表格使用‘|’、‘:’、‘-’三种符号进行构造，其中‘-’符号必须有至少3个才能构造表格。
代码:

```
| Left | Center | Right |
|:-----|:------:|------:|
|aaa   |bbb     |ccc    |
|ddd   |eee     |fff    |

A |B
--|--
12|45
```
预览:
***
| Left | Center | Right |
|:-----|:------:|------:|
|aaa   |bbb     |ccc    |
|ddd   |eee     |fff    |

A |B
--|--
12|45
***
## Span Elements
### Links
HTML标签: `<a>`

Markdown支持两种类型的链接：内联在段落中和引用模式.

#### 内联

内联在段落中`[Link Text](URL "Title")`，如果是在本站的网址，可以使用相对地址进行访问。

代码:

    This is [an example](http://example.com/ "Title") inline link.

    See my [About](/about/) page for details.
    
预览:
***
This is [an example](http://example.com/ "Title") inline link.

See my [About](/about/) page for details.
***

#### 参考引用
参考链接的形式如此：`[id]: URL "Title"`

代码:

    [id]: http://example.com/  "Optional Title Here"
    This is [an example][id] reference-style link.
预览:
***
[id]: http://example.com/  "Optional Title Here"
This is [an example][id] reference-style link.
***

### Emphasis
HTML标签: `<em>`, `<strong>`

Markdown使用‘\*’、‘_’符号进行斜体的标注，‘\*\*’、‘__’符号进行粗体的标注，如果直接使用‘*’符号可以加‘\’进行转义。

代码:

    *single asterisks*
    _single underscores_
    **double asterisks**
    __double underscores__
    
预览:
***
*single asterisks*

_single underscores_

**double asterisks**

__double underscores__

### Code
HTML标签: `<code>`

使用‘\`’来对内联的代码进行包裹，**backtick quotes (`)**.

Code:

    Use the `printf()` function.
Preview:
***
Use the `printf()` function.
***

### Images
HTML标签: `<img />`

Markdown支持内联和参考引用两种方式进行图片的添加。

#### 内联

内联图片代码如此： `![Alt text](URL "Title")`，其中title是可选的一项。

代码:

    ![Alt text](/path/to/img.jpg)

    ![Alt text](/path/to/img.jpg "Optional title")

#### Reference
参考引用方式的代码如此： `![Alt text][id]`

代码:

    [img id]: url/to/image  "Optional title attribute"
    ![Alt text][img id]

## Miscellaneous
### Automatic Links
使用‘< >’符号将链接包裹形成自动链接。

代码:

    <http://example.com/>

    <address@example.com>
预览:
***
<http://example.com/>

<address@example.com>
***

### Backslash Escapes
使用反斜线来进行某些符号的转义

代码:

    \*literal asterisks\*
预览:
***
\*literal asterisks\*
***
支持以下符号:

Code:

    \   backslash
    `   backtick
    *   asterisk
    _   underscore
    {}  curly braces
    []  square brackets
    ()  parentheses
    #   hash mark
    +   plus sign
    -   minus sign (hyphen)
    .   dot
    !   exclamation mark

## Inline HTML
如果需要一些HTML中的标签而Markdown中没有的，可以直接使用HTML标签进行包裹使用。

代码:

    This is a regular paragraph.

    <table>
        <tr>
            <td>Foo</td>
        </tr>
    </table>

    This is another regular paragraph.
预览:
***
This is a regular paragraph.

<table>
    <tr>
        <td>Foo</td>
    </tr>
</table>

This is another regular paragraph.
***
