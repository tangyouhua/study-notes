# Mini Rollup（7）

> 前端进阶训练营笔记-2月打卡-Day12，2023-2-17

手写 RollUp，了解其原理。

## 目标

- v0.6
  - 增加 expandAllStatements 单模块内组装方法。将变量调用与声明合并，依赖的外部模块通过 bundle 加载，实现单模块 Tree shaking

## 步骤

### 编写 v0.6 版本

考虑下面这个例子：

```js
const a = () => 1
const b = () => 2
a()
```

在这个单模块里，期望解析后未实际调用的函数 b 会被清理，即 Tree shaking。

首先，编写单元测试：

```js
// lib/__tests__/module.spec.js
describe('ExpandAllStatements', () => {
    it('基础 ', () => {
        const code = `
            const a = () => 1
            const b = () => 2
            a()
        `
        const module = new Module({ code })
        const statements = module.expandAllStatements()
        expect(statements.length).toBe(2)
        expect(module.code.snip(statements[0].start,
            statements[0].end).toString()).toEqual(`const a = () => 1`)
        expect(module.code.snip(statements[1].start,
            statements[1].end).toString()).toEqual(`a()`)
    })
})
```

这里期待 `module.expandAllStatements()` 方法：

- 解析示例代码后，保留两条语句
- 第一条为函数 a 定义
- 第二条为函数 a 调用

思考新方法的实现

- 通过 acorn 将代码解析为 ast
- 由于只考虑单个模块，对节点类型为：ImportDeclaration，VariableDeclaration 的节点不做分析
- 递归解析节点
  - 标记当前节点为已处理
  - 找到语句的依赖，从 analyse 后的 `statement._dependsOn` 进行判断，对所有依赖语句递归展开，递归终止条件为语句已处理
  - 添加当前节点到结果列表

下面是核心的代码实现：

解析 ast 与 analyse 部分参考之前的实现，这里需要对 analyse 进行扩展：

```js
// lib/analyse.js
Object.defineProperties(statement, {
    _defines: { value: {} },
    _dependsOn: { value: {} },
    _included: { value: false, writable: true } // add _included
})
```

扩展所有语句：

```js
// lib/module.js
expandAllStatements() {
    const allStatements = []

    this.ast.body.forEach(statement => {
        // ignore import and declaration
        if (statement.type === 'ImportDeclaration' ||
            statement.type === 'VariableDeclaration') {
            return
        }

        const statements = this.expandStatement(statement)
        allStatements.push(...statements)
    })
    return allStatements
}
```

扩展单条语句：

```js
expandStatement(statement) {
    // terminate recursion: current statement is included
    statement._included = true

    const result = []
    // _dependsOn: {a: true, b: true}
    const dependencies = Object.keys(statement._dependsOn)
    dependencies.forEach(name => {
        const definitions = this.define(name)
        result.push(...definitions)
    })

    result.push(statement)
    return result
}
```

递归扩展单条语句，递归终止条件为 statement._inclded = true

```js
// lib/module.js
define(name) {
    if (has(this.imports, name)) { // from import
        // todo
    } else { // this module
        const statement = this.definitions[name]
        if (statement) {
            if (statement._included) {
                return []
            } else {
                // recursive call case: const b = a + 1 => a = 3 + f => f = 1
                return this.expandStatement(statement)
            }
        } else if (SYSTEM_VARS.includes(name)) {
            return []
        } else {
            throw new Error(`variable ${name} not found`)
        }
    }
}
```

完整代码可查看 [v0.6 版本](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.6)。

## 效果

- [v0.6](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.6)
  - 执行 `jest module` 可以看到除1个单元测试通过

## 总结

- 增加 expandAllStatements，实现单个模块 Tree shaking

此文章为2月Day12学习笔记，内容来源于极客时间《前端进阶训练营》
