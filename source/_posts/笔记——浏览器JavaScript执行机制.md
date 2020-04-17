---
title: "笔记——浏览器JavaScript执行机制"
date: 2019-08-13 18:17:01
tags:
  - 笔记
  - 前端
categories: javascript
---

## 单线程的 JavaScript

JS 被设计为单线程运行的，这是因为 JS 主要用来实现很多交互相关的操作，如 DOM 相关操作，如果是多线程会造成复杂的同步问题。

因此浏览器在运行时只开启了一个 JS 引擎线程来解析和执行 JS。那为什么只有一个引擎呢？如果同时有两个线程去操作 DOM，浏览器是不是又要不知所措了。

> JS 引擎可以说是 JS 虚拟机，负责 JS 代码的解析和执行。通常包括以下几个步骤：
>
> - 词法分析：将源代码分解为有意义的分词
> - 语法分析：用语法分析器将分词解析成语法树
> - 代码生成：生成机器能运行的代码
> - 代码执行

虽然 JavaScript 是单线程的，可是浏览器内部不是单线程的。你的一些 I/O 操作、定时器的计时和事件监听（click, keydown...）等都是由浏览器提供的其他线程来完成的。

> 一个浏览器通常由以下几个常驻的线程
>
> - 渲染引擎线程：顾名思义，该线程负责页面的渲染
> - JS 引擎线程：负责 JS 的解析和执行
> - 定时触发器线程：处理定时事件，比如 setTimeout, setInterval
> - 事件触发线程：处理 DOM 事件
> - 异步 http 请求线程：处理 http 请求

## 任务队列

单线程就意味着，所有任务需要排队。任务队列中，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等着。因此一些耗时的操作如 I/O，http 请求等任务可以先挂起，主线程先运行排在后面的任务。等到 IO 设备返回了结果，再回过头，把挂起的任务继续执行下去。

### JavaScript 中将任务分为两种

#### 同步任务

在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务

#### 异步任务

不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。异步执行的运行机制如下：

1. 所有同步任务都在主线程上执行，形成一个**执行栈**（execution context stack）。
2. 主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件
3. 一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
4. 主线程不断重复上面的第三步。

![](https://raw.githubusercontent.com/clanaid/pic_repository/master/img/bg2014100801.jpg)

### 任务规范

在任务队列中存放的是一个个`任务(Task)`。规范中规定，**Task 分为两大类, 分别是 Macro Task(宏任务) 和 Micro Task(微任务)**，并且每个 Macro Task 结束后, 都要清空所有的 Micro Task. 其中 Micro Task 细分为两个队列——`Micro Task Queue`() 和 `Tick Task Queue`(专门用于存放 process.nextTick 的任务). `Tick Task`任务优先于`Micro Task`任务执行

#### Macro Task

每次执行栈执行的代码就是一个宏任务（包括每次从事件队列中获取一个事件回调并放到执行栈中执行）

每一个 task 会从头到尾将这个任务执行完毕，不会执行其它
浏览器为了能够使得 JS 内部 task 与 DOM 任务能够有序的执行，会在一个 task 执行结束后，在下一个 task 执行开始前，对页面进行重新渲染

包含：

- setImmediate
- setTimeout
- setInterval

#### Micro Task

所有微任务都是添加到微任务队列（Job Queues）中，等待当前 macrotask 执行完毕后执行，而这个队列由 JS 引擎线程维护

包含：

- process.nextTick
- Promise
- Object.observe
- MutaionObserver

![](https://raw.githubusercontent.com/clanaid/pic_repository/master/img/20190813164356.png)

## Event Loop

从上述任务队列中可以知道，主线程从任务队列中读取任务是不断循环的，每次栈被清空后，都会在消息队列中读取新的任务，如果没有新的任务，就会等待，直到有新的任务，这一机制称为**事件循环**。

主线程运行的时候，产生`堆（heap）`和`栈（stack）`。

栈存放大量的同步任务，同时如变量和函数的初始化、事件的绑定等等那些不需要回调函数的操作都可归为这一类。

堆用来存储声明的变量、对象。

某个异步任务有了响应就会被推入 callback 队列中。如用户的点击事件、浏览器收到服务的响应和 setTimeout 中待执行的事件，每个异步任务都和回调函数相关联。

主线程执行栈中的同步任务，当所有同步任务执行完毕后，栈被清空，然后读取消息队列中的一个待处理任务，并把相关回调函数压入栈中，单线程开始执行新的同步任务。

![](https://raw.githubusercontent.com/clanaid/pic_repository/master/img/20190813172912.png)

执行流程如图所示：

![](https://raw.githubusercontent.com/clanaid/pic_repository/master/img/20190813171334.png)

## 引用

**参考文章**

[JavaScript 异步机制详解](https://juejin.im/post/5a6ad46ef265da3e513352c8)

[JavaScript 中的 JS 引擎的执行机制：探究 Event Loop](https://zhuanlan.zhihu.com/p/33104163)

[JavaScript 单线程和异步机制](https://github.com/pramper/Blog/issues/4)

[JavaScript 运行机制详解：再谈 Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
