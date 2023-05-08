layout: post
title: "理解 JavaScript（Node）中的 Cookie 和 Session"
date: 2021-05-03 22:00:00
comments: true
categories: 
- Translation
tags:
- JavaScript
- NodeJS
- 基础知识
- 浏览器存储
---

问题一：为什么网站（网页应用）需要有登录的功能？
问题二：Cookie 到底是个什么东西？
问题三：Cookie 是怎样告诉服务器”是谁发送的请求“？
问题四：Cookie 和 Session 验证之间有什么不同？
问题五：Session 又是什么？

<!-- more -->

### 登录/登出功能在网站中的必要性

- 登录/登出的验证对于开发者来说是最难搞的功能之一
- 但是为了让网站更可靠，登录/登出功能又是必须的

**客户端请求有几个非常严肃的问题**：

- 客户端是不知道到底是谁触发的这个请求
- 当然，我们有 IP 地址和浏览器的一些信息
- 但是很有可能，不同的用户在不同的电脑上使用的是同一个 IP 地址
- 所以我们需要登录功能来告诉服务器究竟是谁发送的这个请求，Cookie 和 Session 验证就是解决这个问题的方法之二

### Cookie 的必要性

- 如果一个网站在刷新页面的时候不会使用户登出，那么这个网站一定是使用了 cookie 或 session
- 几乎每一个你所浏览的网站都使用了 cookie 和session
- 客户端之所以能够发送一些关于你和你的验证信息、状态到服务器，就是用了 cookie
- Cookie 就是非常简单的**键值对**： `name=gplee`
- 浏览器将 cookie 保存在内部，并在每一次发送请求的时候把这些 cookie 也一并发送到服务端
    
    ![%5B%E7%BF%BB%E8%AF%91%5D%E7%90%86%E8%A7%A3%20JavaScript%EF%BC%88Node%EF%BC%89%E4%B8%AD%E7%9A%84%20Cookie%20%E5%92%8C%20Session%20821cab22b670445cacf2816c132b0710/0_ZIkc8UzGscPbkxFg.png](%5B%E7%BF%BB%E8%AF%91%5D%E7%90%86%E8%A7%A3%20JavaScript%EF%BC%88Node%EF%BC%89%E4%B8%AD%E7%9A%84%20Cookie%20%E5%92%8C%20Session%20821cab22b670445cacf2816c132b0710/0_ZIkc8UzGscPbkxFg.png)
    

- 换句话说，cookie 能够记录和追踪你和关于你的信息
- 这也是为什么你会经常被建议去阶段性地删除 cookie（因为要防止个人信息的泄露）
- Cookie 是在 request 和 response 的 header 中进行传输的

### 让我们创建 cookie 并把它发送给浏览器吧

```jsx
const http = require('http');

const parseCookies = (cookie = '') => cookie.split(';')
	.map(v => v.split('='))
	.map(([k, ...vs]) => [k, vs.join('=')])
	.reduce((acc, [k, v]) => {
	    acc[k.trim()] = decodeURIComponent(v);
	    return acc;
	}, {});

http.createServer((req, res) => {
  const cookies = parseCookies(req.headers.cookie);
  console.log(req.url, cookies);
  res.writeHead(200, {'Set-Cookie': 'mycookie=test'});
  res.end('Hello Cookie');
}).listen(8082, () => {
  console.log('listening on port 8082')
})
```

**parseCookie 方法**

- 该方法将字符串转换为对象
- Cookie 是字符串类型，例如 `name=gplee;year=1993`，通过这个方法转化为一个对象

**createServer 方法**

- 在第一个回调方法中，将请求中的 cookie 字符串（在 req.headers.cookie 中）转化为一个对象
- 使用 res.writeHead 方法，第一个参数为 200 状态码，第二个参数为响应头的信息（Set-Cookie 字段能够让浏览器保存这个 cookie）
- 浏览器收到响应之后便将 `mycookie=test` 这个 cookie 保存下来

**在浏览器中查看 cookie**

运行上面代码，并在浏览器中访问 [http://localhost:8082](http://localhost:8082) 这个地址，能够在服务器端的控制台看到

```jsx
➜  test node index.js
listening on port 8082
/ { '': '' }
/favicon.ico { mycookie: 'test' }
```

第三行返回的是初始浏览器的第一次请求，该请求中没有任何 cookie，但是在该请求的响应里，服务器已经通知浏览器保存 `mycookie=test` 这个 cookie。最后一行是当浏览器没有在 HTML 文档中找到 favicon 的信息会再次请求 favicon 的信息，这时就能够看到 mycookie 了。

浏览器端，我们可以看到

![%5B%E7%BF%BB%E8%AF%91%5D%E7%90%86%E8%A7%A3%20JavaScript%EF%BC%88Node%EF%BC%89%E4%B8%AD%E7%9A%84%20Cookie%20%E5%92%8C%20Session%20821cab22b670445cacf2816c132b0710/1_WYrTZqQI8PaZjYLZcaLizA.png](%5B%E7%BF%BB%E8%AF%91%5D%E7%90%86%E8%A7%A3%20JavaScript%EF%BC%88Node%EF%BC%89%E4%B8%AD%E7%9A%84%20Cookie%20%E5%92%8C%20Session%20821cab22b670445cacf2816c132b0710/1_WYrTZqQI8PaZjYLZcaLizA.png)

- General 是通用的头部信息
- Request Header 是请求头独有的信息
- Response Header 是响应头独有的信息
- 我们能够在 Request Header 中看到 Cookie 字段，也能在 Response Header 中看到 Set-Cookie 字段
- Set-Cookie 字段就是服务器通知浏览器保存这个字段的 cookie 值，在浏览器保存完成之后，每次发送请求到服务器就能够带上 Cookie 字段了

### HTTP Header 和 HTTP Body

![%5B%E7%BF%BB%E8%AF%91%5D%E7%90%86%E8%A7%A3%20JavaScript%EF%BC%88Node%EF%BC%89%E4%B8%AD%E7%9A%84%20Cookie%20%E5%92%8C%20Session%20821cab22b670445cacf2816c132b0710/0_i6W-t0A6zjk7-4mP.png](%5B%E7%BF%BB%E8%AF%91%5D%E7%90%86%E8%A7%A3%20JavaScript%EF%BC%88Node%EF%BC%89%E4%B8%AD%E7%9A%84%20Cookie%20%E5%92%8C%20Session%20821cab22b670445cacf2816c132b0710/0_i6W-t0A6zjk7-4mP.png)

- 客户端请求和服务器响应都由 HTTP Header 和 HTTP Body组成（详细点讲，是（请求行+请求头）+空行+（请求内容））
- HTTP Header 中包含了请求和相应的一些描述信息
- Cookie 保存在 HTTP Header 中

### 将 Cookie 作为请求的标识符

在上面的代码例子中，我们发送的 cookie 并没有表明是谁发送的。这次我们将在 cookie 中发送一些用户的信息。

网页代码

```html
<!DOCTYPE html>
<html lang="en">
		<head>
		    <meta charset="UTF-8">
		    <title>Understand Cookie and Session</title>
		</head>
		<body>
		    <form action="/login">
		        <input id="name" name="name" placeholder="Write your name">
		        <button id="login">Login</button>
		    </form>
		</body>
</html>
```

服务器代码

```jsx
const http = require('http');
const fs = require('fs');
const url = require('url');
const qs = require('querystring');

http.createServer((req, res) => {
  const cookies = parseCookies(req.headers.cookie);
  if(req.url.startsWith('/login')) {
    const { query } = url.parse(req.url);
    const { name } = qs.parse(query);
    const expires = new Date();
    expires.setMinutes(expires.getMinutes() + 1);
    res.writeHead(302, {
      Location: '/', 
			'Set-Cookie': `name=${encodeURIComponent(name)};Expires=${expires.toGMTString()};HttpOnly; Path=/`,
    });
    res.end();
  } else if (cookies.name) {
    res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8'});
    res.end(`Welcome ${cookies.name}`);
  } else {
    fs.readFile('./index.html', (err,data) => {
      if (err) {
        throw err;
      }
      res.end(data);
    });
  }
}).listen(8083, () => {
  console.log('listening on port 8083');
})
```

我们直接来看服务器的代码：

- 第一个方法仍然是将 cookie 字符串解析成对象的方法
- 在 createServer 的回调中我们可以看到如果直接访问 [https://localhost:8083](https://localhost:8083) 这个地址，会跳过前两个判断的分支（因为请求地址没有带着 /login 的目录，初始也没有 [req.headers.cookie.name](http://req.headers.cookie.name) 这个属性，直接返回 index.html 这个静态文件，在浏览器渲染出一个提交名字的表单
- 当我们在表单输入名字并点击提交后，会请求 /login，这时服务器会使用 querystring 模块对query 字段进行解析，读出 name 字段。最后服务器会返回 302，并让浏览器在 cookie 中保存了 name 字段，并给 name 设置了一分钟的过期时间，然后重定向到 / 目录。
- 浏览器收到上面的响应之后，会根据 302 状态码重定向到 [localhost:8083](http://localhost:8083)
- 这时浏览器的请求中就有了 [cookies.name](http://cookies.name) 字段，服务器会返回 `Welcome ${cookies.name}`

### Cookie 选项

cookie 能够使用分号分隔来配置一些属性。

- `cookiename=cookievalue` ：默认的 cookie 键值对
- `Expires=date` ：cookie 的过期时间，默认是浏览器关闭就会被删除
- `Max-age=seconds` ：和 Expires 功能一样也是设置过期时间，但是是以秒计数，优先级高于 Expires
- `Domain=DomainName` : 设置 cookie 发送时的域名，默认是当前页面域名
- `Path=URL` : 指明 cookie 发送的 url 地址，默认为 '/'
- `Secure` : 只有在 https 协议下才会发送该 cookie
- `HttpOnly` : 禁止客户端 javascript 手动设置 cookie

### 使用 Session 代替 Cookie

```jsx
const http = require('http');
const fs = require('fs');
const url = require('url');
const qs = require('querystring');
const session = {};

http.createServer((req, res) => {
    const cookies = parseCookies(req.headers.cookie);
    if(req.url.startsWith('/login')) {
        const { query } = url.parse(req.url);
        const { name } = qs.parse(query);
        const expires = new Date();
        expires.setMinutes(expires.getMinutes() + 1);
        const randomInt = +new Date();
        session[randomInt] = {
            name,
            expires
        };
        res.writeHead(302, {
            Location: '/',
            'Set-Cookie': `session=${randomInt};Expires=${expires.toUTCString()};HttpOnly;Path=/`,
        });
        res.end();
    } else if (cookies.session && session[cookies.session].expires > new Date()) {
        res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8'});
        res.end(`Welcome ${session[cookies.session].name}`)
    } else {
        fs.readFile('./server4.html', (err,data) => {
            if (err) {
                throw err;
            }
            res.end(data);
        });
    }
}).listen(8084, () => {
    console.log('listening on port 8084');
})
```

这个例子中，我们在提交名字后让浏览器设置了一个名为 session 值为一个随机整数的 cookie，并在服务端维护了一个 session 的缓存对象。如果 cookie.session 没有过期，那么我们可以从中获取 session 的值。

**Session 是什么？**

- 基于 session 的身份验证，用户信息是保存在服务器端的，而浏览器和服务器之间只通过一个 session id 进行交流
- 有很多实现 session id 的方式，大部分网站使用 cookie 进行实现
- 服务端的 session 通常是维护在数据库中（redis）
- 具体实现中，我们不会直接使用一个变量作为 session id，我们经常会使用第三方模块（例如 express-session 或 cookie-parser）。这样更加安全。