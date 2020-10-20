layout: post
title: "JavaScript模块化"
date: 2020-01-12 12:30:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories: 
- Reading
tags:
- JavaScript
- 读书笔记
- 规范
---

#### 名词定义先行

模块化主要解决以下三个问题：

- 代码分离
- 不同模块间的依赖定义
- 代码到执行环境的传递

能够解决其中一两个点的解决方案我们称之为“**模式**”，能够解决全部三个问题的方案我们称之为“**模块系统**”；

我们将封装好的导出实例（对象、方法等）和引入的实例称为“**模块格式**”；用“分离依赖定义（detached dependency definitions，即用不同文件存储不同文件）”来描述模块系统中可被使用的独立依赖。

#### 关于模块化所解决的问题

- 命名冲突

在1995-1999年间，使用var来定义全局变量是非常方便的，因为当时的JavaScript就是用来编写脚本处理小微任务，而随着应用的代码量上升，全局变量的缺点就越来越明显，因此我们甚至不能引用第三方脚本。
```javascript
// file greeting.js
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

function writeHello(lang) {
    document.write(helloInLang[lang]);
}

// file hello.js
function writeHello() {
    document.write('The script is broken');
}
```

- 支持大量代码仓库

由于编写应用的代码量越来越大，我们常常需要将代码分为多个脚本文件逐一引入，这导致了越来越多数量的脚本文件需要我们手动维护，而且还要考虑脚本引入的顺序！这真是太令人头痛！

#### Directly Defined Dependencies

1999年提出的首次对于独立依赖的模式“直接定义依赖”。该模式是由 Erik Arvidsson 在1999年提出。

```javascript
// file greeting.js
dojo.provide("app.greeting");

app.greeting.helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

app.greeting.sayHello = function (lang) {
    return app.greeting.helloInLang[lang];
};

// file hello.js
dojo.provide("app.hello");

dojo.require('app.greeting');

app.hello = function(x) {
    document.write(app.greeting.sayHello('es'));
};
```

代码使用dojo 1.6编写，其中dojo.provide用来定义模块，获取模块时需要使用dojo.require

#### 命名空间模式

在JavaScript中，函数是`first class citizens`，意味着可以被赋给变量，也可以被函数返回。命名空间模式是由 Erik Arvidsson于2002年发明的互联网应用开发工具Bindows开始形成。该模式类似于：

```javascript
// file app.js
var app = {};

// file greeting.js
app.helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

// file hello.js
app.writeHello = function (lang) {
    document.write(app.helloInLang[lang]);
};
```

可见，所有的逻辑和数据都被放在了app对象的属性中，所以不会污染其他全局变量。尽管该模式看似让代码组织有了一定的规律，但是很明显上面的数据和逻辑仍然没有被隔离，我们很轻易就可以修改这些代码。

#### 模块模式

模块模式的主旨是将数据和逻辑代码放在闭包中并提供一些公用方法作为对外接口对这些资源进行访问[JavaScript Module Pattern: In-Depth](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)。例如：

```javascript
var greeting = (function () {
    var module = {};

    var helloInLang = {
        en: 'Hello world!',
        es: '¡Hola mundo!',
        ru: 'Привет мир!'
    };

    module.getHello = function (lang) {
        return helloInLang[lang];
    };

    module.writeHello = function (lang) {
        document.write(module.getHello(lang))
    };
    
    return module;
}());
```

这种将数据和逻辑闭包在立即调用方法中的方式在2008年Douglas Crockford发表的《JavaScript the Good Parts》一书中被称作“模块”。

#### 模板定义依赖（Template Defined Dependencies)

当有多个script文件依赖时，我们可以使用一个模板来将这些依赖进行顺序管理：

```javascript
// file app.tmp.js

/*borschik:include:../lib/main.js*/
/*borschik:include:../lib/helloInLang.js*/
/*borschik:include:../lib/writeHello.js*/

// file main.js
var app = {};

// file helloInLang.js
app.helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

// file writeHello.js
app.writeHello = function (lang) {
    document.write(app.helloInLang[lang]);
};
```

#### 注释定义依赖

```javascript
// file helloInLang.js
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

// file sayHello.js

/*! lazy require scripts/app/helloInLang.js */

function sayHello(lang) {
    return helloInLang[lang];
}

// file hello.js

/*! lazy require scripts/app/sayHello.js */

document.write(sayHello('en'));
```

这种依赖的工作方式是：首先下载这些依赖文件，并进行文件内容解析，解析到有依赖存在的注释时迭代下载、解析……

#### 外部定义依赖

```javascript
// file deps.json
{
    "files": {
        "main.js": ["sayHello.js"],
        "sayHello.js": ["helloInLang.js"],
        "helloInLang.js": []
    }
}

// file helloInLang.js
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

// file sayHello.js
function sayHello(lang) {
    return helloInLang[lang];
}

// file main.js
console.log(sayHello('en'));
```

deps.json文件就是我们定义所有依赖的外部上下文依赖文件。当运行这个应用时，加载器会获取到这个文件，将这个文件中的依赖按照数组的正确顺序进行读取和加载。lodash就是使用的这种方法进行依赖加载的。

#### 沙箱模式

```javascript
// file sandbox.js
function Sandbox(callback) {
    var modules = [];

    for (var i in Sandbox.modules) {
        modules.push(i);
    }

    for (var i = 0; i < modules.length; i++) {
        this[modules[i]] = Sandbox.modules[modules[i]]();
    }
    
    callback(this);
}

// file greeting.js
Sandbox.modules = Sandbox.modules || {};

Sandbox.modules.greeting = function () {
    var helloInLang = {
        en: 'Hello world!',
        es: '¡Hola mundo!',
        ru: 'Привет мир!'
    };

    return {
        sayHello: function (lang) {
            return helloInLang[lang];
        }
    };
};

// file app.js
new Sandbox(function(box) {
    document.write(box.greeting.sayHello('es'));
});
```

该模式的关键是使用一个全局的构造函数来代替全局对象，依赖模块可以被定义为这个构造函数的属性。

#### 依赖注入

```javascript
// file greeting.js
angular.module('greeter', [])
    .value('greeting', {
        helloInLang: {
            en: 'Hello world!',
            es: '¡Hola mundo!',
            ru: 'Привет мир!'
        },

        sayHello: function(lang) {
            return this.helloInLang[lang];
        }
    });

// file app.js
angular.module('app', ['greeter'])
    .controller('GreetingController', ['$scope', 'greeting', function($scope, greeting) {
        $scope.phrase = greeting.sayHello('en');
    }]);
```

#### CommonJS Modules

```javascript
// file greeting.js
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

var sayHello = function (lang) {
    return helloInLang[lang];
}

module.exports.sayHello = sayHello;

// file hello.js
var sayHello = require('./lib/greeting').sayHello;
var phrase = sayHello('en');
console.log(phrase);
```

CommonJS使用require和module两个标志来表示依赖的引入和导出。

#### AMD

```javascript
// file lib/greeting.js
define(function() {
    var helloInLang = {
        en: 'Hello world!',
        es: '¡Hola mundo!',
        ru: 'Привет мир!'
    };

    return {
        sayHello: function (lang) {
            return helloInLang[lang];
        }
    };
});

// file hello.js
define(['./lib/greeting'], function(greeting) {
    var phrase = greeting.sayHello('en');
    document.write(phrase);
});
```