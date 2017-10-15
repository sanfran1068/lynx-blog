layout: post
title: "Babel的使用方法"
date: 2017-09-21 12:00:00
banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg
comments: true
categories: 
- Document
tags:
- JavaScript
- 基础知识
- Babel
---

Babel是一个广泛使用的转码器，可以将ES6代码转为ES5代码，从而在现有环境执行。

<!-- more -->

##### 使用命令行转码

- 安装babel-cli：`npm install --global babel-cli`

- 使用：

	+ 转码结果输出到标准输出：`babel compiled.js`
	+ 转码结果写入一个文件：`babel source.js -o compiled.js`
	+ 整个目录转码：`babel src -d lib`
	+ 生成source map文件：`babel src -d lib -s`

- 在package.json中使用babel-cli
	
	+ 在项目中安装babel-cli：`npm install --save-dev babel-cli`
	+ 在package.json中进行配置（使用npm run build就可以进行babel转码）：
		
		```javascript
		{
		  // ...
		  "devDependencies": {
		    "babel-cli": "^6.0.0"
		  },
		  "scripts": {
		    "build": "babel src -d lib"
		  },
		}
		```


##### 安装和注册

- 安装

	+ ES2015转码规则
		npm install --save-dev babel-preset-es2015

	+ react转码规则
		npm install --save-dev babel-preset-react

	+ ES7不同阶段语法提案的转码规则（共有4个阶段），选装一个
		npm install --save-dev babel-preset-stage-0
		npm install --save-dev babel-preset-stage-1
		npm install --save-dev babel-preset-stage-2
		npm install --save-dev babel-preset-stage-3

		如果是想使用比较新的语法时，比如ES7中的async/await语法，选择安装stage-3是比较稳妥的。在安装好之后，需要在Babel的配置文件中进行注册，在项目的根目录下新建.babelrc文件并进行配置：

		```javascript
		{
		    "presets": [
		      "es2015",
		      "react",
		      "stage-2"
		    ],
		    "plugins": []
		}
		```

##### webpack配置babel

- 安装：安装方式仍如上一步npm安装方式
- 配置：

	```javascript
	//webpack1.0
	module: {
	    loaders: [
	        {
	            test: /\.js$/,
	            exclude: /(node_modules|bower_components)/,
	            loader: "babel",
	            query: {
	              presets: ['es2015', 'stage-3']
	            }
	        },
	    ]
	}

	//webpack3.0，preset如果已经在package.json中指定（`babel:{"presets": ["lastest"]}`），这里可以不用配置
	module: {
		rules: {
			{
				test: /\.js$/,
				use: 'babel-loader',
				exclude: /(node_modules|bower_components)/,
			}
		}
	}
	```

##### 开发环境使用babel-register

babel-register的作用是在模块使用**require**命令进行引入时，对该命令引入的模块进行babel的转码（只要是js/jsx/es/es6类型文件）：

- 安装babel-register：`npm install --save-dev babel-register`
- 加载babel-register：`require("babel-register");require("./index.js");`，即可对index.js进行转码（**注意：register不会对当前js文件进行转码**）

##### 使用babel-core的API进行转码

- 安装：`npm install babel-core --save`
- 使用：

	```javascript
	var babel = require('babel-core');
	// 字符串转码
	babel.transform('code();', options);
	```

##### babel-polyfill

Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。
举例来说，ES6在Array对象上新增了Array.from方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用babel-polyfill，为当前环境提供一个垫片。[^1]

例如之前在学习koa2的过程中，遇到了async/await的情况，就必须安装polyfill才能够正常加载。
