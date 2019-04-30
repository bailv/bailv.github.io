---
title: Node的文件路径
date: 2019-04-30 17:25:03
tags: Nodejs
categories: javascript
---

Node 中的文件路径大概有`__dirname`,`__filename`,`process.cwd()`, `./` 或者 `../`，前三个都是绝对路径，为了便于比较，`./`和`../`我们通过 `path.resolve('./')`来转换为绝对路径。

例如当前的文件结构如下：

```bash
app/
    -lib/
        -common.js
    -model
        -tesk.js
        -test.js
```

在`model/task.js`中编写如下代码：

```js
var path = require('path');

console.log(__dirname);
console.log(__filename);
console.log(process.cwd());
console.log(path.resolve('./'));
```

在`mode/test.js`中编写如下代码：

```js
var fs = require('fs');
var common = require('../lib/common');

fs.readFile('../lib/common.js', function(err, data) {
  if (err) return console.log(err);
  console.log(data);
});
```

在`app`目录下，执行`node model/task.js`，得到的输出：

```bash
/$dir/app/model
/$dir/app/model/task.js
/$dir/app
/$dir/app
```

执行`node model/test.js`，会报路劲错误，`readFile`找不到`../lib/common.js`路劲

**结论：**

- `__dirname`: 总是返回被执行的 js 所在文件夹的绝对路径
- `__filename`: 总是返回被执行的 js 的绝对路径
- `process.cwd()`: 总是返回运行 node 命令时所在的文件夹的绝对路径
- `../`: 在`require()`中使用,返回被运行的 js 所在的上一层文件夹路劲,其他情况下返回运行 node 命令时所在的上一层文件夹路劲
- `./`: 在`require()`中使用是跟`__dirname`的效果相同，不会因为启动脚本的目录不一样而改变，在其他情况下跟`process.cwd()`效果相同，是相对于启动脚本所在目录的路径。

只有在`require()`时才使用相对路径`(./, ../)`的写法，其他地方一律使用绝对路径，如下：

```js
// 当前目录下
path.dirname(__filename) + '/test.js';
// 相邻目录下
path.resolve(__dirname, '../lib/common.js');
```
