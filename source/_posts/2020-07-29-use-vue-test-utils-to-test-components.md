layout: post
title: "使用 vue-test-utils 进行测试"
date: 2020-07-29 22:23:00
<!-- banner: http://oqcytejyk.bkt.clouddn.com/post-bg-javascript%E7%9A%84%E5%89%AF%E6%9C%AC.jpg -->
comments: true
categories: 
- Document
tags:
- JavaScript
- Vue
- Test
---
## 插件安装

我们使用 Vue CLI 脚手架进行项目的创建，按照提示一步步进行项目配置，尽量都选择无任何配置，尤其是自带的测试插件：

```jsx
vue create vue-test
```

新建好项目之后，我们需要在项目根目录下**新建一个 test 目录**来放置我们的测试文件。

由于我们是在 Vue 项目中进行测试，采用的是 vue-test-utils 这个测试工具插件进行测试，所以下面几类 npm 插件是必须要安装的：

- jest 相关

    jest 是 vue-test-utils 官方推荐的测试运行器，所以这个插件是必要的；要使用 jest 来处理 *.vue 文件，需要安装和配置 vue-jest 插件。

    ```jsx
    npm i --save-dev jest vue-jest
    ```

    ```jsx
    // 在 package.json 中做如下配置
    {
    	"scripts": {
    		"test": "jest"
    	},
    	"jest": {
        "moduleFileExtensions": [
          "js",
          "json",
          "vue"
        ]
      }
    }
    ```

- babel 相关

    我们一般都不可避免地希望在代码中使用 ES2015 的特性，这时我们需要安装 babel 相关的插件，其中 babel-jest 是测试中的 ES2015 特性编译的插件

    ```jsx
    npm install --save-dev babel-core babel-preset-env babel-jest
    ```

    ```jsx
    // 在 package.json 中做如下配置
    {
    	"jest": {
    		"transform": {
          ".*\\.(vue)$": "vue-jest",
          "^.+\\.js$": "<rootDir>/node_modules/babel-jest"
        }
    	}
    }

    // 在 .babelrc 或 babel.config.js 文件中做如下配置
    {
      "presets": [["env", { "modules": false }]],
      "env": {
        "test": {
          "presets": [["env", { "targets": { "node": "current" } }]]
        }
      }
    }
    ```

- vue-test-utils 依赖浏览器环境，我们使用 JSDOM 在 Node 虚拟浏览器环境运行测试。jest 中有自动设置 JSDOM，如果没有，需要安装以下插件

    ```jsx
    npm install --save-dev jsdom jsdom-global
    ```

至此，所以前期的插件安装和配置工作就完成了。这时，我们在 test 目录下新建一个 *.test.js 文件，并在 terminal 中运行 npm run test 命令，就可以进行 vue 组件的测试了。