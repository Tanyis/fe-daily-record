## JavaScript 语言特点

1. 单线程，而这个线程中拥有唯一的一个事件循环。（web worker 这里不参与讨论）
2. 代码的执行过程中，除了依靠函数调用栈来搞定函数的执行顺序外，还依靠任务队列(task queue)来搞定另外一些代码的执行。
3. 一个线程中，事件循环是唯一的，但是任务队列可以拥有多个。
4. 任务队列又分为 macro-task（宏任务）与 micro-task（微任务），在最新标准中，它们被分别称为 task 与 jobs 。
5. setTimeout/Promise 等我们称之为任务源。而进入任务队列的是他们指定的具体执行任务。
```javascript
// setTimeout 中的回调函数才是进入任务队列的任务
setTimeout(function() {
  console.log('xxxx');
})
```
6. 来自不同任务源的任务会进入到不同的任务队列。其中 setTimeout 与 setInterval 是同源的。
7. 事件循环的顺序，决定了 JavaScript 代码的执行顺序。它从 script (整体代码)开始第一次循环。之后全局上下文进入函数调用栈。直到调用栈清空(只剩全局)，然后执行所有的 micro-task 。当所有可执行的 micro-task 执行完毕之后。循环再次从 macro-task 开始，找到其中一个任务队列执行完毕，然后再执行所有的 micro-task，这样一直循环下去。
8. 其中每一个任务的执行，无论是 macro-task 还是 micro-task，都是借助函数调用栈来完成。

> ![](https://raw.githubusercontent.com/xuqiang521/fe-daily-record/master/javascript/event-loop/EventLoop.jpg)

## macrotask 与 microtask 类别具体分类

```js
// macrotasks
script(整体代码), setImmediate, setTimeout, setInterval, I/O, UI rendering

// microtasks
process.nextTick, Promises, Object.observe, MutationObserver
```

## DEMO

```javascript
setImmediate(function () {
    console.log(1);
}, 0);
setTimeout(function () {
    console.log(2);
}, 0);
new Promise(function (resolve) {
    console.log(3);
    resolve();
    console.log(4);
}).then(function () {
    console.log(5);
});
console.log(6);
process.nextTick(function () {
    console.log(7);
});
console.log(8);
// 3 4 6 8 7 5 1 2
```

## 执行过程如下：

1. JavaScript引擎首先会从macrotask queue中取出第一个任务，即 script (整段代码) 
2. 执行完毕后，将microtask queue中的所有任务取出，按顺序全部执；
3. 然后再从macrotask queue中取下一个，
执行完毕后，
4. 再次将microtask queue中的全部取出；
循环往复，直到两个queue中的任务都取完。

## 解释：
1. 代码开始执行时，所有这些代码在 `macrotask queue` 中，取出来执行之。
2. 后面遇到了 `setTimeout`，又加入到`macrotask queue`中，
3. 然后，遇到了 `promise.then`，放入到了另一个队列 `microtask queue`。
4. 整个`execution context stack` 执行完后，
5. 取 `microtask queue` 中的任务了。

因此 `promise.then` 的回调比 `setTimeout` 先执行。

## 参考

[Event loops](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)

**队列执行顺序**

> ![](https://raw.githubusercontent.com/xuqiang521/fe-daily-record/master/javascript/event-loop/FIFO.png)

## question

**Q：`process.nextTick` 也会放入 `microtask quque`，为什么优先级比 `promise.then` 高呢？**

**A：process.nextTick 永远大于 promise.then** 
在 Node 中，_tickCallback 在每一次执行完 TaskQueue 中的一个任务后被调用，而这个 _tickCallback 中实质上干了两件事：
1. nextTickQueue中所有任务执行掉(长度最大1e4，Node版本v6.9.1)
2. 第一步执行完后执行 `_runMicrotasks`函数，执行 `microtask` 中的部分(`promise.then` 注册的回调)所以很明显 `process.nextTick > promise.then`”
。