layout: post
title: "Angular 脏检的细节理解"
date: 2022-07-03 22:00:00
comments: true
categories: 
- Document
tags:
- Angular
- Change Detection
- Framework
---

前端框架最主要解决的就是当数据状态改变时去更新 DOM，Angular 使用的是 change detection。change detection 一般可分为两部分，更新应用数据模型 model，Angular 根据更新后的 model 重新渲染 DOM。
大概的一个过程：

1. 开发者代码更新组件的数据模型 model（例如请求接口返回数据更新）
2. angular 检测变化
3. 脏检机制对整个应用从根到叶检测对应的数据模型是否变化
4. 如果有数据更新，则更新 DOM

<!-- more -->

change detector 是整个应用启动时创建的（每个组件都会有），它会检测值变化前后的值，如果改变了会设置 isChanged 属性为 true。

官方文档[Angular Change Detection - How Does It Really Work?]([https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/)](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/))

Angular 的 cd 是基于 JS runtime 重载机制。Angular 启动时会重载一些底层浏览器 API，将 cd、DOM更新代码塞入。这个重载主要是在 Zone.js 实现的。

zone：是能够存活多个 JS 事件循环的执行上下文？
zone 重载了以下浏览器异步 API ：所有浏览器事件（鼠标、键盘等事件）、setTimeout 和 setInterval、ajax http 请求。
cd tree：每一个 Angular 组件都有对应的 cdor，在应用启动时生成。
zone.js：可以跟踪并拦截任意的异步任务。Angular 使用 zone 并将其成为 NgZone，全局只存在一个 NgZone，cd 只在这个 NgZone 中的异步操作时触发。
对于嵌套的对象：
cd 默认对模板表达式所用的值改变进行检测（针对所有组件），对象类型不进行深度比较和检测除非模板表达式中使用了其中的一些属性值

angular 有两种 cd 策略：

- Default：默认策略，该策略在每次异步操作时从根到叶对所有组件进行检测（性能较差）
- OnPush：在 @Component 装饰器添加 `changeDetection: ChangeDetectionStrategy.OnPush` 可以启用该策略。该策略只针对以下几种情况进行 cd
- @Input 引用变化（修改对象、数组属性不触发 cd，可以引入 Immutable.js）
- 该组件和其子组件触发 event handler（setTimeout/setInterval/Promise.then/this.http.get/subscribe...不会触发 cd）
- 手动触发 cd（三种方式，ChangeDetectorRef.detectChanges()/ApplicationRef.tick()/cd（三种方式，ChangeDetectorRef.markForCheck()）
- 模板中使用 async pipe 抛出新值

深入理解[Everything you need to know about change detection in Angular]([https://indepth.dev/posts/1053/everything-you-need-to-know-about-change-detection-in-angular)](https://indepth.dev/posts/1053/everything-you-need-to-know-about-change-detection-in-angular))

### Core concept - View

Angular app 就是一棵组件树，更底层的抽象被叫做视图（View），每一个组件都和一个视图绑定。一个视图绑定着对应组件实例上的属性的引用。cd 和 DOM 更新都是在视图上进行操作的。

视图是应用 UI 的构成基础，是页面元素最小组成单位（创建和销毁）。视图中元素的属性可以被修改，但是元素个数、顺序无法被修改，只能通过 ViewContainerRef 插入、移动和删除。每个视图包含多个 View Container。每个视图与其子视图通过 nodes 属性关联。

每个 View 都有一个状态 state，angular 会根据 state 来判断是否对其和其子视图进行 cd 与否。与 cd 相关的状态有 FirstCheck、ChecksEnabled、Errored 和 Destroyed。

在状态 ChecksEnabled 为 false、或为 Errored、Destroyed 时，cd 会跳过该视图及其子视图。默认在视图初始化时状态为 ChecksEnabled（除非启动 OnPush 模式）。状态的值可以是组合的。

### cd operations

cd 主要逻辑存在于 checkAndUpdateView 方法，其功能主要是被用来递归地对组件树进行检测。对某个组件的 view 进行检测主要有以下过程：

- 为 ViewState.firstCheck 赋值，第一次 cd 设为 true，第一次 cd 完后设为 false
- 将子组件/指令实例的 @Input 属性 cd 和更新
- 更新子视图的 cd state
- 对视图进行 cd（进入递归）
- 如果子组件属性有变化，执行 nChanges 钩子
- 执行子组件的 OnInit 和 ngDoCheck
- 更新 ContentChildren
- 执行 AfterContentInit（只在第一次 cd 执行） 和 AfterContentChecked 钩子
- 如果当前组件实例属性有变化，更新 DOM 中的插入值
- 对子视图进行 cd （继续递归）