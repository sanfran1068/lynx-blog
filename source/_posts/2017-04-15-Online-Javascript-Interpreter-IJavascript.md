---
layout: post
title: "IJavascript-个人在线前端代码解释器安装与使用"
date: 2017-05-09 12:00:00
comments: true
categories:
- Document
tags:
- Jupyter
- 前端
- 解释器
- 说明书
---

#### Jupyter-开源的文档/代码编辑、解释、分享工具

ProjectJupyter[1](https://github.com/jupyter)是一项开源的项目，只在创造一个平台供各种开发人员、前沿科学工作者以及坐在电脑前进行工作的各位进行协同开发。该项目下拥有Jupyter Notebook、nbdime、qtconsole等开源应用。本文主要讲述使用Jupyter Notebook应用和IJavascript插件来进行前端代码的可视化编辑、解释以及协同分享。

![](https://img.alicdn.com/imgextra/i4/O1CN01KMRQCb1j1YNXyLSAg_!!6000000004488-49-tps-824-340.webp)

<!-- more -->

#### 安装Jupyter Notebook

*   **Python** 环境时安装Jupyter Notebook的必要环境（3.3以上或2.7均可）
*   可以先从官网下载Anaconda安装包，Anaconda中继承了NumPy和Jupyter Notebook等工具包，所以直接可以打开Jupyter Notebook应用
*   macOS系统下可以通过终端的pip工具进行安装：

    *   首先要升级pip到最新版本：

        ```javascript
        pip3 install --upgrade pip
        ```

        如果pip3命令不存在，需要安装pip3或者（如果电脑中安装的是python2）使用pip命令进行安装。

    *   使用pip3命令进行安装jupyter：

        ```javascript  	
        pip3 install jupyter
        ```

#### 安装IJavascript

Jupyter Notebook自带python语言编写程序的编译，如果需要其他语言版本的notebook（Julia，Ruby，Haskell等），还需要安装相应语言对应的Kernel[2](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels)，对于前端工程师来说，当然是需要一个Html、CSS和JavaScript集成的在线解释器，所以IJavascript[3](https://github.com/n-riesco/ijavascript)就应运而生。IJavascript是Jupyter Notebook的Kernel（支持该功能的是Jupyter messaging protocol）之一。IJavascript同时也在npm包管理工具中，而npm又需要Node.js环境。所以，需要首先确定系统中有Node.js和npm的环境。

在终端中运行以下代码可以完成IJavascript Kernel的安装：

```javascript
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install pkg-config node zeromq
sudo easy_install pip
sudo pip install --upgrade pyzmq jupyter
sudo npm install -g ijavascript
```

#### IJavascript的使用

在终端中输入代码

```javascript
jupyter notebook
```

启动Jupyter Notebook，可以在浏览器打开地址为[http://localhost:8888的web应用，在右上角“new”选项中选择Javascript(Node.js),就可以开始使用了！](http://localhost:8888的web应用，在右上角“new”选项中选择Javascript(Node.js),就可以开始使用了！)

IJavascript不仅可以像控制台一样进行交互式的JavaScript调试，而且可以对HTML和SVG代码进行输出，只需要在输入的时候加上$$html$$或$$svg$$就可以进行代码的所见即所得了。详细的教程可以查看IJavascript官方文档[4](http://n-riesco.github.io/ijavascript/index.html).