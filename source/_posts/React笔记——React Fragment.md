---
title: React笔记——React Fragment
date: 2019-06-12 15:07:13
tags:
  - React
  - 笔记
categories: 前端
---

### 只支持单元素渲染

React 16 之前，子组件的渲染只支持单个元素，`render`函数的返回必须有一个根节点，通常使用一个`div`或`span`。

### 返回多元素数组

而在 React 16 之后，增加了对组件的`render`方法返回一个包含元素的数组的支持。你可以将其他们放进数组里，而不用将子元素包装在一个 DOM 元素内：

```js
render(){
    return [
    "text",
    <h2>Head2</h2>,
    "text",
    <h3>Head3</h3>
    ]
}
```

### Fragments

React 16 提供了`Fragment`组件来替代数组的写法，提供一致性的 JSX 开发

```
render(){
    return(
        <Fragment>
            Text.
            <h2>Head2</h2>
            Text.
            <h3>Head3</h3>
        </Fragment>
    )
}
```

### 语法棒简写`<></>`

JSX 增加了`fragment`的语法支持，可简写为：

```js
render(){
    return(
        <>
            Text.
            <h2>Head2</h2>
            Text.
            <h3>Head3</h3>
        </>
    )
}
```

其中`<></>`是`<React.Fragment />`的语法糖
