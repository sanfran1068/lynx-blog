---
layout: post
title: "Angular 的脏检到底如何工作"
date: 2022-06-12 22:30:00
comments: true
categories: 
- Document
tags:
- Angular
- Change Detection
- Framework
---

### 脏检如何实现？

Angular 的变化检测机制是建立在 Javascript 的可覆写机制。Angular 在启动阶段会对浏览器的一些底层 API 进行补丁，比如在 addEventListener 方法中添加一些功能：

```jsx
function addEventListener(eventName, callback) {
     // call the real addEventListener
     callRealAddEventListener(eventName, function() {
        // first call the original callback
        callback(...);     
        // and then run Angular-specific functionality
        var changed = angular.runChangeDetection();
         if (changed) {
             angular.reRenderUIPart();
         }
     });
}
```

这个为底层浏览器 API 打补丁的过程是由 zone.js 工具库提供的。补丁和脏检机制一班发生在以下几种异步情况：

- 所有浏览器事件（鼠标事件、键盘事件...）
- setTimeout() 和 setInterval()
- Ajax HTTP 请求

为了了解 zone 的概念，我们需要先了解浏览器任务和时间循环的机制。

<!-- more -->

Angular 官方文档中指出，一个 zone 指的是跨多个异步任务存在的执行上下文。

> 执行上下文可理解为作用域，JavaScript 有全局上下文（相对于函数之外的任何代码）、本地上下文（函数执行时自己创建）和由 eval()上下文（由 eval 函数创建的上下文）。程序开始运行，全局上下文即创建，执行到某个函数时则会为该函数创建其上下文，并推入执行上下文栈中；退出该函数或该程序时，其上下文也会从执行栈中弹出并销毁。
> 

zone.js 就是通过创建一个持久的跨多个异步操作的执行上下文，并提供了异步操作的各个生命周期（onScheduleTask、onInvokeTask、onHasTask、onInvoke），在这些生命周期中可以监测数据变化，从而达到脏检的目的。

### JavaScript 运行时和事件循环（[来源](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide/In_depth)）

为了了解 zone 的机制，我们还需要复习下 JavaScript 运行时。 执行 JS 代码时，JavaScript 运行时维护了一组用于执行 JS 代码的代理，每个代理包含一组执行上下文的集合、执行上下文栈、主线程、一组可能创建用于执行 worker 的额外的线程集合、一个任务队列以及一个微任务队列构成。

#### 事件循环

每个线程都拥有自己的事件循环。每个循环负责收集事件（用户、非用户事件），对任务进行排队（安排回调的执行），执行排队中的 Task（即宏任务），然后执行微任务，最后在下一次循环之前执行一些必要的渲染和绘制操作。事件循环有三种：

- Window 事件循环：驱动所有同源（意为起源，即一个窗口中打开另一个窗口或一个窗口中有多个 iframe，可能会共享一个事件循环）窗口。
- Worker 事件循环：驱动 workder 的事件循环
- Worklet 事件循环：用于驱动运行 worklet 的代理

#### 任务和微任务

任务是指按标准机制执行的任何 JavaScript，包括程序初始化、事件触发的回调、setTimeout 添加的任务。微任务指特定的一些任务，例如 Promise、Object.observe、MutationObserver、 NodeJS 环境下的 process.nextTick 和 async/await

- 每一次事件循环的迭代会执行队列中的每一个任务，每次迭代开始后加入的任务在下一次迭代才被执行
- 每次当一个任务退出且执行上下文为空时，事件循环会执行微任务队列中的每一个微任务，直到微任务队列为空才停止，期间新添加的微任务也会在这个队列被执行。可以通过 queueMicrotask() 方法添加新的微任务。

这也就是为什么 promise 的执行顺序要在 setTimeout 回调之前的原因。

### 变化检测树

每个 Angular 组件都有对应的一个变化检测器，在应用启动时被创建。Angular 的数据流自顶向下单向流动，变化检测树与之呼应（检测完父组件后，子组件也可能改变父组件的数据使得父组件再次被检查，所以在开发环境会出现 ExpressionChangedAfterItHasBeenCheckedError 报错）。

需要注意的是，Angular 使用 view 作为底层抽象，一个 view 与一个 component 相互关联，所有的属性值的变化检测和 DOM 更新都是在 view 上操作。自顶向下，每一个 view 都与其 child views 链接，通过 nodes 属性对 child views 进行操作。每一个 view 都有一个状态（state，分别有 FirstCheck、ChecksEnabled、Errored 和 Destroyed 四种状态，默认策略下，状态可以相互组合）。

针对 view 变化检测的主要逻辑是在 checkAndUpdateView 方法中。

> 默认的检测机制，只在模板表达式的值发生改变时触发；并且不会针对对象进行深比较（只对模板中使用到的属性进行检测）。
> 

上面提到的单向流动同时也表示数据更新后视图会随之更新，但是视图的更新不会触发更多的视图更新。在 ngAfterViewChecked 生命周期里调用 this.callbacks 会触发 "EXCEPTION: Expression '{{message}} in App@3:20' has changed after it was checked" 的报错（该报错仅在开发环境中出现）。

有时候变化检测不到怎么办？

在开发过程中，经常会遇到某一些数据变化但是视图上没有展示出来，甚至将数据打印出来已经发生了变化，但是视图上却不会变化的情况。

- 第一种情况可能是由于修改了对象或数组类型的某个属性，这时导致的脏检不成功，整个情况尝试整个变量重新赋值。
- 第二种情况可能是触发试图改变，而视图又触发视图改变，这个时候可以尝试手动使用 setTimeout() 方法进行包裹。
- 还有一种比较特殊的情况，上面我们提到了 Angular 对三种类型的一部操作进行变化检测，而当我们使用第三方的库导致的异步操作，但这些操作并没有触发到以上三种底层的异步操作时，变化就没办法被检测到。这个时候可以尝试引入 NgZone，使用 NgZone.run 包裹这个异步操作即可。

### 参考文档

[官方文档](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/)

[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

[The event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)

[When to use Ngzone.run()?](https://stackoverflow.com/questions/51455545/when-to-use-ngzone-run)