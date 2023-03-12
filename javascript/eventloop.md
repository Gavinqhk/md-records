# js的执行机制

JavaScript的任务分为两种，1、同步任务，2、异步任务。
事件循环中涉及到js执行栈，任务队列。

js任务调用执行都要进入执行栈。  
同步任务进入执行栈执行，执行完出栈，后进先出。
异步任务分为宏任务，微任务，  
任务队列分为宏任务队列（macroQueues），微任务队列（microQueues）  

![eventloop](https://user-gold-cdn.xitu.io/2019/9/29/16d7ace2eda820a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
执行顺序：

1. 执行js从顶部向下同步执行，同步任务压入js执行栈，遇到异步任务会进入Event Table并注册函数，
2. 当指定当事件执行完成之后，event table 的函数，推送到任务队列Event Queue
3. 当主线程执行完成（js执行栈的任务执行完），检查微任务队列（micro Queue）是否有任务等待执行，有则一次压入js执行栈执行，无则查看宏任务队列（macroQueue）是否有任务执行，有则执行，无则执行完成。
4. 当异步任务入栈执行时，可能又会有宏任务，微任务，又会循环以上当操作执行顺序，当宏任务，微任务队列为空，执行栈为空，则说明任务完全执行完。

异步任务包括：
宏任务：

- setTimeout
- setInterval
- setImmediate
- I/O
- UI rendering

微任务：

- Promise的会调函数
- process.nextTick()
- Object.observe()
- MutationObserver

test试一试
1、

```js
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});
// browser: script start, promise1, promise2, setTimeout
// node: script start, promise1, promise2, setTimeout
```

2、

```js
console.log('1');
async function async1() {
    console.log('2');
    await async2();
    console.log('3');
}
async function async2() {
    console.log('4');
}
// node
process.nextTick(function() {
    console.log('5');
})

setTimeout(function() {
    console.log('6');
    //node
    process.nextTick(function() {
        console.log('7');
    })
    new Promise(function(resolve) {
        console.log('8');
        resolve();
    }).then(function() {
        console.log('9')
    })
})

async1();

new Promise(function(resolve) {
    console.log('10');
    resolve();
}).then(function() {
    console.log('11');
});
console.log('12');

// browser: 1, 2, 4, 10, 12, 3, 11, 6, 8, 9 
// node: 1, 2, 4, 10, 12, 5, 11, 3, 6, 8, 7， 9

// 知识点
/**  搞清楚async await 以及promise对原理。
 * 1、process.nextTick()执行会在当前任务执行完之后立即执行（不管宏任务微任务队列是否有任务），执行后继续事件循环流程。
 * 2、async await 返回的是promise
 * 3、同步执行完await的语句，之后的语句进入微任务队列。
*/
```

3、

```js
console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})
// browser: 1, 7, 8, 2, 4, 5, 9, 11, 12
// node: 1, 7, 6, 8, 2, 4, 3, 5, 9, 11, 10, 12（不一定对）
```

<https://segmentfault.com/a/1190000019494012>
