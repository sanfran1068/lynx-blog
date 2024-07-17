---
layout: post
title: "用 JavaScript 实现有限状态机"
date: 2021-02-14 22:01:01
comments: true
categories: 
- Toolkit
tags:
- JavaScript
- 设计模式
---

当一个事物拥有有限多种状态，任一时刻只处于一种状态，而且在某种条件下可以从一种状态转变为另一种状态，满足这三个条件就可以被抽象成为一个有限状态机。当一个对象的状态越多、状态转换的事件越多，就越适合使用状态机来实现。

"javascript-state-machine"是一个很好用的有限状态机函数库。

<!-- more -->

该库提供一个全局对象StateMachine，使用该对象的create方法，可以生成有限状态机的实例。生成的时候，需要提供一个参数对象，用来描述实例的性质。比如，交通信号灯（红绿灯）可以这样描述：

```jsx
var fsm = StateMachine.create({
	initial: 'green',
  events: [
		{ name: 'warn',  from: 'green',  to: 'yellow' },
		{ name: 'stop', from: 'yellow', to: 'red' },
		{ name: 'ready',  from: 'red',    to: 'yellow' },
		{ name: 'go', from: 'yellow', to: 'green' }
　]
});
```

initial 参数是初始状态，events 参数是触发状态改变的各种事件。生成实例之后，可以通过以下接口访问当前状态：

```jsx
fsm.current // 返回当前状态。
fsm.is(s)   // 返回一个布尔值，表示状态 s 是否为当前状态。
fsm.can(e)  // 返回一个布尔值，表示事件 e 是否能在当前状态触发。
fsm.cannot(e) // 返回一个布尔值，表示事件 e 是否不能在当前状态触发。
```

其中，每个事件可以指定两个回调函数，一般为事件发生之前和发生之后两个时机触发，`onbefore$event/onafter$event`。每个状态指定两个回调函数，`onleave$status/onenter$status`。