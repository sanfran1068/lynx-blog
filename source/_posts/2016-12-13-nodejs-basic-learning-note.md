layout: post
title: "Node-js基础学习笔记"
date: 2016-12-13 12:00:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories: 
- Document
tags:
- JavaScript
- Nodejs
- 读书笔记
- 基础知识
---

#### Node.js基础学习笔记

在前端的学习过程当中，不可避免地要学习到一定的后台服务器的技术。之前一直都沉浸在servlet中无法自拔，所以Java一度成为自己还比较熟悉的后台语言，但是相对于本身比较熟悉的JavaScript来说，当然是尽量用JS来处理后台最好不过了，所以Node.js顺其自然成为了学习路上必要的一道关卡。

略过安装、配置和创建第一个应用，想必很多Node.js相关网站都可以找到相应的教程，本文仅记录在学习Node.js基础知识中一些关键点技术。

<!-- more -->

##### Node.js应用

Node.js 应用是由下面几部分组成的：

*   引入 required 模块：我们可以使用 require 指令来载入 Node.js 模块。
*   创建服务器：服务器可以监听客户端的请求，类似于 Apache 、Nginx 等 HTTP 服务器。
*   接收请求与响应请求 服务器很容易创建，客户端可以使用浏览器或终端发送 HTTP 请求，服务器接收请求后返回响应数据。

*   创建一个服务器：

    ```javascript
    var http = require('http');
    http.createServer(function (request, response) {
        // 发送 HTTP 头部 
        // HTTP 状态值: 200 : OK
        // 内容类型: text/plain
        response.writeHead(200, {'Content-Type': 'text/plain'});
        // 发送响应数据 "Hello World"
        response.end('Hello World\n');
    }).listen(8888);
    // 终端打印如下信息
    console.log('Server running at http://127.0.0.1:8888/');
    ```

##### 回调函数

Node.js是单进程单线程应用程序，但是通过事件和回调支持并发和异步。

Node的所有API都支持回调函数，并作为一个独立线程运行，完美支持异步处理大量的请求。回调函数作为**最后一个参数**传入异步函数，而回调函数的第一个参数为**错误对象**：

```javascript
var fs = require("fs");
fs.readFile('input.txt', function (err, data) {
   if (err){
      console.log(err.stack);
      return;
   }
   console.log(data.toString());
});
console.log("程序执行完毕");
```

##### 事件循环

Node.js使用事件驱动模型，所有的事件机制都是用设计模式中观察者模式实现。代码中引入events模块，通过实例化EventEmitter类来绑定和监听事件：

```javascript
var events = require('events');                 // 引入 events 模块
var eventEmitter = new events.EventEmitter();   // 创建 eventEmitter 对象
event.on('some_event', function() {             // 绑定事件，function中可添加多参数
    console.log('some_event 事件触发'); 
}); 
setTimeout(function() {                         // 触发事件
    event.emit('some_event'); 
}, 1000);
```

events模块下只有一个对象：events.EventEmitter，其核心就是**事件触发与事件监听**两者的封装。EventEmitter的每个事件都由一个事件名（一个字符串）和若干参数构成。一般来说，不会直接使用EventEmitter，而是在对象中继承它，包括fs、net、http在内都是EventEmitter的子类（为了符合事件语义，并且符合JS的原型对象机制）。

*   EventEmitter对象的属性

    *   .addListener(event, listener)：为指定事件添加一个监听器到监听器数组的尾部
    *   .on(event, listener)：为指定事件注册一个监听器，接受一个字符串 event 和一个回调函数
    *   .once(event, listener):为指定事件注册一个单次监听器，即 监听器最多只会触发一次，触发后立刻解除该监听器
    *   .removeListener(event, listener)：移除指定事件的某个监听器，监听器必须是该事件已经注册过的监听器。
        它接受两个参数，第一个是事件名称，第二个是回调函数名称
    *   .removeAllListeners([event])：移除所有事件的所有监听器， 如果指定事件，则移除指定事件的所有监听器
    *   setMaxListeners(n)：默认情况下， EventEmitters 如果你添加的监听器超过 10 个就会输出警告信息；setMaxListeners 函数用于提高监听器的默认限制的数量
    *   listeners(event)：返回指定事件的监听器数组。
    *   emit(event, [arg1], [arg2], […])：按参数的顺序执行每个监听器，如果事件有注册监听返回 true，否则返回 false。

##### 缓冲区

由于JS中不支持二进制数据类型，所以在处理像TCP流或文件流时，Node.js提供了Buffer类，用来创建一个专门存放二进制数据的缓存区（一个Buffer类类似于一个整数数组）。创建Buffer实例有多种方法：

```
+ 以字节数量创建：var buf = new Buffer(10);
+ 以数组创建：var buf = new Buffer([10, 20, 30, 40, 50]);
+ 以字符串创建：var buf = new Buffer("www.runoob.com", "utf-8");
```

*   写入缓冲区：buf.write(string[, offset[, length]][, encoding]);参数分别为必须的写入缓冲区的字符串，可选的开始写入索引值（默认为0）、写入字节数（默认buffer.length）、使用的编码（默认为utf8）。该写入的方法会**返回实际写入的大小**
*   读缓冲区：buf.toString([encoding[, start[, end]]]);
*   将Buffer实例转换为JSON对象：buf.toJSON()
*   缓冲区合并：Buffer.concat(bufArray[, totalLength]);其中bufArray是需要进行合并的Buffer实例数组
*   缓冲区比较：buf.compare(otherBuffer);返回负数表示buf在前，0表示相同，正数表示buf在后
*   拷贝缓冲区：buf.copy(targetBuffer[, targetStart[, sourceStart[, sourceEnd]]]);其中buf是被复制的，targetBuffer是复制的目标缓冲区
*   缓冲区裁剪：buf.slice([start[, end]]);返回一个新的缓冲区，与旧缓冲区指向同一块内存，索引不同
*   缓冲区长度：buf.length;返回缓冲区所占内存长度

##### Stream

是一个抽象接口，流类型有四种：

```
+ Readable：可读操作
+ Writable：可写操作
+ Duplex：可读可写操作
+ Transform：操作被写入数据，然后读出结果
```

所有的Stream对象都是EventEmitter的实例，所以可以绑定的常用事件有data（有数据可读时触发）、end（没有更多数据可读时触发）、error（接收和写入过程发生错误时触发）、finish（所有数据写入到底层系统时触发）。

Node中又有很多对象实现了这个接口，例如对http服务器发起请求的request对象就是一个Stream，stdout（标准输出）也是一个Stream。

*   读取流

    ```javascript
    var fs = require("fs");
    var data = '';
    var readerStream = fs.createReadStream('input.txt');//创建可读流
    readerStream.setEncoding('UTF8');
    // 处理流事件 --> data, end, and error
    readerStream.on('data', function(chunk) {    //此处的data并不是读取结果字符串
       data += chunk;
    });
    readerStream.on('end',function(){
       console.log(data);
    });
    readerStream.on('error', function(err){
       console.log(err.stack);
    });
    ```
    
*   写入流

    ```javascript
    var fs = require("fs");
    var data = '菜鸟教程官网地址：www.runoob.com';
    var writerStream = fs.createWriteStream('output.txt');//创建写入流
    writerStream.write(data,'UTF8');
    writerStream.end();                                   // 标记文件末尾
    writerStream.on('finish', function() {
        console.log("写入完成。");
    });
    writerStream.on('error', function(err){
       console.log(err.stack);
    });
    ```
    