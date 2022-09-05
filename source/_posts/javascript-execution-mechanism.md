---
title: JavaScript 执行机制
tags: 
  - 同步与异步
  - 宏任务与微任务
categories: JavaScript
abbrlink: 2616
date: 2019-07-02 23:00:44
---

`JavaScript`是一门单线程语言，在最新的`HTML5`中提出了`Web-Worker`，但`JavaScript`是单线程这一核心仍未改变。所以一切`JavaScript`版的“多线程”都是用单线程模拟出来的。

> 所谓单线程，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。

<!-- more -->

这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等待，会拖延整个程序的执行。

## 同步与异步

为了解决这个问题，`JavaScript`语言将任务的执行模式分成两种：在主线程上执行的任务`"同步任务"`，被主线程挂载起来的任务`"异步任务"`，后者一般是放在一个叫`任务队列`的数据结构中。

“同步模式”就是后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的；

“异步模式”则完全不同，每一个任务有一个或多个回调函数，前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行与任务的排列顺序是与不一致的、异步的。在浏览器端，耗时很长的操作都应该是异步执行，避免浏览器失去响应。

## 事件循环

一般异步执行运行机制如下：

- 所有同步任务都在主线程上执行，形成一个执行栈，异步的进入事件表`(Event Table)`并注册函数
- 主线程之外，还有一个“任务队列”`(Event Queue)`，当指定的事情完成时，`Event Table`会将这个函数移入`Event Queue`
- 直到“执行栈”中的所有同步任务执行完毕，就会读取`Event Queue`对应的函数，进入主线程执行
- 上述过程会不断重复，也就是常说的事件循环`(Event Loop)`

## 定时器

定时器主要由`setTimeout()`和`setInterval()`两个函数来完成，它们内部运行机制完全不一样，不同的只是，前者一次性执行，后者反复执行。定时器，属于任务队列中的异步任务。

常见问题，如以下代码：

```javascript
setTimeout(()=>{
    task()
},3000)

sleep(10000)
```

以上代码执行`task()`需要的时间超过3秒，原因：

- `task()`进入`Event Table`并注册，计时开始
- 执行`sleep`函数，耗时非常多，计时仍在继续
- 3秒到了，计时事件`timeout`完成，`task()`进入`Event Queue`，但是`sleep`仍没有执行完成，只能继续等待
- 直到`sleep`执行完成，`task()`从`Event Queue`进入主线程执行

> 补充：即便`setTimeout`设置为0毫秒，也同样为异步执行，也同样需要主线程任务执行完成，即使主线程为空，0毫秒实际上也是达不到的，根据`HTML`的标准，最低是4毫秒。

## 宏任务与微任务

- 宏任务`(macro-task)`：包括整体代码`script`，`setTimeout`，`setInterval`，`setImmediate`
- 微任务`(micro-task)`：`Promise`，`process.nextTick`

微任务队列与宏任务队列是相互独立的，不同类型的任务会进入对应的`Event Queue`，比如`setTimeout`和`setInterval`会进入相同的`Event Queue`。事件循环的顺序，决定`JS`代码的执行顺序。进入整体代码（宏任务）后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，直到其中一个任务队列执行完毕，再执行所有的微任务。

举个栗子：

```javascript
console.log('script start');
Promise.resolve().then(()=>{
    setTimeout(()=>{
        console.log('setTimeout1')
    },0);
}).then(()=>{
    console.log('promise1');
});

setTimeout(()=>{
    console.log('setTimeout2');
    Promise.resolve().then(()=>{
        console.log('promise2');
    })
},0);
console.log('script end');

/* 谷歌浏览器控制台输出结果 */
// script start
// script end
// promise1
// setTimeout2
// promise2
// setTimeout1
```

分析执行步骤：

1. 进入宏任务，执行当前代码，输出 `script start`
2. 执行 `Promise.resolve(1)`，回调（内含函数 `setTimeout(1)`）进入微任务
3. 执行`setTimeout(2)`，回调 （内含函数 `Promise.resolve(2)`）进入下一个宏任务
4. 输出 `script end`，当前宏任务结束
5. 宏任务结束，查看微任务队列，当前微任务是`Promise.resolve(1)`回调，执行回调里面的`setTimeout(1)`，定时器回调进入宏任务，接着执行微任务队列里下一个任务，输出`promise1`，当前微任务为空
6. 当前微任务为空，执行下一宏任务。当前宏任务是`setTimeout(2)`回调，输出`setTimeout2`，执行回调里面的`Promise.resolve(2)`，回调进入微任务，当前宏任务结束
7. 当前宏任务结束，执行微任务。当前微任务是`Promise.resolve(2)`回调，输出`promise2`，微任务为空
8. 执行下一个宏任务，`setTimeout(1)`回调输出 `setTimeout1`

以上仅对谷歌浏览器进行执行分析，不同浏览器可能会从性能优化方面做一些处理从而导致输出的结果会不一样，这里不做分析。

## 总结

- `JavaScript`是一门单线程语言
- 事件循环是`js`实现异步的一种方法，也是`js`的执行机制
- 宏任务按顺序执行，且浏览器在每个宏任务之间渲染页面
- 所有微任务也按顺序执行，且在以下场景会立即执行所有微任务
&ensp;&ensp; -- 每个回调之后且`js`执行栈为空
&ensp;&ensp; -- 每个宏任务结束后

参考：[JS事件循环机制（event loop）之宏任务、微任务](<https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/?utm_source=html5weekly>)、阮一峰老师的JavaScript运行机制详解

