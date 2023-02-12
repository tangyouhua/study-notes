# Mini Rollup（2）

> 前端进阶训练营笔记-2月打卡-Day7，2023-2-12

手写 RollUp，了解其原理。

## 目标

- v0.1
  - 为简单对象、数组实现 AST 节点遍历器 walk

### 概念介绍

涉及的概念介绍：

### AST 节点遍历器

AST (Abstract Syntax Tree) 节点遍历器是一种用于处理抽象语法树（AST）的工具。它通过遍历 AST 中的节点，从而可以访问和操作该语法树中的每个节点。

一个典型的 AST 节点遍历器实现了一种递归算法，它遍历每个节点，并递归地访问它的所有子节点。当遍历器到达一个节点时，它可以执行任意操作，例如替换节点的值、删除节点或添加新的节点。

这种工具在处理编译原理和代码生成等方面非常有用。它可以在没有改变源代码的前提下，对代码进行各种操作，并生成新的代码。例如，它可以用于代码优化、代码转换、代码分析等。

在实验中，通过自己实现 walk 理解节点遍历器的作用，为 RollUp 的打包功能做好准备。

## 步骤

### 编写 v0.1 版本

本次采用 Jest 测试驱动开发的方式实现 walk 例子。

首先，在项目根目录添加 Jest 配置文件：

```js
// ./jest.config.js
module.exports = {
  roots: ['./lib']
}
```

接着，创建 walk.js，加入函数初始定义：

```js
// lib/walk.js
function walk(ast, { enter, leave} ) {
  enter(666)
  leave(888)
}
```

为其创建单元测试：

```js
// lib/__tests__/walk.spec.js
const walk = require('../walk')

describe('单个walk测试函数', () => {
  it('单个节点', () => {
    const ast = { 'a': 1 }
    const enter = jest.fn()
    const leave = jest.fn()

    walk(ast, { enter, leave })

    // 验证 enter(666)
    let calls = enter.mock.calls
    console.log(calls)
    expect(calls.length).toBe(1)
    expect(calls[0][0]).toEqual(666)

    // 验证 leave(888)
    calls = leave.mock.calls
    expect(calls.length).toBe(1)
    expect(calls[0][0]).toEqual(888)
  })
}
```

这里用到了 mock 函数 `jest.fn()` 来测试函数是否被正确调用。

执行 `jest walk` 可以查看测试结果。也可以增加 `--watch` 参数，让代码修改后自动运行单元测试。

骨架搭建完成后修改测试用例，传入我们期望的结果：

```js
it('单个节点', () => {
  const ast = { 'a': 1 }
  const enter = jest.fn()
  const leave = jest.fn()

  walk(ast, { enter, leave })

  // 验证 enter({ 'a': 1 })
  let calls = enter.mock.calls
  console.log(calls)
  expect(calls.length).toBe(1)
  expect(calls[0][0]).toEqual({ 'a': 1 })

  // 验证 leave({ 'a': 1 })
  calls = leave.mock.calls
  expect(calls.length).toBe(1)
  expect(calls[0][0]).toEqual({ 'a': 1 })
})
```

这时，单元测试会失败。所以，我们要根据要求完善 walk 的代码：

- 将 `enter`, `leave` 函数直接调用测试值，改为调用 ast
- 实现 `visit` 遍历节点，
  - 调用 `enter` 函数；
  - 根据 AST node 的类型分别处理：如果为 `object`，则递归遍历各属性；
  - 调用 `leave` 函数。

```js
function walk(ast, { enter, leave} ) {
  visit(ast, null, enter, leave)
}

function visit(node, parent, enter, leave) {
  if (!node) return
  
  if (enter) {
    enter.call(null, node, parent)
  }

  // 对象遍历
  const childKeys = Object.keys(node)
                          .filter((key) => typeof(node[key]) === 'object')
  childKeys.forEach((childKey) => {
    const value = node[childKey]
    visit(value, node, enter, leave)
  })

  if (leave) {
    leave.call(null, node, parent)
  }
}

module.exports = walk
```

运行单元测试，`jest walk` 结果为单元测试通过。

接下来，增加测试用例，例如对象中包含数组：

- 进入对象第一级，然后逐个层次处理
  - `enter`
  - 递归遍历下一级
  - `leave`
- 编写测试用例时，需要注意 `leave` 函数的断言要从最内层开始。

```js
it('多个节点', () => {
  const ast = { 'a': [{ 'b': 2 }] }
  const enter = jest.fn()
  const leave = jest.fn()

  walk(ast, { enter, leave })

  // 验证 enter({ 'a': [{ 'b': 2 }] }）
  let calls = enter.mock.calls
  expect(calls.length).toBe(3)
  expect(calls[0][0]).toEqual({ 'a': [{ 'b': 2 }] })
  expect(calls[1][0]).toEqual([{ 'b': 2 }])
  expect(calls[2][0]).toEqual({ 'b': 2 })

  // 验证 leave({ 'a': [{ 'b': 2 }] }）
  calls = leave.mock.calls
  expect(calls.length).toBe(3)
  expect(calls[0][0]).toEqual({ 'b': 2 })
  expect(calls[1][0]).toEqual([{ 'b': 2 }])
  expect(calls[2][0]).toEqual({ 'a': [{ 'b': 2 }] })
})
```

同样的，执行 `jest walk` 可以查看测试结果。

这里，实验只处理了简单类型和包含数组的对象。实际开发时，需要处理 ECMAScript 规范中的各种情形。

## 效果

- [v0.1](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.1)
  - 执行 `jest walk` 可以看到除2个单元测试通过

## 总结

- AST walk 是处理复杂节点的递归访问工具，在此基础上可以实现打包、代码转换等功能

此文章为2月Day7学习笔记，内容来源于极客时间《前端进阶训练营》
