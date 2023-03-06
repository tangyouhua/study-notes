# 手写 Webpack（2）：AST 与模块分析

> 前端进阶训练营笔记-3月打卡-Day6，2023-3-6

前文中我们有了一个手写Webpack的方案，通过 Javascript 自运行匿名函数和 `eval` 执行代码的方式进行打包。

## Day1 遗留问题

当前版本不支持路径只有文件名，那么 Webpack 是如何解决的？

### 问题描述

对下面的代码，编写 webpack.js 实现：

- 解析文件
- 对文件中的 `import ` 语句，解析依赖关系，得到文件绝对路径与文件内容
- 对于文件内容，要求解析为可遍历的对象 AST node

```JavaScript
// index.js
import add from './add'
console.log(add(2, 4))

// src/add.js
export default (a, b) => a + b;
```

## 实现思路

先考虑实现单个模块：

- 使用 Babel 解析模块代码
- 针对 `import` 声明进行路径解析，转换成绝对路径
- 读取文件内容，与绝对路径一起进行保存

### 工具

- `@babel/core` 
- `@babel/parser`：一个 JavaScript parser（也称作 Babylon），[官网介绍](https://babeljs.io/docs/babel-parser)
- `@babel/preset-env`
- `@babel/traverse`：遍历与更新 AST 节点，[官网介绍与示例](https://babeljs.io/docs/babel-traverse)

查阅API可参考 [Babel Plugin Handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md)。

## 核心代码

### webpack.js

下面是 webpack.js 中的 `getModuleInfo ` 函数实现：

- 用 Babel parser 按照 module 解析
- 遍历节点时，通过 `ImportDeclaration`回调处理相对路径与绝对路径转换
- 保存到 `deps` 对象
- 使用 [preset-env](https://babeljs.io/docs/env) 将 AST 转换为代码

```JavaScript
// find imports in a module
function getModuleInfo(file) {
  const body = fs.readFileSync(file, "utf-8");

  // AST: source code => objects => iterate
  const ast = parser.parse(body, {
    sourceType: "module",
  });

  const deps = {};
  traverse(ast, {
    ImportDeclaration({ node }) {
      const dirname = path.dirname(file);
      const abspath = "." + path.sep + path.join(dirname, node.source.value);
      deps[node.source.value] = abspath;
    },
  });

  // ES6 => ES5
  const { code } = babel.transformFromAst(ast, null, {
    presets: ["@babel/preset-env"],
  });
  const moduleInfo = { file, deps, code };
  return moduleInfo;
}
```

### 示例转换

之前的示例需要转换为 ES Module，参考前面“问题描述”中的代码。

### 单模块转换结果

```Bash
$ node .\src\webpack.js
info {
  file: './src/index.js',
  deps: { './add': '.\\src\\add' },
  code: '"use strict";\n' +
    '\n' +
    'var _add = _interopRequireDefault(require("./add"));\n' +
    'function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }\n' +
    'console.log((0, _add["default"])(2, 4));'
}
```

完整代码可参考这里：[https://github.com/tangyouhua/lab-mini-webpack/tree/day1](https://github.com/tangyouhua/lab-mini-webpack/tree/day2)

## 遗留问题

如何利用分析出的模块信息生成需要的 Bundle.js？如何支持多个模块？

此文章为3月Day6学习笔记，内容基于极客时间前端训练营。
