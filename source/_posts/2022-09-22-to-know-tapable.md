---
layout: post
title: "认识 Tapable"
date: 2022-09-22 23:30:00
comments: true
categories: 
- SourceCodeReading
tags:
- Webpack
- Tapable
- Compile Tool
---

实现了一种发布订阅者模式的插件机制。通过 Tapable 我们可以注册时间，从而在不同时机去触发注册的事件进行执行。Tapable 怎么使用呢？

```jsx
// 初始化 hook，此处的字符串数组比较重要
const hook = new SyncHook(['arg1', 'arg2', 'arg3']);

// 为 hook 注册事件，同步 hook 只有 tap 方法，
// 接受两个参数，一个是字符串的标识，第二个是注册的函数，在调用时执行
hook.tap('flag1', (arg1, arg2, arg3) => {
    console.log('flag1:', arg1, arg2, arg3)
})
hook.tap('flag2', (arg1, arg2, arg3) => {
    console.log('flag2:',arg1, arg2, arg3)
})

// 调用事件并传递执行参数
hook.call('19Qingfeng', 'wang', 'haoyu')

// 打印结果
// flag1: 19Qingfeng wang haoyu
// flag2: 19Qingfeng wang haoyu
```

<!-- more -->

### hook 的分类

#### 根据同步和异步分类

![6000000002244-0-tps-1313-685](https://img.alicdn.com/imgextra/i2/O1CN01sDdVcy1SRnbq3kJc8_!!6000000002244-0-tps-1313-685.jpg)

同步的 hook，只有 tap 是唯一的注册事件的方法，通过 call 方法触发 hook 的执行。

异步的 hook，可以通过 tap、tapAsync 和 tapPromise 三种方式来注册，可以通过 call、callAsync 和 promise 三种方式来触发注册函数。tapAsync 注册函数额外接受一个 callback 参数，调用 callback 表示函数执行完毕。tapAsync 和 tapPromise 如果有错误对象或 reject，则中断执行。

异步的 hook，另外还可以分为可串行和可并行执行两种类型。

#### 根据执行机制分类

- Basic Hook：仅执行钩子注册的函数，不关心任何的函数返回值。例如 SyncHook；
- Waterfall：类似与基础类型钩子，但是函数执行时会将非 undefined 的返回值传递给后续的函数作为参数。例如 SyncWaterfallhook;
- Bail：如果注册函数执行后返回非 undefined 的值，则会中断整个 hook 的函数执行。例如 SyncBailHook；
- Loop：如果注册函数执行后返回非 undefined 的值，会立刻葱头开始重新执行函数，直到所有注册函数都返回 undefined。例如 SyncLookHoop；

### 拦截器

Tapable 提供的所有 Hook 都支持注入 Interception，通过拦截器对整个 Tapable 发布/订阅流程进行监听，从而触发对应的逻辑。

```jsx
const hook = new SyncHook(['arg1', 'arg2', 'arg3']);

hook.intercept({
  // 每次调用 hook 实例的 tap() 方法注册回调函数时, 都会调用该方法,
  // 并且接受 tap 作为参数, 还可以对 tap 进行修改;
  register: (tapInfo) => {
    console.log(`${tapInfo.name} is doing its job`);
    return tapInfo; // may return a new tapInfo object
  },
  // 通过hook实例对象上的call方法时候触发拦截器
  call: (arg1, arg2, arg3) => {
    console.log('Starting to calculate routes');
  },
  // 在调用被注册的每一个事件函数之前执行
  tap: (tap) => {
    console.log(tap, 'tap');
  },
  // loop类型钩子中 每个事件函数被调用前触发该拦截器方法
  loop: (...args) => {
    console.log(args, 'loop');
  },
});
```

- register：注册 tap、tapAsync、tapPromise 函数时，会触发该拦截器；接受注册的 tap 作为参数，同时可以对于注册的事件进行修改。
- call：调用 hook 实例的 call 方法时执行，接受的参数就是函数传入的参数。
- tap：每个被注册的函数调用之前执行，接受的参数是对应的 Tap 对象。
- loop：每次重新开始循环时执行，接受的参数就是函数传入的参数

### Tapable 其他 API

注册函数时，之前我们传入的第一个参数为标识符字符串，其实它还可以是一个对象，这个对象表示注册函数时的一些其他配置。

#### before

before 属性的值可以传入一个数组或者字符串,值为注册事件对象时的名称，它可以修改当前事件函数在传入的事件名称对应的函数之前进行执行。

```jsx
hooks.tap({
	  name: 'flag1',
	},
  () => console.log('This is flag1 function.')
);
hooks.tap({  // flag2 事件函数会在flag1之前进行执行
    name: 'flag2',
    before: 'flag1',
  },
  () => console.log('This is flag2 function.')
);
```

#### stage

该属性能够标识注册函数执行的前后权重，类型是数字，默认为0，也支持传入负数，数字越大其回调函数执行的越晚。

结合上面的 before 属性，如果同时使用，优先会处理 before 属性值，满足该条件后再判断 stage 属性值的判断。所以使用 stage 属性时最好不要使用 before 属性。

#### Hookmap

本质上是一个辅助类，可以更好地管理 hook。另外还有 MultiHook，主要作用也就是通过 MultiHook 批量注册事件函数在多个钩子中。

```jsx
// 创建 HookMap 实例
const keyedHook = new HookMap((key) => new SyncHook(['arg']));

// 在 keyedHook 中创建一个 name 为 key1 的 hook，同时为该 hook 通过 tap 注册事件 
keyedHook.for('key1').tap('Plugin 1', (arg) => {
  console.log('Plugin 1', arg);
});

// 从 HookMap 中拿到 name 为 key1 的 hook
const hook = keyedHook.get('key1');
if (hook) {
  // 通过call方法触发Hook
  hook.call('hello');
}
```

### 深入源码

https://github.com/sanfran1068/tapable