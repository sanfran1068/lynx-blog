layout: post
title: "Express源码阅读-router相关"
date: 2020-03-15 12:30:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories: 
- SourceCodeReading
tags:
- JavaScript
- 源码学习
- Express
- NodeJS
---

Express 是一种保持最低程度规模的灵活 Node.js Web 应用程序框架，为 Web 和移动应用程序提供一组强大的功能。

### Express中的app

```javascript
// express.js
function createApplication() {
    // typeof app === 'function'，同时它身上还绑定着无数属性和方法
    var app = function(req, res, next) {
        app.handle(req, res, next);
    };
    // app同时继承了事件监听/触发机制和application.js中的各种方法
    mixin(app, EventEmitter.prototype, false);
    mixin(app, proto, false);
    // 此处暴露了request和response两个类
    app.request = Object.create(req, { ... });
    app.response = Object.create(res, { ... });
    // 调用application.js中的init方法，主要调用的是defaultConfiguration方法
    app.init();
    return app;
}
// 这里需要注意的是exports和module.exports的不同：当module.exports被赋值后，exports就不同于module.exports了，所以需要这么做；当然也可以只用module.exports；
exports = module.exports = createApplication;
```

可以看出，调用express()返回的app其实是一个函数，调用app.listen()其实执行的是http.createServer(app).listen()。因此，app其实就是一个请求处理函数，作为http.createServer的参数。而express其实是一个工厂函数，用来生成请求处理函数。

### 中间件

>An Express application is essentially a series of middleware calls.
>一个Express应用实际上就是一系列中间件的调用。

中间件大致可分为两种，一种是普通中间件，通过 `app.use('/user')` 方法进行注册，该方法中的路径是匹配所有以 `/user` 开头的路径；另外一种是路由中间件，通过 `app.METHOD()` 方法进行注册，这种方式精确匹配路径，且只能处理确定请求方法的请求。

针对app.use部分的代码如下：
```javascript
app.use = function use(fn) {
  var offset = 0;
  // 默认设置路径为根路径，同时判断中间件是方法还是方法所在路径
  var path = '/';
  if (typeof fn !== 'function') {
    var arg = fn;
    while (Array.isArray(arg) && arg.length !== 0) {
      arg = arg[0];
    }
    if (typeof arg !== 'function') {
      offset = 1;
      path = fn;
    }
  }

  // router初始化
  this.lazyrouter();
  var router = this._router;
  
  var fns = flatten(slice.call(arguments, offset));
  fns.forEach(function (fn) {
    // non-express app
    if (!fn || !fn.handle || !fn.set) {
      return router.use(path, fn);
    }

    debug('.use app under %s', path);
    fn.mountpath = path;
    fn.parent = this;

    // restore .app property on req and res
    router.use(path, function mounted_app(req, res, next) {
      var orig = req.app;
      fn.handle(req, res, function (err) {
        setPrototypeOf(req, orig.request)
        setPrototypeOf(res, orig.response)
        next(err);
      });
    });

    // mounted an app
    fn.emit('mount', this);
  }, this);

  return this;
};
```

针对app.METHODS的代码如下：
```javascript
methods.forEach(function(method){
  app[method] = function(path){
    if (method === 'get' && arguments.length === 1) {
      // app.get(setting)
      return this.set(path);
    }

    // 路由初始化
    this.lazyrouter();
    var route = this._router.route(path);
    route[method].apply(route, slice.call(arguments, 1));
    return this;
  };
});
```

### 路由详解

在上面两种中间件的实现代码中，都调用了 `this.lazyrouter()` 方法，所有涉及到路由的方法都会调用这个方法，作用是初始化一个应用的内部路由。
```javascript
app.lazyrouter = function lazyrouter() {
  if (!this._router) {
    // 生成一个路由实例
    this._router = new Router({
        caseSensitive: this.enabled('case sensitive routing'),
        strict: this.enabled('strict routing')
    });
    this._router.use(query(this.get('query parser fn')));
    this._router.use(middleware.init(this));
  }
};
```
针对第一个默认路由，可以看下具体调用了哪些方法，最终实现了什么逻辑：
```javascript
    this.get('query parser fn');
    // 上面的 get 方法同 app.get，可以在 methods.forEach 循环处理时，如果 get 方法的入参长度为1时，会调用 this.set(arg)
    // 在set方法中，有针对 'query parser' 的处理
    // 最终，query 方法实现了将 req.query 进行 querystring.parse 的一个解析过程
    case 'query parser':
      this.set('query parser fn', compileQueryParser(val));
      break;
```

针对第二个默认路由 init，是给 app 上的暴露出的 req、res 继承 node 原生的 request 和 response 的一些属性。

之所以不在 `defaultConfiguration` 方法中进行这一步路由的初始化，原因在于设置路由的相关参数需要调用app.set方法，这个方法明显需要有app实例，如果在获取app实例的时候就初始化了一个路由，这个路由的参数就没办法配置了。因此，在获取app实例后，必须先对路由参数进行配置，然后再调用对应的app.use等方法。

#### app.Methods

我们先以 `app.get` 为例，通过断点调试的方式来查看 `app.get` 这种路由的 `_router` 对象是什么结构。

![702ca38d0160d8b82ab5d1b7a552bec1.png](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiZmI2ODUxODEtZjg1Yi00MTc3LTg5MTUtMTMxYTY5OTU2NWU2IiwicmVzb3VyY0d1aWQiOiIzMzk0MTlkZi05NDNlLTQzNzAtOTc0YS1mYjI5NmU3MzQxMDgifQ==)

上面的截图是当我们初始化一个 express 实例，并设置了一个 `app.get()` 的路由后，在 app.listen 处添加断点进行调试时的 app 实例的属性。可以看到在 `app._router` 中有一个 stack，里面按顺存放着三个 layer 对象，分别是初始化时的 query 和 init 两个路由，和第三个则是我们所设置的 get 路由。每一个 layer 中包含路由处理的回调函数 `handle`，路由对象 `route<Route>`，该对象中还包含一个存有 Layer 对象的栈（stack)，就和 `app._router.stack` 相同。

![81b68f825db7a3f5bf1ff7d87c09496d.png](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiZmI2ODUxODEtZjg1Yi00MTc3LTg5MTUtMTMxYTY5OTU2NWU2IiwicmVzb3VyY0d1aWQiOiJkOGY5MWM1ZC1lOWRkLTRmNDYtYmJlMC01MGRkNjJjMDUzMTYifQ==)

根据上面对于 `app.get` 这种路由的结构分析，我们可以先猜想它的实现流程：

- 首先根据传入的路径封装一个Route对象，再对传入的回调函数封装成Layer对象，接着把这个Layer对象push到Route.stack里面去

- 再创建一个默认Layer(跟app.use里面Layer的同级)，把步骤一中的Route挂到这个Layer.route属性上面，而这个Layer对象，会被push到app._router.stack里面

#### app.use

接下来我们来看 `app.use` 这种路由又有什么不同。

![08bdd22fe221f77141d8af6afb5ac8bc.png](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiZmI2ODUxODEtZjg1Yi00MTc3LTg5MTUtMTMxYTY5OTU2NWU2IiwicmVzb3VyY0d1aWQiOiIyNzA3MmYzNC03ZTY0LTQ0ODUtOTQxNC05M2RkMmVjNTkwMWYifQ==)

可以看到，在这种路由中，除了 `query` 和 `init` 两个初始化路由的方法外，第三个真正的回调方法被封装成的 `Layer` 对象中，`route` 属性的值为 `undefined`。
所以我们可以猜想：在 `app.use` 这种路由里，传入的参数（路径、回调函数）会被封装成 `Layer` 对象（其中 `route` 属性为 `undefined`），压入 `app._router.stack` 栈中。

#### 两种路由的源码实现

接下来我们通过源码来分析我们写的 `app.use('/main', someFun())` 是如何成为一个 layer 对象并压入 `app._router` 的路由栈中的。

**首先来看 `app.use`：**

```javascript
app.use = function use(fn) {
  var path = '/';   // 设置一个默认的路由方法路径
  // ...
  // 设置路由，同样也是由lazyrouter进行路由对象的初始化：this._router = new Router({})
  this.lazyrouter();
  var router = this._router;

  fns.forEach(function (fn) {
    // router.use是接下来重点要看的方法
    router.use(path, function mounted_app(req, res, next) {
      // ...
      });
    });
    // ...
  }, this);

  return this;
};
```

接下来看 Router 对象的 use 方法实现：

```javascript
proto.use = function use(fn) {
  var path = '/';
  // ...
  var callbacks = flatten(slice.call(arguments, offset));
  for (var i = 0; i < callbacks.length; i++) {
    var fn = callbacks[i];
    // 针对传入路由中的每一个回调方法，都包装成一个 Layer 对象并压入
    var layer = new Layer(path, {
      sensitive: this.caseSensitive,
      strict: false,
      end: false
    }, fn);
    layer.route = undefined;
    this.stack.push(layer);
  }
  return this;
};
```

从代码中可以看到，与我们的猜想一致，`app.use` 这种路由的实现，是将传入的一个个路径或回调方法等参数封装成 `Layer` 对象压入 `_router.stack` 栈中。

**然后来看 `app.get` 这种路由的代码实现：**

```javascript
// methods 中包含各种请求类型等，get 就包含在其中
methods.forEach(function(method){
  app[method] = function(path){
    if (method === 'get' && arguments.length === 1) {
      // app.get(setting)
      return this.set(path);
    }

    this.lazyrouter();  // 路由对象 Router 初始化
    var route = this._router.route(path);
    route[method].apply(route, slice.call(arguments, 1));
    return this;
  };
});
```

上面代码中最终返回的是 `app`，那么这端代码对 `app` 进行了什么样的操作呢？我们可以将目光聚焦在 `route[method].apply()` 这行代码上。接下来我们可以看下 `route` 到底是个什么东西。`this._router.route` 的代码实现：

```javascript
proto.route = function route(path) {
  var route = new Route(path);

  var layer = new Layer(path, {
    sensitive: this.caseSensitive,
    strict: this.strict,
    end: true
  }, route.dispatch.bind(route));

  layer.route = route;

  this.stack.push(layer);
  return route;
};
```

这里可以看到返回的 `route` 是一个 `Route` 对象，在这个对象中将参数封装成了 `Layer` 对象推入了 `app._router.stack` 中。

接下来我们看一下 `Route` 对象中的一些实现代码：

```javascript
function Route(path) {
  // Route对象初始化时添加了一个空数组stack属性
  this.path = path;
  this.stack = [];
  this.methods = {};
}

// 对于app.METHODS这种路由，将路由参数（路径、回调方法）包装成Layer对象压入Route.stack中
methods.forEach(function(method){
  Route.prototype[method] = function(){
    var handles = flatten(slice.call(arguments));
    for (var i = 0; i < handles.length; i++) {
      // ...
      var layer = Layer('/', {}, handle);
      layer.method = method;

      this.methods[method] = true;
      this.stack.push(layer);
    }
    // ...
    return this;
  };
});
```

至此，在调用 `route[method].apply()` 时就会为 `app.METHODS` 这种类型的路由的 `Layer` 中添加 `route<Route>` 属性。

![e6778eb14b0b4f77fc6bc60d49e9a053.png](https://app.yinxiang.com/files/common-services/binary-datas/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiZmI2ODUxODEtZjg1Yi00MTc3LTg5MTUtMTMxYTY5OTU2NWU2IiwicmVzb3VyY0d1aWQiOiIyYWEwMDk1ZS1jOGQwLTRkNjMtOWUzNi0yNGZmMjY4NjllOTkifQ==)
