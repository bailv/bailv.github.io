---
title: '笔记:利用记忆化技术提升React性能'
date: 2019-10-09 17:13:25
tags:
    - 笔记
    - 前端
categories:
    - javascript
    - React
---

## 记忆化技术

望文生义，记忆化就是把函数的调用结果记录下来或者缓存下来。下次调用这个函数的时候，如输入参数和上一次完全一致，则无需再次计算，直接返回上一次的结果。

## 使用动机

React项目中，有一个场景很常见：从api请求拿到一个复杂的数据结构，该数据一般通过React组件的`props`传入组件。而当前组件只需展示部分的数据，因此在`render`的时候需对该数据进行处理，如过滤一些无用的信息或者重新组合该数据结构。

而我们知道`state`和`props`频繁修改会触发`render`。有时候组件的更新并不是因为重新请求api返回的数据结构导致变化，但对数据的处理逻辑都是在`render`中，因此不可避免每次`render`都对数据进行一次无用的处理，造成性能浪费。

## memoize-one

借助memoize-one这个库，我们可以实现本来需要 O(n) ，O(n2) 甚至更高复杂度的算法，我们瞬间可以以 O(1) 的效率把结果直接从缓存中读取出来。

memoize-one是采用闭包来缓存数据的，其源码如下：
```js
type EqualityFn = (a: mixed, b: mixed) => boolean;

const simpleIsEqual: EqualityFn = (a: mixed, b: mixed): boolean => a === b;

export default function <ResultFn: (...Array<any>) => mixed>(resultFn: ResultFn, isEqual?: EqualityFn = simpleIsEqual): ResultFn {
  let lastThis: mixed; // 用来缓存上一次result函数对象
  let lastArgs: Array<mixed> = []; // 用来缓存上一次的传参
  let lastResult: mixed; // 用来缓存上一次的结果
  let calledOnce: boolean = false; // 是否之前调用过
  // 判断两次调用的时候的参数是否相等
  // 这里的 `isEqual` 是一个抽象函数，用来判断两个值是否相等
  const isNewArgEqualToLast = (newArg: mixed, index: number): boolean => isEqual(newArg, lastArgs[index]);

  const result = function (...newArgs: Array<mixed>) {
    if (calledOnce &&
      lastThis === this &&
      newArgs.length === lastArgs.length &&
      newArgs.every(isNewArgEqualToLast)) {
      // 返回之前的结果
      return lastResult;
    }

    calledOnce = true; // 标记已经调用过
    lastThis = this; // 重新缓存result对象
    lastArgs = newArgs; // 重新缓存参数
    lastResult = resultFn.apply(this, newArgs); // 重新缓存结果
    return lastResult; // 返回新的结果
  };

  // 返回闭包函数
  return (result: any);
}
```

### isEqual参数

一般两个对象比较是否相等，我们不能用`===`或者`==`来处理，memoize-one允许用户自定义传入判断是否相等的函数。官方推荐使用`lodash.isEqual`

## React Hook useMemo

React 16.8引入Hook。其中官方提供了原生的记忆化API——`useMemo`。
具体的使用方法参考[官方文档](https://reactjs.org/docs/hooks-reference.html#usememo)。

## 参考链接

[记忆化技术介绍——使用闭包提升你的 React 性能](https://zhuanlan.zhihu.com/p/37913276)
[React优化-记忆化技术-使用闭包提升你的React性能](https://segmentfault.com/a/1190000015301672)