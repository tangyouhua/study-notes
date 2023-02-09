# 抽象语法树 AST 与实验

> 前端进阶训练营笔记-2月打卡-Day4，2023-2-9

## 是什么

[抽象语法树-wikipedia](https://zh.wikipedia.org/zh-cn/%E6%8A%BD%E8%B1%A1%E8%AA%9E%E6%B3%95%E6%A8%B9)

> 在计算机科学中，抽象语法树（Abstract Syntax Tree，AST），或简称语法树（Syntax tree），是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。比如，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现；而类似于 if-condition-then 这样的条件跳转语句，可以使用带有三个分支的节点来表示。

比如下面这段 Javascript 代码：

```js
let message = 'greetings from youhua'
console.log(message)
```

打开工具网站，例如 [astexplorer](https://astexplorer.net/)，可以看到对应的抽象语法树。如下图：

![Javascript Greeting AST](images/ast-js-greeting.png)

这里可以看到，JavaScript AST 遵循 [ECMAScript 语言标准](https://262.ecma-international.org/9.0/#sec-intro)。

如果要了解更多每个 AST 元素的含义，可以在标准文档中搜索。例如，VariableDeclaration 可以在 [Variable Statement](https://262.ecma-international.org/9.0/#sec-variable-statement) 中看到对应的说明。

## 为什么

理解 AST 及相关的开发库与工具，对前端工程化工具的原理有更好地理解，比如 RollUp, Webpack 等。

## 怎么做

根据需求，通常的操作步骤如下：

1. 解析文件，转换为 AST
2. 遍历 AST，根据需求对符合条件的 AST 节点进行操作
3. 将 AST 输出为代码

## 实操案例

为 `console.log`, `console.error` 加上所在的文件与行号。

首先，建立测试项目，添加实验所需要的依赖。

```shell
mkdir lab-ast
cde lab-ast
npm i @babel/parser -D
npm i @babel/traverse -D
npm i fs
```

接着，准备待处理的输入文件 src/source.tsx。

```ts
console.log(1)
function log(): number {
  console.debug('before')
  console.error(2)
  console.debug('after')
  return 0
}
log()
class Foo {
  bar(): void {
    console.log(3)
  }
  render() {
    return ''
  }
}
```

然后，编写 src/index.js 实现功能。

```js
const parser = require('@babel/parser')
const traverse = require('@babel/traverse').default
const generator = require('@babel/generator').default
const types = require('@babel/types')
const fs = require('fs')
const fileName = 'source.tsx'

// read files
const source = fs.readFileSync(__dirname + '/' + fileName).toString()

// transform ast
const ast = parser.parse(source, {
  plugins: ['typescript', 'jsx']
})

// walker
traverse(ast, {
  // visitor
  CallExpression(path) {
    const calleeStr = generator(path.node.callee).code
    if (['console.log', 'console.error'].includes(calleeStr)) {
      const {line, column} = path.node.loc.start
      path.node.arguments.unshift(types.stringLiteral(`${fileName}(${line}, ${column})`))
    }
  }
})

const {code} = generator(ast, {fileName})
console.log('code:', code)
```

最后，测试效果：

```shell
$ node ./src/index.js 
code: console.log("source.tsx(1, 0)", 1);
function log(): number {
  console.debug('before');
  console.error("source.tsx(4, 2)", 2);
  console.debug('after');
  return 0;
}
log();
class Foo {
  bar(): void {
    console.log("source.tsx(11, 4)", 3);
  }
  render() {
    return '';
  }
}
```

如果调整一下 `console.error()` 所在行，再次运行会发现结果也随之变化：

```ts
console.log(1)
function log(): number {
  console.error(2)
  console.debug('after once')
  console.debug('after twice')
  return 0
}
log()
class Foo {
  bar(): void {
    console.log(3)
  }
  render() {
    return ''
  }
}
```

```shell
$ node ./src/index.js 
code: console.log("source.tsx(1, 0)", 1);
function log(): number {
  console.error("source.tsx(3, 2)", 2);
  console.debug('after once');
  console.debug('after twice');
  return 0;
}
log();
class Foo {
  bar(): void {
    console.log("source.tsx(11, 4)", 3);
  }
  render() {
    return '';
  }
}
```

此文章为2月Day4学习笔记，内容来源于极客时间《前端进阶训练营》
