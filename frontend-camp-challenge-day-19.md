# Babel 编译器

> 前端进阶训练营笔记-打卡-Day19，2023-2-3

## 为什么

解决代码兼容问题：将 ECMA2015+ [新特性](https://babeljs.io/docs/en/learn.html)转换成能够兼容老版本浏览器的 Javascript 代码。

## 是什么

[Babel](https://babeljs.io/) 是一个 Javascript 编译器，提供[以下功能](https://babeljs.io/docs/en/)：

* 转换语法
* 垫片 Polyfill
* 源代码转换
* 等等

Babel 有以下概念

* 配置文件
* 配置选项
* Preset
* Plugin

下面通过一个示例了解 Babel 中的概念与实际使用。

## 操作实例

准备工作：安装 babel-cli

```shell
npm i babel-cli -g
```

新建示例代码：sample.js

```js
const a = '123'
const add = (a, b) => a + b
const set = new Set([1,2,3])
```

在项目根目录新建配置文件：.babelrc

首先，配置一个插件，转换箭头函数。

1. 安装插件：`npm i babel-plugin-transform-es2015-arrow-functions -d`
2. 配置文件加入该插件
3. 执行代码转换：`babel sample.js`
4. 可以看到，转换后箭头函数 `=>` 进行了转换

> 资源：[Plugin 列表](https://babeljs.io/docs/en/plugins-list)

```json
{
  "plugins": [
    "babel-plugin-transform-es2015-arrow-functions"
  ]
}
```

```shell
% babel sample.js
const a = '123';
const add = function (a, b) {
  return a + b;
};
const set = new Set([1, 2, 3]);
```

接下来，思考一个问题：如果有很多功能要配置，岂不是要安装并配置很多的插件？

这时就有了预设 Preset。比如常用的预设 @babel/preset-env ES2015

1. 安装插件：`npm i --save-dev babel-preset-es2015`
2. 配置文件加入该预设
3. 执行代码转换：`babel sample.js`
4. 可以看到，转换后箭头函数 `=>` 同样进行了转换

> 资源：[Preset 列表](https://babeljs.io/docs/en/presets)

```json
{
  "presets": [
    "babel-preset-es2015"
  ]
}
```

```shell
% babel sample.js
'use strict';

var a = '123';
var add = function add(a, b) {
  return a + b;
};
var set = new Set([1, 2, 3]);
```

最后，了解如何引入垫片 Polyfill 并查看 Preset 转换后的效果

1. 安装插件：`npm i @babel/polyfill -s`
2. 在 sample.js 中引入 polyfill
3. 执行代码转换：`babel sample.js`
4. 可以看到，转换后箭头函数与 import module 都进行了转换

```js
import "@babel/polyfill"

const a = '123'
const add = (a, b) => a + b
const set = new Set([1,2,3])
```

```js
"use strict";

require("@babel/polyfill");

var a = '123';
var add = function add(a, b) {
  return a + b;
};
```
