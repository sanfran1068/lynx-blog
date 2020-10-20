layout: post
title: "从输入URL到页面加载发生了什么之进阶篇"
date: 2018-03-22 12:00:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories: 
- Reading
tags:
- JavaScript
- 基础知识
- 读书笔记
- HTTP
---

文章来源：https://segmentfault.com/a/1190000006879700
文章优点：采用总分总的形式进行撰写，结构和章节明确，发散较广。

#### 文章首先指出整个过程分为六大步骤：
- DNS解析
- TCP连接
- 发送HTTP请求
- 服务器处理请求并返回HTTP报文
- 浏览器解析渲染页面
- 连接结束

#### DNS解析：
将网站的【姓名】翻译成【身份证号】，是一个翻译的过程。
这个过程是将网址从右到左进行解析和查询的过程。.=>com=>google.com
为了查询效率，DNS解析简历起多级缓存（浏览器缓存，系统缓存，路由器缓存，IPS服务器缓存，根域名服务器缓存，顶级域名服务器缓存，主域名服务器缓存）
为了均衡地发送和处理请求，DNS进行负载均衡（DNS重定向），CDN也是利用了该技术，根据每台机器负载量、地理位置距离等返回给用户最优的一个IP地址
没有使用CDN时请求服务器资源：localDNS-rootDNS-逐级查询到域名授权的DNS记录-使用查询到的ip去请求内容
使用了CDN之后：localDNS-rootDNS-逐级查询到域名授权的DNS记录-向DNS服务器请求最优ip地址-使用该ip去CDN节点服务器返回对应的资源

![DNS解析流程图](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiNmMwYzU1MDItZTBkNy00MmUxLWJmNzctZTZiZmQzN2RhMDI0IiwicmVzb3VyY0d1aWQiOiIwN2JkOWFiOS0zZTYyLTQzNTMtYmQ3YS0yNWE2ZjA4ZDk0MGEifQ==)

![DNS解析流程图](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiNmMwYzU1MDItZTBkNy00MmUxLWJmNzctZTZiZmQzN2RhMDI0IiwicmVzb3VyY0d1aWQiOiI1ZWRmZWExMC0xOTAwLTQzMmYtYTUwZS1lYmVkZjI4YzIxY2QifQ==)

#### TCP连接
网站请求大部分使用http请求，http协议是使用tcp作为其传输层协议，（应用http、传输tcp、网络ip、数据、物理），http由tcp报文中提取。
由于http报文为明文存在一定风险，所以在http报文包裹入tcp报文之前使用ssl对其进行加密。
https在原有tcp握手的基础上需要提前进行一个解密握手。https会有时间损耗，需要在安全和性能两者做出权衡。（公钥加密报文，私钥加密公钥）


#### 发送HTTP请求
请求报文：请求行（请求方法、内容、协议），请求报头（Accept、Content-Type、Cookie、User-Agent）、请求正文（POST/PUT时会有）
响应报文：状态码、响应报头、相应报文
状态码：
1xx：指示信息–表示请求已接收，继续处理。
2xx：成功–表示请求已被成功接收、理解、接受。
3xx：重定向–要完成请求必须进行更进一步的操作。4xx：客户端错误–请求有语法错误或请求无法实现。5xx：服务器端错误–服务器未能实现合法的请求。

#### 浏览器解析渲染页面
文档来源：https://www.html5rocks.com/en/tutorials/internals/howbrowserswork
主要文章：https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#The_rendering_engine
总流程：
![浏览器解析渲染页面总流程](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiNmMwYzU1MDItZTBkNy00MmUxLWJmNzctZTZiZmQzN2RhMDI0IiwicmVzb3VyY0d1aWQiOiI3Njc1ZTE1ZS1hZjZhLTRlMmItYmZmNC05OTU3ZGIzM2YyNzYifQ==)

没有脚本的流程：
![没有脚本的流程](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiNmMwYzU1MDItZTBkNy00MmUxLWJmNzctZTZiZmQzN2RhMDI0IiwicmVzb3VyY0d1aWQiOiI4MTNmMmU1Yi05M2I5LTQ5NTItYWIzYy0yZmE1NzA2ZjQ5YmMifQ==)

HTML Parsing: 输出DOM tree
HTML不是一种context free grammar，是因为它不同于XML，它允许一些类似标签缺失等的容错
分为两部分：tokenize和tree construction
Tokenize：状态机实现，初始状态为DATA，遇到<状态改为TAGOPEN，之后遇到第一个a-z的字符状态改为TAGNAME，之后遇到>状态改为DATA，直到</>，tokenize就commit一个token，
Tree Construction：状态机实现。Document Object在token的parser创建时同时创建，每当一个token被commited，constroctor会判断该token跟哪个dom元素相关，新建该token的元素，并添加到开放元素栈（这个栈用于检查标签闭合以及未正常闭合标签）中：
![DOM tree 生成](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiNmMwYzU1MDItZTBkNy00MmUxLWJmNzctZTZiZmQzN2RhMDI0IiwicmVzb3VyY0d1aWQiOiI1ZWEwYjRlMi03ZTg1LTRjNzAtOGNhMi0yMjc2NGExZWE2NGYifQ==)

css和js的parse过程：
Firefox blocks all scripts when there is a style sheet that is still being loaded and parsed. 
WebKit blocks scripts only when they try to access certain style properties that may be affected by unloaded style sheets.
文章的优点：
从dom tree的构建的词法分析层面指出一些标签错误以及兼容方式，从语法分析层面指出了类似<table><table></table></table>这种情况应当怎样处理。
render tree构建过程中，构建方法对head标签以及display:none的标签不予插入树结构中，而hidden则会存在在结构当中。

Web优化
雅虎35条军规：https://juejin.im/post/5b73ef38f265da281e048e51
![35条军规](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiNmMwYzU1MDItZTBkNy00MmUxLWJmNzctZTZiZmQzN2RhMDI0IiwicmVzb3VyY0d1aWQiOiJhODJiMjU2NS1kNTFiLTQzYjMtOTZkYS0wNzIwNmE3MTRlYjYifQ==)
