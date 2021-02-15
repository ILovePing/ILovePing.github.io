---
title: JS EventLoop
date: 2021-02-15 15:12:14
tags:
categories: 技术
---

EventLoop

- 浏览器端的是各浏览器厂商对 html5 里的 EventLoop 执行模型规范的实现。
- nodeJs 里的是基于 libuv 实现的。

MacroTask(task)宏任务

- setTimeout
- setInterval
- setImmediate（node only）
- requestAnimationFrame（web only）
- I/O
- UI rendering（web only）

MicroTask(Job)微任务

- process.nextTick(node only)
- Promise
- Object.observe
- MutationObserver

web 中 eventLoop 执行流程是： 1.同步的脚本执行完毕后，调用栈清空 2.执行 MicroTask 微任务队列，在此过程中可能会产生新的微任务，比如 promise.resolve().then()里面又产生一个 promise.then 那么新产生的微任务会放在微任务队列的末尾，也在本次执行周期中执行掉。 3.当微任务执行完毕，队列为空后，开始执行宏任务队列的队首的任务（**队首的任务即一个宏任务**），执行完毕后，开始循环回到第二步执行微任务的队列，如此 2，3 两步循环至两个队列和调用栈都为空。
例子：

```javascript
console.log(1);
setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
  });
});
new Promise((resolve, reject) => {
  console.log(4);
  resolve(5);
}).then((data) => {
  console.log(data);
  Promise.resolve()
    .then(() => {
      console.log(6);
    })
    .then(() => {
      console.log(7);
      setTimeout(() => {
        console.log(8);
      }, 0);
    });
});
setTimeout(() => {
  console.log(9);
});
console.log(10);
```

最终输出为：
1 4 10(第一次同步的执行完了)
5 6 7 （第一次微任务执行完了）
2（第一次宏任务执行完了）
3（第二次微任务执行完了）
9（第二次宏任务执行完了）
8（第三次宏任务执行完了）

![image](https://user-images.githubusercontent.com/13430709/63282738-10194600-c2e2-11e9-85f8-714f0898be6e.png)

node 中 eventLoop 就不同了：
话不多说，先上 node 官网的 eventloop 图：
![image](https://user-images.githubusercontent.com/13430709/63279019-e4df2880-c2da-11e9-83c1-844b38f0412a.png)
可以看到事件循环宏队列有六个阶段：

- timers 阶段：执行 setTimeout 和 setInterval 的回调
- I/O callback 阶段：执行一些系统调用错误，除了其他几个阶段中的 callback 之外的 callbak，比如 TCP 错误。
- idle, prepare 阶段：node 执行过程中的空闲时间,（联想到 window. requestIdleCallback）仅 node 内部使用
- poll 阶段：轮询，获取新的 I/O 事件，适当的条件下 node 将阻塞在这里
  poll 阶段主要两个功能：
  - 处理 poll 队列的事件
  - 当有已超时的 timer，执行它的回调函数
    问：如何理解第二点？答：当没有 setImmediate 设置过时 check 阶段就不会进去，那么就一直阻塞在 poll 阶段，这时候 poll 阶段会有一个检查机制。检查 timer 队列是否为空，如果 timer 队列非空，event loop 就开始下一轮事件循环，即重新进入到 timer 阶段。
- check 阶段：执行 setImmediate 的回调
  - setTimeout 和 setImmediate 的执行顺序是不确定的，依赖于当前主线程是否空闲或者机器上的其他任务。但是如果将两者放在 io 回调里面，即在 poll 阶段之后执行，那么直接进入 check 阶段，即 setImmediate 回调肯定会先执行，然后进入下一个 timmer 回调的执行。）
- close callbacks 阶段：执行 socket.on('close', ....)这些回调

概括一下，对于我们用户来说主要是四个宏任务队列：

1. timer queue
   2.io callback queue
   3.check queue
   4.close callback queue

与此同时，node 还有两个微任务队列：
1.precess.nextTick callback queue
2.other microtask queue(like promise)

具体的执行顺序是： 1.执行全局同步代码 2.按顺序（先 nextTick callback queue 然后 other microtask queue）执行两个微任务队列中的所有微任务，在此过程中新产生的微任务也会在在本次循环中继续执行知道两个微任务队列都为空，因此会出现所谓的 io starving 的情况（递归调用 process.nextTick）使当前微任务队列一直不为空，一直在执行，io callback 队列永远处于等待的情况，因此官方推荐使用 setImmediate，这样就不会一直被一次的微任务执行卡着。 3.开始执行 macrotask 宏任务，共 6 个阶段，从第 1 个阶段开始执行相应每一个阶段 macrotask 中的所有任务（这里是一个 queue 里的所有任务，同 web 端作区分，web 端是一个 queue 里的首个任务），一个阶段执行完毕之后回到第二步执行两个微任务队列，以此循环。
![image](https://user-images.githubusercontent.com/13430709/63282698-f972ef00-c2e1-11e9-97f2-8e5400ab4ee8.png)
