---
layout:     post
title:      "弄懂Vue中nextTick执行顺序"
date:       2021-09-23 21:18:00
author:     "Mountain"
header-img: "img/post-bg-2015.jpg"
tags:
    - 前端综合
---

## Vue中的nextTick官方说明

在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

```
// 修改数据
vm.msg = 'Hello'
// DOM 还没有更新
Vue.nextTick(function () {
  // DOM 更新了
})

// 作为一个 Promise 使用 (2.1.0 起新增，详见接下来的提示)
Vue.nextTick()
  .then(function () {
    // DOM 更新了
  })
```

## 原理

Vue（2.6+） 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver ，如果执行环境不支持，则会采用setImmediate和setTimeout(fn, 0) 代替，前两者微任务，后两者宏任务。

## 事件循环中执行

Vue中，this.a="22222",数据被修改，简易版流程（微任务版）：

1. a被修改；
2. 触发a的setter，通知订阅了a属性的所有Watcher；
3. Watcher收到通知，把该Watcher放到更新数组中；（过滤掉已存在数组中的相同Watcher，保证同一个的Watcher在同一个tick只被执行一次）
4. 执行其他同步代码；
5. 遍历第三步收集到的更新数组，执行里面所有Watcher的更新Dom方法；
6. 执行nextTick产生的微任务，此时就能获取到更新后的Dom；
7. 渲染Dom，进入下一个任务队列；

![nextTick在事件循环中执行](D:\pros\blog\m-blog\img\in-post\nextTick\nextTick演示.webp)

测试代码：

```
<i id="a">{{a}}</i>
data() {
    return {
      a: 11
    }
  },
mounted() {
    let time_start = Date.now()
    consoleData('初始状态')
    // 两秒后修改a的值
    setTimeout(() => {
      console.log('（2s后修改a的值）')
      this.a = 22222
      consoleData('修改后马上获取')
      syncAwait()
      this.$nextTick(function () {
        consoleData('nextTick里获取')
        console.log('（此时更新了Dom，但视图未渲染）')
        syncAwait()
        consoleData('执行完nextTick')
        console.log('（执行完该任务队列，渲染视图）')
      })
    }, 2000)
    // 打印元素的内容与宽度
    function consoleData(title) {
      console.log(`${Date.now() - time_start}ms,${title}:值为${$('#a').text()},宽度为${$('#a').width()}`);
    }
    // 模拟两秒的同步代码执行
    function syncAwait(time = 2000) {
      console.log('执行2s的同步代码')
      const startTime = Date.now();
      while (true) {
        if (Date.now() - startTime > time) { break; }
      }
    }
},
  
 0ms,初始状态:值为11,宽度为16
（2s后修改a的值）
 2006ms,修改后马上获取:值为11,宽度为16
 执行2s的同步代码
 4008ms,nextTick里获取:值为22222,宽度为42
（此时更新了Dom，但视图未渲染）
 执行2s的同步代码
 6010ms,执行完nextTick:值为22222,宽度为42
 （执行完该任务队列，渲染视图）

```

**注意！！！**此时dom更新了但未渲染

![dom更新了但没渲染](D:\pros\blog\m-blog\img\in-post\nextTick\dom更新了但没渲染.webp)



## 其他

#### 微任务与宏任务

- 常见的微任务：Promise、MutationObserver、Object.observe(废弃)，以及nodejs中的process.nextTick；
- 常见的宏任务：setTimeout、setInterval、setImmediate、I/O；

#### setImmediate

- 该方法可能不会被批准成为标准，目前只有最新版本的 Internet Explorer 和Node.js 0.10+实现了该方法。它遇到了 Gecko(Firefox) 和Webkit (Google/Apple) 的阻力；
- node.js里面setTimeout(fn, 0)会被强制改为setTimeout(fn, 1)，HTML 5里面setTimeout最小的时间限制是4ms；

#### MutationObserver

- 会在指定的DOM发生变化时被调用；
- vue用的不是MutationObserver的DOM监听功能，而是用其生成微任务；

#### Vue nextTick 实现顺序修改历史

- 第一版的 nextTick 实现顺序为 MutationObserver, setImmediate,setTimeout
- 第一次修改是将 MutationObserver 替换为 postMessage
- 第二次修改顺序变为 Promise, MutationObserver, setTimeout.
- 2.5.2+ Promise，setImmediate, Messagechannel, setTimeout
- 2.6+ Promise、MutationObserver、setImmediate、setTimeout



## 参考

[vue-$nextTick浅析](https://www.jianshu.com/u/7fcd83fb01b5)
[vue nextTick原理](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F6880710529355251726)
[Vue.nextTick，了解一下？](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F6844904000542736398%23heading-0)
[Vue nextTick 变迁史](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F6844903876584275976)
[setTimeout和setImmediate到底谁先执行，本文让你彻底理解Event Loop](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F6844904100195205133)

