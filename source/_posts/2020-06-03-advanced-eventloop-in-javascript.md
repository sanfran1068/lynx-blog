layout: post
title: "JavaScript 基于事件循环的并发模型"
date: 2020-06-03 16:31:00
comments: true
categories: 
- Document
tags:
- JavaScript
---

不去了解 JavaScript 的事件循环就相当于没有入门 JavaScript 这门语言程序，本文第一次对事件循环进行了了解和学习。

<!-- more -->

runtime（运行时刻）是指一个程序在运行的状态。JavaScript 的运行时刻有一个基于事件循环的并发模型，事件循环负责执行代码、收集和处理事件以及执行队列中的子任务。

![JavaScript%20%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B%20b0cfb1289e8e4098b7cae991e050b301/The_Javascript_Runtime_Environment_Example.svg](JavaScript%20%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B%20b0cfb1289e8e4098b7cae991e050b301/The_Javascript_Runtime_Environment_Example.svg)

JavaScript 中的函数调用会形成一个由若干帧组成的栈，当一个函数中存在闭包函数时，外层函数会先压入栈中，其中包含参数和局部变量；闭包函数跟着压入栈中，也包含了自己的参数和局部变量。闭包函数先从栈中弹出，执行完了之后外层函数从栈中弹出执行，栈就被清空了。

对象是被分配在堆中。

一个 JavaScript 运行时包含了一个待处理消息的消息队列。每一个消息都关联着一个用以处理这个消息的回调函数。

## 事件循环

```jsx
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

JavaScript 中每一个函数都是需要完整执行完成才会继续进行下一步（单线程语言：one thread ⇒ one call stack ⇒ one thing at a time），所以 JavaScript 的代码很容易出现一个函数执行时间过长的情况。为了实现程序的并发执行，JavaScript 引入了事件循环机制。

在上面所提到的函数调用栈之外，还会有一个任务队列。当 runtime 执行代码过程中遇到了像 setTimeout 或 DOM 事件绑定的回调函数，会首先调用 WebAPIs 去进行执行——例如 setTimeout 会产生一个 timer 计时器，当计时器到达指定时间后，会将这个事件推入任务队列。任务队列就是由一个个异步操作的回调函数所组成的队列。事件循环简单来说就是：当 runtime 函数调用栈清空后，查看当前任务队列，并把队列第一个任务压入 runtime 函数调用栈。

浏览器的 render 刷新最快是 16.6ms 一次（浏览器刷新一般是 60次/s ），这是在主线程为空的时候（所以任务队列里的任务是不会阻塞浏览器渲染的）。**如果想在浏览器渲染任务队列中添加任务，可以使用 requestAnimationFrame(callback)方法来进行添加**。如果函数调用栈不为空，则 render 会被阻塞：主线程中的死循环会阻塞浏览器渲染，但是看下面的代码： 

```jsx
function loop() {
	setTimeout(loop, 0);
}
loop();
```

这个无限自我递归调用也会形成一个循环，但是由于 setTimeout 方法只是不停地往任务队列里面添加任务，而主线程还是每次取一个任务去执行，主线程不会被阻塞，所以浏览器渲染也不会被阻塞。

到目前为止，在事件循环中，我们认识到了主线程、任务队列和渲染任务队列。当主线程任务全部执行完成清空后，会从非空的任务队列中拿第一个任务到主线程尽心执行，每次循环都是只取一个任务队列中的任务。而渲染任务则是一次性执行完成的。

![JavaScript%20%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B%20b0cfb1289e8e4098b7cae991e050b301/_1.png](JavaScript%20%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B%20b0cfb1289e8e4098b7cae991e050b301/_1.png)

实际上，浏览器总还有一个 microTask queue，这个队列中一般我们遇到的 99.9% 应该都是以 promise 形式出现的，microTask 执行的时间是每一段 JS 代码执行完毕（主线程任务清空、任务队列取得第一个任务到主线程并执行完、浏览器渲染任务执行完）都会进行执行，而且一旦执行，microTask queue 中无论有多少个任务（包括同时添加进来的任务）都会一直执行完。所以，microTask queue 是会产生阻塞的。

![JavaScript%20%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B%20b0cfb1289e8e4098b7cae991e050b301/_1%201.png](JavaScript%20%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B%20b0cfb1289e8e4098b7cae991e050b301/_1%201.png)

如上图所示，我们所知的浏览器的事件循环有以下的几种队列机制：

- 主线程任务队列

    接受 JS 程序中按顺序的同步任务，在每次事件循环中都会执行任务直至队列清空

- 异步回调任务队列

    接受像 setTimeout、ajax 等的异步回调函数任务，在每次时间循环中主线程任务队列清空后，会拿出该队列的头部任务放入主线程进行执行

- microtask 任务队列

    接受 promise.then 的回调函数任务，在事件循环中，每一段 JS 代码执行完毕（主线程任务清空、任务队列取得第一个任务到主线程并执行完、浏览器渲染任务执行完）都会执行，且每次执行都会将该队列清空，包括同时入队的任务也会一并清空

- 动画帧回调任务队列

    接受 requestAnimationFrame 的回调函数任务，在每次浏览器渲染任务执行之前进行执行（目前仅在 chrome 中是该顺序），每次执行都清空当前队列，如有新的任务添加则等待下次渲染前执行

- 浏览器渲染任务队列

    每当上面主线程、异步任务队列、microTask 任务队列执行完毕，且距离上一次渲染时长大于 16.6ms 时，就会进行任务执行，每一次执行都会将当前队列任务全部执行完毕

用一个伪代码来进行上面流程的表述，如下：

```jsx
while (true) {
	while (mainThread.hasTasks()) {
		execute(mainThread.getNextTask());
	}

	queue = getNextQueue();
	task = queue.pop();
	execute(task);

	while(microTask.hasTasks()) {
		doMicroTask();
	}

	if (isRepaintTime()) {
		animationTasks = animationQueue.copyTasks();
		for (task in animationTasks) {
			doAnimationTask();
		}
	}

	repaint()
}
```

## Node 中的事件循环

NodeJS 的事件循环基本上与浏览器中的事件循环是一致的，只不过浏览器使用的是 WebAPIs，而 NodeJS 中使用的是 C++ APIs。

![JavaScript%20%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B%20b0cfb1289e8e4098b7cae991e050b301/_1%202.png](JavaScript%20%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B%20b0cfb1289e8e4098b7cae991e050b301/_1%202.png)

主线程与异步任务队列执行以及 microTask 任务队列都与浏览器中的事件循环相同，我们可以看到上图中多了三个没有见到过的任务队列：

- check phase 任务队列

    接受使用 setImmediate 方法传入的回调函数任务，每次在异步任务队列执行完成后，清空该队列

- timer phase 任务队列

    接受 setTimeout、setInterval 传入的回调函数，每次在 check phase 任务队列清空后，清空该任务队列

- nextTick 任务队列

    与 microTask 任务队列相同， 接受 process.nextTick 传入的回调函数，该队列优先级高于 microTask 

用伪代码进行上述流程的模拟，如下：

```jsx
while (tasksAreWaiting()) {
	queue = getNextQueue();
	while (queue.hasTasks()) {
		task = queue.pop();
		execute(task);
		
		while (nextTickQueue.hasTasks()) {
			doNextTickTask();
		}
		
		while (promiseQueue.hasTasks()) {
			doPromiseTask();
		}
	}
}
```