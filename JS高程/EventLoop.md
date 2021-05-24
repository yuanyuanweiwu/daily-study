

# Event Loop

首先Event Loop 是什么？简单来讲就是为了实现异步的一种机制。Event Loop 分为两种，一种存在于 [Browsing Context](https://link.zhihu.com/?target=https%3A//html.spec.whatwg.org/multipage/browsers.html%23browsing-context) 中，还有一种在 [Worker](https://link.zhihu.com/?target=https%3A//html.spec.whatwg.org/multipage/workers.html%23workers) 中。这里主要是对浏览器中的js环境的Event Loop进行总结

在JavaScript中，任务分为两种任务。一是宏任务  `MacroTask`也叫`Task`，一种是微任务 `MicroTask`

- MacroTask

  script的全部代码，setTimeout、setInterval、setImmediate、I\O、UI Rendering

- MicroTask

  Procsee.nextTick(node)、promise、MutationObserver

在JavaScript单线程运行机制中分为同步和异步任务，同步任务在等待主线程依次执行，异步任务在异步有了结果后，将注册的回调函数放入任务队列中等待主线程空闲的时候，再被读取到栈等待主线程操作。如下图所示

![](images\Event Loop.png)

------

以下还有一张浏览器EventLoop执行图，将对其进行一个JavaScript代码的执行流程分析

![](images\Brow_eventloop.png)

1. 执行全局script同步代码。
2. 全局script代码执行完后，调用栈清空，检查微任务队列是否为空，为空，取宏任务队首任务进行执行。不为空，此时跳转微任务执行步骤。
3. 从微任务队列microtask queue取出位于队首的回调任务，放入调用栈执行，重复此操作，将微任务队列中的所有任务都执行完。
4. 微任务队列此时执行完毕，调用栈为空，取出宏任务队列macrotask queue中队首任务，放入调用栈执行。
5. 执行完后调用栈清空，重复234操作。

------

由上总结后，我们对下面的代码进行分步解析。

```javascript
console.log('script start')
setTimeout(function(){
    console.log('setTimeout')
},0)
Promise.resolve().then(function(){
    console.log('promise1')
}).then(function(){
    console.log('promise2')
})
console.log('script end')

//script start
//script end
//promise1
//promise2
//setTimeout
```

1. ```javascript
   console.log('script start')
   ```

   | Macrotask  | run script     |
   | ---------- | -------------- |
   | Microtasks | []             |
   | stack      | script         |
   | log        | 'script start' |

2. ```javascript
   setTimeout(function(){
       console.log('setTimeout')
   },0)
   //settimeout的callback放入宏任务队列
   ```

   | Macrotask  | run script  setTimeout callback |
   | ---------- | ------------------------------- |
   | Microtasks | []                              |
   | stack      | script                          |
   | log        | 'script start'                  |

3. ```javascript
   Promise.resolve().then(function(){
       console.log('promise1')
   }).then(function(){
       console.log('promise2')
   })
   //promise.then属于微任务队列，在一个微任务结束之后如果紧接着有下一个，马上塞入微任务队列进行排队执行
   ```

   | Macrotask  | run script  setTimeout callback                       |
   | ---------- | ----------------------------------------------------- |
   | Microtasks | `promise.then` `promise.then`                         |
   | stack      | `promise callback` `promise callback`                 |
   | log        | 'script start'   'script end'  'promise1'  'promise2' |



4. | Macrotask  | run script  setTimeout callback                       |
   | ---------- | ----------------------------------------------------- |
   | Microtasks | 此时微任务执行完，可能现在浏览器需要执行ui rendering  |
   | stack      |                                                       |
   | log        | 'script start'   'script end'  'promise1'  'promise2' |



5. 此时微任务队列为空，取宏任务队列的队首任务进行执行

   | Macrotask  | setTimeout callback                                   |
   | ---------- | ----------------------------------------------------- |
   | Microtasks | 此时微任务执行完，可能现在浏览器需要执行ui rendering  |
   | stack      |                                                       |
   | log        | 'script start'   'script end'  'promise1'  'promise2' |

6.  此时按照先进先出原则，宏任务走完，最后清空队列

| Macrotask  | setTimeout callback=>[]                                      |
| ---------- | ------------------------------------------------------------ |
| Microtasks |                                                              |
| stack      | setTimeout callback=>[]                                      |
| log        | 'script start'   'script end'  'promise1'  'promise2' 'setTimeout' |

------

再来一道例题：

```javascript
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3)
  });
});

new Promise((resolve, reject) => {
  console.log(4)
  resolve(5)
}).then((data) => {
  console.log(data);
  
  Promise.resolve().then(() => {
    console.log(6)
  }).then(() => {
    console.log(7)
    
    setTimeout(() => {
      console.log(8)
    }, 0);
  });
})

setTimeout(() => {
  console.log(9);
})

console.log(10);
```

结果为：

```javascript
1
4
10
5
6
7
2
3
9
8
```



总结部分来参考自以下文章：

> <https://segmentfault.com/a/1190000016278115#item-3>
>
> <https://juejin.im/post/5c3d8956e51d4511dc72c200#heading-15>