layout: post
title: "前端要学习的http基础知识"
date: 2017-09-21 12:00:00
comments: true
categories: 
- Document
tags:
- JavaScript
- 基础知识
- Http
---

#### HTTP协议基础

##### HTTP

Hyper Text Transfer Protocol(超文本传输协议)，基于TCP/IP通信协议来传递数据，默认端口号80。HTTP协议是前端技术基础中的基础，所以掌握其相关知识也是必要技能之一。

<!-- more -->

##### HTTP工作原理

工作于C/S架构上，浏览器作为http客户端通过URL向WEB服务器（Apache、IIS）发送所有请求

##### HTTP的特点

- 无连接：每次连接只处理一个请求（节省传输时间）
- 媒体独立：任何类型内容数据都可通过HTTP发送（制定MIME-type内容类型）
- 无状态：对事务没有记忆能力，若要在当前状态进行后续处理，则必须重传（导致每次连接传送的数据量增大，但是应答比较快）

##### HTTP请求与返回

- HTTP请求：

    ```javascript
    GET / HTTP/1.0
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5)
    Accept: */*
    ```
    
    其中第一行为请求命令，命令有POST/GET/PUT/DELETE等等，且请求的资源后面要添加所使用的协议版本，详见附录；第三行中的Accept指定的是可以接受的MIME type，与回应头中的Content-Type字段相对应。
    
- HTTP回应：

    ```javascript
    HTTP/1.0 200 OK 
    Content-Type: text/plain
    Content-Length: 137582
    Expires: Thu, 05 Dec 1997 16:00:00 GMT
    Last-Modified: Wed, 5 August 1996 15:55:28 GMT
    Server: Apache 0.84
    
    <html>
      <body>Hello World</body>
    </html>
    ```
    
    回应信息分为两部分，一部分是回应头信息，一部分是回应的数据和内容。其中第一行是"协议版本 + 状态码（status code） + 状态描述"；第二行是用来指定数据格式的"Content-Type字段"。
    
#### 各版本HTTP协议的区别

- HTTP/0.9

    只有GET一个请求命令，而且服务器只能回应HTML格式的字符串，服务器发送完毕，TCP连接就关闭。

- HTTP/1.0

    完善了POST和HEAD命令，服务器可以回应更多种格式的（MIME type/多种压缩方法）数据，但是需要在回应中有一个header来描述元数据。另外添加了状态吗、权限、缓存等等。
    
    但是与0.9版本相同的是，每个TCP连接只能发送一个请求，发送完成就关闭连接。所以有一个hack方法：Connection字段设置为keep-alive来实现保持连接和请求复用。

- HTTP/1.1

    添加了PUT/PATCH/HEAD/OPTIONS/DELETE请求命令，以及HOST字段指定服务器的域名（该字段可以将请求发往同一台服务器的不同网站，是虚拟主机实现的基础）；

    最大的改动是**默认服务器连接为keep-alive**，不需要声明Connection字段，但是如果主动关闭连接需要添加Connection: close的字段；添加**管道机制**，即在同一个TCP连接里面，客户端可以同时发送多个请求；在管道机制之上需要有区分数据包的机制，该版本提供了Content-Length字段；有了管道机制，就可以实现**分块传输编码**（不使用Content-Length字段的方法），只需要设置Transfer-Encoding字段为chunked即可，每一个返回的内容数据块之前苏姚添加该数据块的长度，最后一个长度需要表示为0来标识回应数据结束。

    1.1版本中唯一的缺点是数据通信单线程按次序进行，非常容易堵塞，所以要求网页设计时减少请求书并多开持久连接；
    
- HTTP/2.0

    该版本主要有四个亮点：
    
    - 二进制协议：头信息和数据块都可以是二进制（被称为头信息帧和数据帧），更加容易解析；
    - 多工通信：在同一个连接里，S/C两端可以同时发送多个请求和回应，避免单线程堵塞；这个技术也为服务器推送奠定了基础；
    - 数据流： 在多工通信的前提下，请求和回应数据必须进行标记（ID），所以提出了数据流概念，奇数表明客户端发出数据流，偶数则为服务器发出；另外客户端还可以指定请求数据流的优先级；
    - 头信息压缩：针对Cookie、User Agent字段的重复和冗余，引入头信息压缩机制，可以使用gzip和compress压缩后再发送，提高速度。
    
#### 附录 

##### HTTP header字段（通用头，请求头，响应头，实体头）

- 通用头：Cache-Control/Connection/Date/Pragma/Transfer-Encoding/Upgrade/Via

- 请求头：

| 字段名 | 含义 | 示例 |
| --------------- | ---------------- | -----------------|
|Accept|		指定客户端能够接收的内容类型||	
|Accept-Charset|	浏览器可以接受的字符编码集	||
|Accept-Encoding|	指定浏览器可以支持的web服务器返回内容压缩编码类型||
|Accept-Language|	浏览器可接受的语言	||
|Accept-Ranges|	可以请求网页实体的一个或者多个子范围字段	||
|Authorization|	HTTP授权的授权证书	||	
|Cookie|		HTTP请求发送时，会把保存该请求域名下所有cookie发给服务器||
|Content-Length|	请求的内容长度|	Content-Length: 348|
|Content-Type|	请求的与实体对应的MIME信息	||
|Expect|		请求的特定的服务器行为|	Expect: 100-continue       |         
|From	|		发出请求的用户的Email|	From: user@email.com|
|Host|			指定请求的服务器的域名和端口号|	Host: www.zcmhi.com|
|If-Match|		只有请求内容与实体相匹配才有效	||
|If-Modified-Since|	若请求部分在指定时间后被修改则请求成功，未被修改则返回304代码||
|If-None-Match|	如果内容未改变返回304代码||
|If-Range	|	如果实体未改变，服务器发送客户端丢失部分，否则发送整个实体||
|If-Unmodified-Since|	只在实体在指定时间之后未被修改才请求成功||
|Max-Forwards|	限制信息通过代理和网关传送的时间	|Max-Forwards: 10|
|Proxy-Authorization	|连接到代理的授权证书	||
|Range	|	只请求实体的一部分，指定范围	|Range: bytes=500-999|
|Referer	|	先前网页的地址，当前请求网页紧随其后,即来路||
|User-Agent|		包含发出请求的用户信息|	User-Agent: Mozilla/5.0 (Linux; X11)|
|Warning	|	关于消息实体的警告信息| |

- 响应头：

| 字段名 | 含义 | 示例 |
| --------------- | ---------------- | -----------------|
|Accept-Ranges|	表明服务器是否支持指定范围请求及哪种类型的分段请求||
|Age		|	从原始服务器到代理缓存形成的估算时间（以秒计，非负）	|Age: 12|
|Allow	|		对某网络资源的有效的请求行为POST/GET等，不允许则返回405||
|Content-Encoding|	web服务器支持的返回内容压缩编码类型|	Content-Encoding: gzip|
|Content-Language|	响应体的语言|	Content-Language: en,zh|
|Content-Length|	响应体的长度|	Content-Length: 348|
|Content-Location|	请求资源可替代的备用的另一地址|	Content-Location: /index.htm|
|Content-MD5|	返回资源的MD5校验值||	
|Content-Range|	在整个返回体中本部分的字节位置	||
|Content-Type|	返回内容的MIME类型|	Content-Type: text/html; charset=utf-8|
|ETag	|		请求变量的实体标签的当前值||
|Expires|		响应过期的日期和时间|Expires: Thu, 01 Dec 2010 16:00:00 GMT|
|Last-Modified|	请求资源的最后修改时间	||
|Location|		用来重定向接收方到非请求URL的位置来完成请求或标识新的资源	|| 
|Proxy-Authenticate	|它指出认证方案和可应用到代理的该URL上的参数||
|Refresh	|	应用于重定向或一个新的资源被创造，在5秒之后重定向||
|Retry-After|	如果实体暂时不可取，通知客户端在指定时间之后再次尝试	|Retry-After: 120|
|Server|	web服务器软件名称|	Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)|
|Set-Cookie|	设置Http Cookie	||
|Trailer|	指出头域在分块传输编码的尾部存在|	Trailer: Max-Forwards|
|Vary|	告诉下游代理是使用缓存响应还是从原始服务器请求	|Vary: *|
|Warning|	警告实体可能存在的问题	|Warning: 199 Miscellaneous warning|
|WWW-Authenticate|	表明客户端请求实体应该使用的授权方案| |

##### HTTP请求方法

GET（请求制定的页面信息，并返回实体主体）
HEAD（与GET相似，只用于获取报头）
POST（向制定资源提交数据进行处理请求，表单或者上传文件等，会导致新资源建立或资源修改）
PUT（从客户端向服务器传送的数据取代制定的文档内容）
CONNECT（HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器）
OPTIONS（允许客户端查看服务器的性能）
TRACE（回显服务器收到的请求，主要用于测试诊断）
DELETE（删除删除服务器上的某资源）

##### 常见的MIME type

text/plain
text/html
text/css
image/jpeg
image/png
image/svg+xml
audio/mp4
video/mp4
application/javascript
application/pdf
application/zip
application/atom+xml
