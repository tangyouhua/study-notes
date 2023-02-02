# 模块化

> 前端进阶训练营笔记-打卡-Day17，2023-2-1
## 是什么

什么是 JavaScript 模块？

> - MDN（Mozilla Developer Network）给出的[定义][doc-js-modules]：一种将 JavaScript 程序拆分为可按需导入的单独模块的机制。
> - ECMA 规范给出的 [Module 结构定义][doc-ecma-modules]: Module, ModuleItem, Module Import, Module Export。
> - Node.js 给出的 [Module 定义][doc-nodejs-esm]: ECMAScript modules 是 JavaScript 代码打包重用的官方标准格式，通过 `import` 与 `export` 语句进行定义。
## 为什么

关于 Module 出现的背景，[MDN][doc-js-modules] 给出了下面的说明：

> JavaScript 程序本来很小——在早期，它们大多被用来执行独立的脚本任务，在你的 web 页面需要的地方提供一定交互，所以一般不需要多大的脚本。过了几年，我们现在有了运行大量 JavaScript 脚本的复杂程序，还有一些被用在其他环境（例如 Node.js）。
>
> 因此，近年来，有必要开始考虑提供一种将 JavaScript 程序拆分为可按需导入的单独模块的机制。
[RequireJS][doc-requirejs-why] 给出了下面的说明：

> - 网站转变成了 Web 应用
> - 代码变得越来越复杂
> - 集成变得更困难
> - 开发者需要独立的 JS 文件或模块
> - 部署的代码要进行优化，以便于在一个或几个 HTTP 请求就能完成需要的功能
[doc-js-modules]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules
[doc-ecma-modules]: https://tc39.es/ecma262/#sec-modules
[doc-nodejs-esm]: https://nodejs.org/api/esm.html
[doc-requirejs-why]: https://requirejs.org/docs/why.html

## 备选方案

- [CommonJS][doc-commonjs-wiki]：Node.js
- 基于 AMD（Asynchronous Module Definition）的其他模块系统，如 [RequireJS][doc-requirejs]
- [Webpack][doc-webpackjs]：从[模块化编程概念][doc-modularprogramming-wiki]触发，支持 [Webpack Module][doc-webpackjs-module]
- [Babel][doc-babeljs]
- 基于 CMD（Common Module Definition）的模块化系统：[Sea.js][doc-seajs]

[doc-commonjs-wiki]: https://en.wikipedia.org/wiki/CommonJS
[doc-requirejs]: https://requirejs.org/docs/start.html
[doc-webpackjs]: https://webpack.js.org/concepts/
[doc-modularprogramming-wiki]: https://en.wikipedia.org/wiki/Modular_programming
[doc-webpackjs-module]: https://webpack.js.org/concepts/modules/#what-is-a-webpack-module
[doc-babeljs]: https://babeljs.io/docs/en/
[doc-seajs]: https://seajs.github.io/seajs/docs/#docs

方案比对：

|                           | 运行环境      | 加载方式 | 运行机制      | 特点                                               |
| ------------------------- | ------------- | -------- | ------------- | -------------------------------------------------- |
| CommonJS                  | 服务器        | 同步     | 运行时        | 第一次加载后会将结果缓存，再次加载会读取缓存的结构 |
| AMD                       | 浏览器        | 异步     | 运行时        | 依赖前置，不管模块是否用到都会全量加载             |
| CMD                       | 浏览器        | 异步     | 运行时        | 依赖就近，延迟加载                                 |
| ESM（ECMAScript Modules） | 浏览器/服务器 | 异步     | 编译时/运行时 | 静态化，在编译时就确定模块之间的依赖关系           |

## 操作实例

1 立即执行函数 IIFE（Immediately-Invoked Function Expression）

原始代码

```js
const spliter = '#'
const format = str => spliter + str + spliter
const util = {
  format
}
```

- 全局污染
- `spliter` 变量有被修改的风险

立即执行函数

```js
const util = (function() {
  const spliter = '#'
  const format = str => spliter + str + spliter
  return {
    format
  }
})
```

2 CommonJS 规范

```js
// a.js
export.a = 'a'
// index.js
const moduleA = require('./a.js')
console.log(moduleA) // {a: 'a'}
```

3 AMD 示例

```js
//Calling define with a dependency array and a factory function
define(['dep1', 'dep2'], function (dep1, dep2) {
    //Define the module value by returning a value.
    return function () {};
});
// Or:
define(function (require) {
    var dep1 = require('dep1'),
        dep2 = require('dep2');
    return function () {};
});
```

3 ESM 示例

```js
//------ lib.js ------
export const sqrt = Math.sqrt;
export function square(x) {
    return x * x;
}
export function diag(x, y) {
    return sqrt(square(x) + square(y));
}
//------ main.js ------
import { square, diag } from 'lib';
console.log(square(11)); // 121
console.log(diag(4, 3)); // 5
```

关于 Modules 详细的示例可参考：[js-example 示例仓库](https://github.com/mdn/js-examples)

## 参考学习

- [ES6 In Depth: Modules](https://hacks.mozilla.org/2015/08/es6-in-depth-modules/)
- [ES modules: A cartoon deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)：漫画讲解 ES Modules
- [Exploring ES6 Modules 章节](https://exploringjs.com/es6/index.html#toc_ch_modules)
- [JavaScript Module Systems Showdown: CommonJS vs AMD vs ES2015](https://auth0.com/blog/javascript-module-systems-showdown/)：各种模块化案例与比较
