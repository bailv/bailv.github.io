---
title: '笔记——Web Components'
date: 2019-10-25 14:23:41
tags:
  - 笔记
  - 前端
categories:
  - html
---

## 四个核心

HTML 和 DOM 标准定义了四种新的标准来帮助定义 Web Component。这些标准如下：

- 自定义元素（Custom Elements）：定义新 HTML 元素的一系列 API；
- 影子 DOM（Shadow DOM）：组合对 DOM 和样式的封装；
- HTML 导入（HTML Imports）：定义在文档中导入其他 HTML 文档的方式；
- HTML 模板（HTML Templates）：HTML 内的 DOM 模板，在<template>元素内声明。

## 自定义元素

自定义元素支持开发者定义一类新 HTML 元素，声明其行为和样式，自定义元素分两类：

- 自定义标签元素（Autonomous custom elements）：完全独立于原始 HTML 元素标签的新标签元素，其所有行为需要开发者定义；
- 自定义内置元素（Customized built-in）：基于 HTML 原始元素标签的自定义元素，以便于使用原始元素的特性，开发者只需要定义拓展行为；

### 自定义标签元素

创建一个自定义标签元素，我们需要继承 HTMLELement 类
自定义元素具有以下生命周期回调函数：

- `connectedCallback` — 每当元素插入 DOM 时被触发。
- `disconnectedCallback` — 每当元素从 DOM 中移除时被触发。
- `attributeChangedCallback` — 当元素上的属性被添加、移除、更新或取代时被触发。

```js
class TestElement extends HTMLElement {
  constructor() {
    super();
  }
}
//注册自定义元素
customElements.define('test-ele', TestElement);
```

在需要使用该组件的页面只需像使用正常 HTML 元素一样：

```html
<test-ele>Test</test-ele>
```

### 自定义内置元素

很多时候我们并不需要完全创建一个新元素，而只是需要在某些内置元素基础上进行拓展，创建自定义内置元素，需要继承该类元素类，如 HTMLButtonElement 或 HTMLDivElement：

```js
class MenuButton extends HTMLButtonElement {
        ...
    }
customElements.define('menu-button', MenuButton);
```

使用也很简单，和内置元素一样的语法；不同的是，在需要使用自定义内置元素时，为内置元素添加 is 特性，该特性值对应创建的自定义内置元素名称:

```html
<button is="menu-button">menu</button>
```

### 对比

通过上面实例可知，自定义标签元素与内置元素主要表现在两点不同：

- 标签：自定义标签元素是完全独立的一个新元素，新标签，而自定义内置标签，使用的依然是已有内置标签；
- 行为与样式：自定义内置元素，继承内置元素的默认行为，样式及语义，可以进行拓展，而自定义标签元素，完全需要开发者定义相关声明。

## Shadow DOM

文档 DOM 树的层次结构中是不存在局部作用域概念的，也就是说文档内所有定义的样式都对整个文档产生影响，文档中的样式也会影响组件内的声明样式，而不限定于元素所处位置，这样显然极大阻碍了组件的独立性和可重用性

影子 DOM API 提供了`attachShadow()`方法，创建一个影子 DOM，支持将封装的内容或组件作为一个独立 DOM 子树附加进一个 HTML 文档，组件内与外部隔离，样式互不影响，这也印证了组件开发的封装性需求。

### 影子树（SHADOW TREE）与影子主体（SHADOW HOST）

使用`attachShadow()`方法创建的元素就是一个影子 DOM，而其子内容就构成一棵影子树（shadow tree），而和影子 DOM 绑定，也就是包含该树的文档内元素通常称为影子主体（shadow host）。

### 创建使用

使用 Shadow DOM，我们需要在一个元素上创建一个影子主体，然后将模板内文档添加到这个主体上即可。

```js
var frag = document.createElement('div');
var shadowRoot = frag.attachShadow({ mode: 'open' });
shadowRoot.innerHTML = '<p>Shadow DOM Content</p>';
```

### 槽位（SLOT）

当一个元素（即影子主体）内存在影子 DOM，浏览器默认只会渲染该影子 DOM 的影子树，而不渲染影子主体的其他子内容。
如果要保存子内容，需要使用`<slot>`槽位元素，相当于做一个占位符

eg.
给 menu 元素绑定子 DOM:

```js
var menus = document.querySelector('.menus');
var shadowRoot = menus.attachShadow({ mode: 'open' });
shadowRoot.innerHTML =
  '<ul>\
        <li>Home</li>\
        <li>About</li>\
        </ul>';
```

```html
<div class="menus">
  <h2>Menus</h2>
  <slot></slot>
</div>

//渲染结果,slot被替代
<div class="menus">
  <h2>Menus</h2>
  <ul>
    <li>Home</li>
    <li>About</li>
  </ul>
</div>
```

**命名槽（named slots）**

```html
//影子主体
<div class="menus">
  <slot></slot>
  <slot name="top"></slot>
  <slot name="right"></slot>
</div>

//影子树内容
<h2>Menus</h2>
<ul slot="top">
  <li>Home</li>
  <li>About</li>
</ul>
<ul slot="right">
  <li>Home</li>
  <li>Top</li>
</ul>

//渲染结果
<div class="menus">
  <h2>Menus</h2>
  <ul>
    <li>Home</li>
    <li>About</li>
  </ul>
  <ul>
    <li>Home</li>
    <li>Top</li>
  </ul>
</div>
```

拥有 name 属性的槽位由对应 slot 属性值相同的影子子树替换，而剩下的内容默认替换空名槽位，若不存在空名槽位，则剩余内容将被抛弃。

## HTML 模板（HTML Templates）

为了更友好的处理组件模板，Web Components 规范，支持`<template>`模板标签，HTML 模板定义了使用`<template>`标签声明可以通过脚本操作插入文档的 HTML 模板片段：

```html
<template id="menusTemplate">
  <ul>
    <li>Home</li>
    <li>About</li>
  </ul>
</template>
```

使用 JS 就可以访问到模板，并将其插入 DOM 中。

```js
var menusTemplate = document.querySelector('#menusTemplate');
var frag = document.importNode(menusTemplate.content, true);
document.querySelector('.menus').appendChild(frag);
```

### TEMPLATE 标签

`<template>`标签本质上与其他 HTML 内置标签一样，可以使用 DOM API 进行操作，但是需要明白，在将模板激活（生成 DOM 或插入文档）前：

1. `<template>`标签内的内容不会被渲染；
2. 标签内的图片，等媒体资源不会被加载；
3. 标签不会出现在 DOM 树，审查元素看不到；

## HTML 引入（HTML Imports）

在文档内直接引入外链资源的文档或 web 组件，语法如下，使用`<link>`标签：

```html
<link rel="import" href="components.html" />
```

为了避免重复执行引入文档内的脚本，对于已加载文档，import 方式将跳过其加载和执行过程。

## 创建步骤

1. 创建自定义元素/自定义内置元素
2. 编写 html 模板
3. 自定义元素绑定 ShadowDOM
4. 导入使用
