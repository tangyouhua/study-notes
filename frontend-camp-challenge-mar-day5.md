# 手写 Webpack（1）：原型与 Bundle

> 前端进阶训练营笔记-3月打卡-Day5，2023-3-5

请思考，下面的代码能否在浏览器中正常运行？

## 代码结构

有这样一个项目：

- index.html 加载 index.js
- index.js 调用 add.js 输出 add(2, 4) 结果
- add.js 定义了 add 方法

```Bash
.
├── index.html
└── src
    ├── add.js
    └── index.js
```

内容如下：

```HTML
<!--index.html-->
<script src="./src/index.js"></script>
```



```JavaScript
// index.js
var add = require("./add").default;
console.log(add(2, 4));
```



```JavaScript
// add.js
exports.default = function (a, b) {
  return a + b;
};

```

## 结果

答案是**不能**。浏览器会报告无法识别 `require`, `exports`。

如果希望在浏览器中能够正常运行上述代码，要怎么做？

## Webpack Bundle

### bundle.js 需求

假设，有这样一个文件 bundle.js 它能够，

- 提供 `require`, `exports`
- 按照期望的方式加载，比如 index.js ⇒ add.js

岂不是很好？

### 工具准备

在实现 bundle.js 之前，需要准备好两个工具：

- [IIFE（立即调用函数表达式）](https://developer.mozilla.org/zh-CN/docs/Glossary/IIFE)：在定义时就会立即执行的函数
- [eval 函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/eval)：将传入的字符当作 Javascript 代码执行

有了这两个工具，接下来就是如何构建模板，并正确填写代码。

### 实现

针对前面的例子，思路比较清晰：

1. 构建一个自运行匿名函数 `fn`；
2. 在匿名函数中，提供 `require`, `exports` 实现：
    1.  `require` 负责加载文件对应的 Javascript 代码（字符串）；
    2.  `require` 返回匿名自运行函数，这个函数以 `(exports, code)` 为参数，调用 `eval` 函数执行 `code` 字符串。
    3. 执行 `require("index.js")`。
3. 提供文件与代码（字符串）的对应列表 `list`；
4. 将上面例子中的 add.js, index.js 与代码填入 `list` 并执行 `fn`，达到期望的效果。

启动 live-server，在浏览器中打开 [http://127.0.0.1:5500/index.html](http://127.0.0.1:5500/index.html)，可以在 console 窗口看到结果 `6`。

完整代码可参考这里：[https://github.com/tangyouhua/lab-mini-webpack/tree/day1](https://github.com/tangyouhua/lab-mini-webpack/tree/day1)

### 动画演示

这个过程，就是 Webpack 打包的简化版本。

详细的动画可到[这篇文章](https://juejin.cn/post/6961961165656326152)中查看，直观形象。

## 遗留问题

目前这个版本不支持路径只有文件名，那么 Webpack 是如何解决的？

此文章为3月Day5学习笔记，内容基于极客时间前端训练营。
