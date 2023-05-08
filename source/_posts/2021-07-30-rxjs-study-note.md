layout: post
title: "RxJS 学习"
date: 2021-07-30 23:01:01
comments: true
categories: 
- Document
tags:
- Angular
- RxJS
---

RxJS 是异步和事件驱动编程的一个库，主要使用 Observable sequence（可观察序列）。RxJS 提供一个核心类 Observable，和一些卫星类（Observer, Schedulers, Subjects）、操作符（Operators）。

RxJS 是适用于“事件”的 Lodash 工具库。

<!-- more -->

RxJS 中异步事件管理的核心概念有如下：

- **Observable:** 定义的在未来可能发生能被观察到的被观察事件
- **Observer:** 当被观察事件发生时的观察者，一般都是回调函数
- **Subscription:** 将观察者订阅在被观察事件上的订阅动作，可以用来取消订阅
- **Operators:** 是一系列纯函数，将被观察事件的结果进行函数式编程处理传送到观察者中，例如 map、filter、concat 和 reduce等
- **Subject:** 相当于一个事件发射器，收到一个信息后将值/事件广播到多个观察者。
- **Schedulers:** 并发控制器，能够协调并发事件的返回顺序？

RxJS 的核心链路：

1. 建立可观察的 Observable： `var clicks$ = rxjs.fromEvent(document, 'click')`
2. 建立观察者 Observer： `var observer = { next: (x) => { console.log(x) } }`
3. 建立订阅关系 Subscription： `var subs$ = clicks$.subscribe(observer);`
4. 取消订阅关系： `subs$.unsubscribe()`

该链路中可以对 Observable 产生的结果进行处理再传入 Observer，这个过程需要用到 Operators，再最新版本中，应该将这些函数式编程通过 pipe() 方法进行流式处理：

```jsx
import { fromEvent } from 'rxjs';
import { throttleTime, map, scan } from 'rxjs/operators';

fromEvent(document, 'click').pipe(
  throttleTime(1000),
  map(event => event.clientX),
  scan((count, clientX) => count + clientX, 0)
).subscribe(count => console.log(count));
```

RxJS 的中心思想：

```jsx
fromEvent(document, 'click').subscribe(() => console.log('Clicked!'));
// Observable.subscribe(Observer);
```

## Observer

是 Observable 生产的数据的消费者，由一系列回调函数组成 next, error, and complete。使用时，需要将 Observer 作为参数传入 subscribe() 方法中：

```jsx
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};

observable.subscribe(observer);
```

简单来说，Observer 就是一个拥有三个回调函数的对象，每个回调函数对应 Observable 的三种生产结果。

## Observable

可以将 Observable 理解为数据生产者，将 Observer 理解为数据消费者。

### Push vs Pull

数据生产者和消费者之间的沟通有两种模式：push 和 pull。pull 模式是指消费者来决定什么时候生产数据，类似于 ajax 请求服务器返回数据（每个 JavaScript 方法都是这种模式，还有新推出的 Generator 函数）。push 模式是指生产者来决定什么时候发送数据到消费者，Promise 是最常见的 push 模式。

Observable 就是一个 push 模式的数据生产者，将生产的数据送到观察者手中。

### 创建 Observable

**第一种方式是通过构造函数来创建：**

```jsx
// Observable 构造函数需要传入一个方法，该方法可以有内部逻辑，也可以实现给 Observer 发送数据（调用 .next() 推入数据）
const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
});

// 订阅时，需要将 Observer 传入
foo.subscribe(x => {
  console.log(x);
});
```

Observable 和 Promise、EventEmitter 都不一样。函数和 Observable 都是惰性的，即不调用/不订阅时，函数和 Observable 都不会发生。为 Observable 订阅观察者就像调用一个函数。Observable 可以**同步或异步**地将数据生产给 Observer。

**第二种方式是通过 creation operator：**

```jsx
first()(of(1, 2, 3)).subscribe((v) => console.log(`value: ${v}`));
```

### 订阅 Observable

```jsx
observable.subscribe(x => console.log(x));
```

上面说到订阅 Observable 就像函数调用，在 subscribe() 的参数中提供回调函数，Observable 会将生产的数据发送到这些回调函数中。

### 执行 Observable

Observable 执行指的就是将生产数据经过函数式编程处理传送到 Observer 回调函数中。执行会有三种情况，分别由三种回调进行传递：

- Next 回调：正常生产数据到观察者
- Error 回调：try……catch 或 throw 出 JavaScript Error or exception 到观察者
- Complete 回调：不传递任何数据，是一个空参数的方法

```jsx
import { Observable } from 'rxjs';
 
const observable = new Observable(function subscribe(subscriber) {
  try {
    subscriber.next(1);
    subscriber.next(2);
    subscriber.complete();
  } catch (err) {
    subscriber.error(err); // delivers an error if it caught one
  }
});
```

### 取消订阅 Observable

```jsx
import { from } from 'rxjs';

const observable = from([10, 20, 30]);
const subscription = observable.subscribe(x => console.log(x));
subscription.unsubscribe();
```

unsubscribe() 方法可以将已经触发的 Observable 停止触发。可以在创建 Observable 时，在其中返回一个 unsubscribe 函数来指明如何取消该 Observable 的订阅。

## Operators

RxJS 中的操作符即函数，其中有两种操作符：

- 管道类操作符：可以使用 `observableInstance.pipe(operator())` 管道方法的操作符函数，它接受一个 Observable 并生成另一个 Observable
    
    ```jsx
    obs.pipe(
      op1(),
      op2(),
      op3(),
      op3(),
    )
    ```
    
- 创建类操作符：可以单独调用来创建一个新的 Observable
    
    ```jsx
    const observable = interval(1000 /* number of milliseconds */);
    ```
    

### 宝石图

![RxJS%20%E5%AD%A6%E4%B9%A0%2077e4aba53ab04c9594330a0dc988ebc3/marble-diagram-anatomy.svg](RxJS%20%E5%AD%A6%E4%B9%A0%2077e4aba53ab04c9594330a0dc988ebc3/marble-diagram-anatomy.svg)

宝石图中，时间从左到右流动，上图描述了每一块“宝石”是如何生产的。上面箭头右边的竖线表示 complete。中间的框指的是上面的输入如何进行处理的方法，即 Operator。下面箭头表示经过 Operator 操作后输出的 Observable。× 表示异常终止。

### 创建操作符

使用 pipe() 函数创建新的操作符：

```jsx
import { Observable } from 'rxjs';

function delay(delayInMillis) {
  return (observable) => new Observable(observer => {
    // this function will called each time this
    // Observable is subscribed to.
    const allTimerIDs = new Set();
    const subscription = observable.subscribe({
      next(value) {
        const timerID = setTimeout(() => {
          observer.next(value);
          allTimerIDs.delete(timerID);
        }, delayInMillis);
        allTimerIDs.add(timerID);
      },
      error(err) {
        observer.error(err);
      },
      complete() {
        observer.complete();
      }
    });
    // the return value is the teardown function,
    // which will be invoked when the new
    // Observable is unsubscribed from.
    return () => {
      subscription.unsubscribe();
      allTimerIDs.forEach(timerID => {
        clearTimeout(timerID);
      });
    }
  });
}
```

## Subject

RxJS Subject 是一种特殊的 Observable，它可以让生产的数据广播到多个 Observers。Subjects 类似于 EventEmitter，他们维护着多个注册的观察者。

Subject 既是 Observable 也是 Observer。也就是说 Subject 可以被 subscribe，也拥有 next、error 和 complete 回调函数。

由于 Subject 是 Observer，所以它可以作为 Observer 去订阅一个 Observable：

```jsx
import { Subject, from } from 'rxjs';

const subject = new Subject<number>();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});

const observable = from([1, 2, 3]);

observable.subscribe(subject); // You can subscribe providing a Subject
```

所以，Subjects 可以订阅 Observable，也可以被多个 Observer 订阅：

```jsx
import { from, Subject } from 'rxjs';
import { multicast } from 'rxjs/operators';

const source = from([1, 2, 3]);
const subject = new Subject();
// multicast 返回一个 ConnectableObservable，这是有着 connect() 方法的 Observable
const multicasted = source.pipe(multicast(subject));

// 底层是 `subject.subscribe({...})`:
multicasted.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
multicasted.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});

// 底层是 `source.subscribe(subject)`，这才是真正决定共享的 Observable 开始执行
multicasted.connect();
```

### 引用计数 ref counting

refCount() 方法可以让一个被共享的 Observable自动从第一个订阅关系开始执行，到最后一个订阅解除时停止。

### 特殊的 Subject

- BehaviorSubject
    
    主要用于时间上连续的值，比如一个人的年龄 Stream 可以用 BehaviorSubject 表示。例如一个 Observer 中途订阅了一个 BehaviorSubject，那么他会紧跟最近一次生产的数据而非下一此生产的数据（普通的 Subject 是从下一次生产数据开始）
    
    ```jsx
    // 传入的参数是初始值
    const subject = new BehaviorSubject(0); 
    ```
    
- ReplaySubject
    
    这种 Subject 会记录所有生产的旧值，在新的观察者订阅时，会将所有旧值重复发送一遍
    
    ```jsx
    // 传入的参数是缓存的旧值个数，例如新的订阅从5开始，那么新的订阅会收到3、4、5三个旧值
    const subject = new ReplaySubject(3); 
    ```
    
- AsyncSubject
    
    这种特殊的 Subject 只会将最后一次生产的数据发送给 Observer，无论新旧订阅都是这样
    
    ```jsx
    const subject = new AsyncSubject();
    ```
    

## Scheduler

Scheduler 能够控制一个订阅何时开始执行、数据何时生产。它有三个组件：

- 是一种数据结构：它知道如何存储、基于优先级来对任务进行排列
- 是一个执行上下文：表示任务在哪里、何时执行，比如立即执行、在 setTimeout 的回调中等
- 有一个虚拟时钟：由 now() 方法提供获取时间的方法

## Testing

## 常见 Observable

- of
    
    Each argument becomes a next notification.
    
    ```jsx
    of(10, 20, 30)
    .subscribe(
      next => console.log('next:', next),
      err => console.log('error:', err),
      () => console.log('the end'),
    );
    ```
    
- forkJoin
    
    Wait for Observables to complete and then combine last values they emitted.
    
    ```jsx
    const observable = forkJoin({
      foo: of(1, 2, 3, 4),
      bar: Promise.resolve(8),
      baz: timer(4000),
    });
    observable.subscribe({
     next: value => console.log(value),
     complete: () => console.log('This is how it ends!'),
    });
    // 可以传对象或数组，输出也会是对应的类型
    ```
    
- timer
    
    Its like interval, but you can specify when should the emissions start.
    
    ```jsx
    // 每一秒触发一次，开始有 3s 的 delay
    const numbers = timer(3000, 1000);
    numbers.subscribe(x => console.log(x));
    ```
    
- iif
    
    If statement for Observables
    
    ```jsx
    let subscribeToFirst;
    const firstOrSecond = iif(
      () => subscribeToFirst,
      of('first'),
      of('second'),
    );
    
    // 根据 subscribeToFirst 这个变量进行条件订阅
    subscribeToFirst = true;
    firstOrSecond.subscribe(value => console.log(value));
    
    subscribeToFirst = false;
    firstOrSecond.subscribe(value => console.log(value));
    ```
    
- ajax / ajax.func
    
    ```jsx
    const obs$ = ajax.getJSON(`https://api.github.com/users?per_page=5`).pipe(
      map(userResponse => console.log('users: ', userResponse)),
      catchError(error => {
        console.log('error: ', error);
        return of(error);
      })
    );
    
    const users = ajax({
      url: 'https://httpbin.org/delay/2',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'rxjs-custom-header': 'Rxjs'
      },
      body: {
        rxjs: 'Hello World!'
      }
    }).pipe(
      map(response => console.log('response: ', response)),
      catchError(error => {
        console.log('error: ', error);
        return of(error);
      })
    );
    ```
    
- fromEvent
    
    ```jsx
    const clicks = fromEvent(document, 'click'); // target, event
    clicks.subscribe(x => console.log(x));
    ```
    
- 

## 常见 Operators

- filter
    
    ```jsx
    const clicks = fromEvent(document, 'click');
    const clicksOnDivs = clicks.pipe(filter(ev => ev.target.tagName === 'DIV'));
    clicksOnDivs.subscribe(x => console.log(x));
    ```
    
- map
    
    将每次生产的数据进行修改
    
    ```jsx
    const clicks = fromEvent(document, 'click');
    const positions = clicks.pipe(map(ev => ev.clientX));
    positions.subscribe(x => console.log(x));
    ```
    
- takeUntil
    
    初始化两种 Observable，在第二种 Observable 触发之前一直触发第一种
    
    ```jsx
    const source = interval(1000);
    const clicks = fromEvent(document, 'click');
    const result = source.pipe(takeUntil(clicks));
    result.subscribe(x => console.log(x));
    // 0, 1, 2... till click on the DOM
    ```
    
- switchMap
    
    先将每个值生成对应的 Observable，然后将这些 Observable 展开
    
    ```jsx
    const switched = of(1, 2, 3).pipe(switchMap((x: number) => of(x, x ** 2, x ** 3)));
    switched.subscribe(x => console.log(x));
    // 1, 1, 1, 2, 4, 8
    ```
    
- tap
    
    可以对每一个生产的值进行处理，然后再返回原值（相当于轻轻触碰）
    
    ```jsx
    const clicks = fromEvent(document, 'click');
    const positions = clicks.pipe(
      tap(ev => console.log(ev)),
      map(ev => ev.clientX),
    );
    positions.subscribe(x => console.log(x));
    ```
    
- finalize
    
    在生产结束时执行传入的指定方法
    
- catchError
- startWith
- debounceTime