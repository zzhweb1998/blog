---
title: javascript执行机制
date: 2020-04-27 21:40:37
tags: ["js执行机制"]
top_img: /img/js执行机制/1.PNG
cover: /img/js执行机制/1.PNG
categories: 练习笔记
copyright: false
---

## 简介
JS的解析是由浏览器的JS解析引擎完成的。JS是单线程运行，换言之：同一个时间只做一件事，所有的任务都得排队，前面一个任务结束，后面一个任务才能开始。所以，当遇到很耗费时间的任务，比如I/O读写等，需要一种机制可以先执行后面的任务。这就有了同步和异步。
JS的执行机制就是一个主线程 + 一个任务队列。同步任务就是放在主线程上执行的任务，异步任务就是放在任务队列的任务。所有的同步任务都在主线程执行，这构成了一个执行栈，异步任务有了运行结果会在任务队列中放置一个事件，比如定时2秒，到2秒后才能放进任务队列（callback放进任务队列，而不是setTimeout函数放进队列）。脚本运行时，先依次运行执行栈，然后从队列中提取事件来运行任务队列中的任务，这个过程是不断重复的。所以叫事件循环（Event Loop）。

## 宏任务和微任务
除了广义的同步任务和异步任务，我们对任务有更精细的定义：宏任务和微任务。
- macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
- micro-task(微任务)：Promise，process.nextTick
事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。如图：
![](/img/js执行机制/1.PNG)

## 代码展示
```
console.log('1');  //事件循环1：整体script作为第一个宏任务进入主线程 1

setTimeout(function() {
    console.log('2');  //事件循环2：整体script作为第一个宏任务进入主线程 1

    process.nextTick(function() {
        console.log('3');  //事件循环2：微任务Event Queue 1：回调函数进入主线程 3
    })  //事件循环2：process.nextTick()，其回调函数被分发到微任务Event Queue 1

    new Promise(function(resolve) {
        console.log('4');  //事件循环2：整体script作为第一个宏任务进入主线程 2
        resolve();
    }).then(function() {
        console.log('5')  //事件循环2：微任务Event Queue 2：then进入主线程 4
    })  //事件循环2：promise，其then被分发到微任务Event Queue 2

})  //事件循环2：setTimeout，其回调函数被分发到宏任务Event Queue 1

process.nextTick(function() {
    console.log('6');  //微任务Event Queue 1：回调函数进入主线程 3
})  //事件循环1：process.nextTick()，其回调函数被分发到微任务Event Queue 1

new Promise(function(resolve) {
    console.log('7');  //事件循环1：整体script作为第一个宏任务进入主线程 2
    resolve();
}).then(function() {
    console.log('8')  //微任务Event Queue 2：then进入主线程 4
})  //事件循环1：promise，其then被分发到微任务Event Queue 2

setTimeout(function() {
    console.log('9');  //事件循环3：整体script作为第一个宏任务进入主线程 1

    process.nextTick(function() {
        console.log('10');  //事件循环3：微任务Event Queue 1：回调函数进入主线程 3
    })  //事件循环3：process.nextTick()，其回调函数被分发到微任务Event Queue 1

    new Promise(function(resolve) {
        console.log('11');  //事件循环3：整体script作为第一个宏任务进入主线程 2
        resolve();
    }).then(function() {
        console.log('12')  //事件循环3：微任务Event Queue 2：then进入主线程 3
    })  //事件循环3：promise，其then被分发到微任务Event Queue 2
})  //事件循环3：setTimeout，其回调函数被分发到宏任务Event Queue 2

/*
以上代码的执行循序为：
事件循环1：(主线程1、主线程 2)：(1、7)、(微任务Event Queue 1、微任务Event Queue 2)：(6、8)。
事件循环2：宏任务Event Queue 1：主线程1-2：(2、4)、微任务Event Queue 1、2：(3、5)。
事件循环3：宏任务Event Queue 2：主线程1-2：(9、11)、微任务Event Queue 1、2：(10、12)。
所以输出的值为：事件循环1、2、3：1、7、6、8、2、4、3、5、9、11、10、12。
*/
```

## 参考链接
详细请参考：https://juejin.im/post/59e85eebf265da430d571f89