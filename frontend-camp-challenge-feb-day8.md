# Mini Rollup（3）

> 前端进阶训练营笔记-2月打卡-Day8，2023-2-13

手写 RollUp，了解其原理。

## 目标

- v0.2
  - 利用 v0.1 实现的 walk 打印函数作用域

### 概念介绍

涉及的概念介绍：

### AST 节点遍历器

AST (Abstract Syntax Tree) 节点遍历器是一种用于处理抽象语法树（AST）的工具。它通过遍历 AST 中的节点，从而可以访问和操作该语法树中的每个节点。

一个典型的 AST 节点遍历器实现了一种递归算法，它遍历每个节点，并递归地访问它的所有子节点。当遍历器到达一个节点时，它可以执行任意操作，例如替换节点的值、删除节点或添加新的节点。

这种工具在处理编译原理和代码生成等方面非常有用。它可以在没有改变源代码的前提下，对代码进行各种操作，并生成新的代码。例如，它可以用于代码优化、代码转换、代码分析等。

在实验中，利用 walk 提供的 `enter`, `leave` 函数，用缩进 indent 体现函数作用域。为 RollUp 的打包功能做好准备。

## 步骤

### 编写 v0.2 版本

针对下面的示例：

```js
// functionscope/source.js
const a = 1
function f1() {
    const b = 2
    function f2() {
        const c = 3
    }
}
```

期望输出：

```
 var: a
 func: f1
     var: b
     func: f2
         var: c
```

**实现思路**

根据 AST node 的类型：

- 打开 <https://astexplorer.net/>
- 粘贴示例代码，理解示例代码结构：针对 VariableDeclarator 与 FunctionDeclaration 进行处理
- 在 walk 递归遍历时，默认 indent 级别为 0
  - 遇到 VariableDeclarator 打印带锁进的变量名
  - 遇到 FunctionDeclaration indent++
  - 打印带缩进的函数名
  - 退出 FunctionDeclaration indent--

**核心代码**

```js
// funcscope/index.js
const fs = require('fs')
const acorn = require('acorn')
const MagicString = require('magic-string')

const code = fs.readFileSync('./source.js').toString()

const ast = acorn.parse(code, {
    sourceType: 'module',
    ecmaVersion: 7
})

const walk = require('../../lib/walk')
let indent = 0
const withindent = () => ' '.repeat(indent * 4)
walk(ast, {
    enter(node) {
        if (node.type === 'VariableDeclarator') {
            console.log(withindent(), 'var:', node.id.name)
        }

        if (node.type === 'FunctionDeclaration') {
            console.log(withindent(), 'func:', node.id.name)
            indent++
        }
    },

    leave(node) {
        if (node.type === 'FunctionDeclaration') {
            indent--
        }
    }
})
```

## 效果

- [v0.2](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.2)
  - 进入 funcscope 目录，执行 `node index.js` 可以看到带作用域的函数输出

## 总结

- AST walk 可用来记录函数作用域，在后续 RollUp 打包中，是很重要的基础

此文章为2月Day8学习笔记，内容来源于极客时间《前端进阶训练营》
