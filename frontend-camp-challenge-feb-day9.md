# Mini Rollup（4）

> 前端进阶训练营笔记-2月打卡-Day9，2023-2-14

手写 RollUp，了解其原理。

## 目标

- v0.3
  - 设计并实现 Scope 作用域

### 概念介绍

涉及的概念介绍：

### Scope（作用域）

在 MDN 的定义<sup>[2][ref2]</sup>中，作用域是当前的执行上下文，值和表达式在其中“可见”或可被访问。如果一个变量或表达式不在当前的作用域中，那么它是不可用的。作用域也可以堆叠成层次结构，子作用域可以访问父作用域，反过来则不行。

JavaScript 的作用域分以下三种：

- 全局作用域：脚本模式运行所有代码的默认作用域
- 模块作用域：模块模式中运行代码的作用域
- 函数作用域：由函数创建的作用域

[ref2]: https://developer.mozilla.org/zh-CN/docs/Glossary/Scope

这里的实验中，考虑的是全局与函数作用域。

## 步骤

### 编写 v0.3 版本

首先，设计 Scope 类：

```js
class Scope {
  constructor(options) {...} // 创建作用域，设置 parent 副作用域与 names 变量名列表
  add(name) {...} // 添加变量名到作用域
  contains(name) {...} // 判断变量是否被声明
  findDefiningScope(name) {...} // 返回变量所在作用域
}
```

这里需要注意的是，findDefiningScope 除了查找当前作用域下是否有包含 name。当查找失败时，如果有父作用域还要向父作用域查找。

接着，按照测试驱动开发方式，编写测试用例：

```js
// lib/scope.spec.js
const Scope = require('../scope')

describe('测试Scope', () => {
    it('简单父子关系', () => {
        const root = new Scope({})
        root.add('a')
        const child = new Scope({
            parent: root
        })
        child.add('b')
        expect(child.contains('b')).toBe(true)
        expect(child.contains('a')).toBe(true)
        expect(child.findDefiningScope('a')).toBe(root)
        expect(child.findDefiningScope('b')).toBe(child)
    });
});
```

上面的测试，对当前作用域和父作用域进行了验证。包括测试作用域是否包含变量，以及所在作用域节点测试。

然后，编写 Scope 核心代码：

```js
// lib/scope.js
class Scope {
    constructor(options) {
        this.parent = options.parent
        this.names = options.names
    }

    add(name) {
        if (this.names) {
            this.names.push(name)
        } else {
            this.names = [name]
        }
    }

    contains(name) {
        return !!this.findDefiningScope(name) // 转换为 boolean
    }

    findDefiningScope(name) {
        if (this.names.includes(name)) {
            return this
        } else if (this.parent) {
            return this.parent.findDefiningScope(name) // 递归查找
        }
        return null
    }
}
module.exports = Scope
```

最后，在实验项目的根目录运行单元测试 `jest scope`，单元测试通过。

## 效果

- [v0.3](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.3)
  - 执行 `jest scope` 可以看到除1个单元测试通过

## 总结

- 设计 Scope 作用域类（全局与函数作用域），为后续 RollUp 解析与打包做准备

此文章为2月Day9学习笔记，内容来源于极客时间《前端进阶训练营》
