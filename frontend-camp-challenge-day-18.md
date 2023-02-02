# Rollup 基础

> 前端进阶训练营笔记-打卡-Day18，2023-2-2

## 是什么

> [rollup.js](https://www.rollupjs.com/) 是一个 JavaScript 模块打包工具，可以将多个小的代码片段编译为完整的库和应用。与传统的 CommonJS 和 AMD 这一类非标准化的解决方案不同，Rollup 使用的是 ES6 版本 Javascript 中的模块标准。新的 ES 模块可以让你自由、无缝地按需使用你最喜爱的库中那些有用的单个函数。这一特性在未来将随处可用，但 Rollup 让你现在就可以，想用就用。

关键词：Javascript Module 打包工具，输出库或应用，支持 ES6 Module。

## 为什么

> ES6 版本的 Javascript 最终带来了 import 和 export 的语法（ES6 Module，译者注），该语法可以让我们在多个脚本中共享函数和数据。虽然这一语法标准已确定下来，但目前仅在较新的浏览器中可以使用，并且 Node.js 也尚未完成该标准的导入。有了 Rollup，你现在就可以放心地使用新的模块标准来书写代码，并将其编译为当前被广泛支持的 CommonJS、AMD 以及 IIFE 风格等多种格式。也就是说，你可以用符合未来标准的代码来赢得当下广大用户的芳心和敬意。:)

关键词：支持 CommonJS、AMD 以及 IIFE 风格等多种格式。

## 打包案例

安装 Rollup

```shell
npm install --global rollup
```

准备一段待打包的代码，例如  src/index.js

```js
export const whoAmI = 'I am utils package'
```

打包为 ES 格式，输出到 dist/bundle.es.js 文件

```shell
rollup -i src/index.js -o dist/bundle.es.js -f es
```

打包为 UMD 格式，输出到 dist/bundle.umd.js 文件

```shell
rollup -i src/index.js -o dist/bundle.umd.js -f umd -n utils
```

:warning: UMD 格式打包需要指定名字。

## 效果验证

### 测试 UMD 包的 IIFE 功能

index.html：

```html
<script src="./dist/bundle.umd.js"></script>

<script>
    console.log(utils.whoAmI)
</script>
```

在浏览器中通过调试工具查看 console，输出 `I am utils package`。

### 测试 AMD 方式

安装 requirejs，修改 index.html，通过 requirejs 调用：

```shell
npm install requirejs -d
```

```html
<script src="./node_modules/requirejs/require.js"></script>
<script>
    require(['./dist/bundle.umd.js'], utils => {
        console.log('amd requirejs: ', utils.whoAmI)
    })
</script>
```

再次在浏览器中通过调试工具查看 console，输出 `amd requirejs: I am utils package`。

### 测试 ES 方式

```js
const utils = require('./dist/bundle.umd')
console.log('utils:', utils.whoAmI)
```

执行

```shell
node index.js
utils: I am utils package
```

**官方文档**

- [命令行接口](https://www.rollupjs.com/guide/command-line-reference)
- [选项大列表](https://www.rollupjs.com/guide/big-list-of-options)
