# Mini Rollup（6）

> 前端进阶训练营笔记-2月打卡-Day11，2023-2-16

手写 RollUp，了解其原理。

## 目标

- v0.5
  - 增加 module 解析 imports, exports, definitions

## 步骤

### 编写 v0.5 版本

首先，采用单元测试驱动，编写代码骨架：

```js
// lib/module.js
class Module {
    constructor({ code }) {...}
}
module.exports = Module
```

为 import 编写单元测试：

```js
// lib/__tests__/module.spec.js
it('单个import', () => {
     const code = `import { a as aa } from '../module'`
     const module = new Module({ code })
     expect(module.imports).toEqual({
         aa: {
             'localName': 'aa',
             'name': 'a',
             'source': '../module'
         }
     })
});
```

接下来，在 module.js 中实现相应功能：

- 引入上个版本实现的 analyse 功能
- 在构造函数中用 acorn 解析为 ast
- 进行 analyse
  - 针对 ast 节点类型为 ImportDeclaration 进行处理
  - 设置 imports，包括 name 名称，localName 别名，module 名

核心代码如下：

```js
// lib/module.js
const analyse = require('./analyse')

class Module {
   constructor({ code }) {
       this.code = new MagicString(code)
       this.ast = parse(code, {
           ecmaVersion: 7,
           sourceType: 'module'
       })
       this.analyse()
   }

   analyse() {
        this.imports = {}
        this.ast.body.forEach(node => {
            if (node.type === 'ImportDeclaration') {
                const source = node.source.value;
                const { specifiers } = node
                specifiers.forEach(specifier => {
                    const localName = specifier.local ? specifier.local.name : ''
                    const name = specifier.imported ? specifier.imported.name : ''
                    this.imports[localName] = { name, localName, source }
                })
            }
        }
   }
}
```

在实验项目的根目录运行单元测试 `jest module`，单元测试通过。

接着，加入多条 import 单元测试内容；

```js
// lib/__tests__/module.spec.js
it('多个import', () => {
    const code = `import { a as aa, b } from '../module'`
    const module = new Module({ code })
    expect(module.imports).toEqual({
        aa: {
            'localName': 'aa',
            'name': 'a',
            'source': '../module'
        },
        b: {
            'localName': 'b',
            'name': 'b',
            'source': '../module'
        }
    })
});
```

再次运行单元测试 `jest module`，2个单元测试通过。

解决了 import，接下来来处理 exports。实现的思路类似：

- 进行 analyse
  - 针对 ast 节点类型为 Export 开头的类型进行处理
  - 设置 exports，包括 node 节点，localName 别名，expression 导出的语句 

核心代码如：

```js
// lib/module.js
class Module {
   analyse() {
        this.exports = {}
        if (/^Export/.test(node.type)) {
            const declaration = node.declaration
            if (declaration.type === 'VariableDeclaration') {
                if (!declaration.declarations)
                    return
                const localName = declaration.declarations[0].id.name
                this.exports[localName] = {
                    node,
                    localName,
                    expression: declaration
                }
            }
        }
   }
}
```

为 exports 添加单元测试：

```js
// lib/__tests__/module.spec.js
describe('exports', () => {
    it('单个export', () => {
        const code = `export var a = 1`
        const module = new Module({ code })
        expect(module.exports['a'].localName).toBe('a')
        expect(module.exports['a'].node).toBe(module.ast.body[0])
        expect(module.exports['a'].expression).toBe(module.ast.body[0].declaration)
    });
});
```

运行单元测试 `jest module`，3 个单元测试通过。

最后，为导出的定义 definition 增加单元测试：

```js
// lib/__tests__/module.spec.js
describe('definitions', () => {
    it('单个变量', () => {
        const code = `const a = 1`
        const module = new Module({ code })
        expect(module.definitions).toEqual({
            a: module.ast.body[0]
        })
    });
});
```

利用之前完成的 analyse 中解析出的 `_defines` 进行赋值：

```js
// lib/module.js
class Module {
    analyse() {
         analyse(this.ast, this.code, this)
         this.definitions = {}
         this.ast.body.forEach(statement => {
             Object.keys(statement._defines).forEach(name => {
                 this.definitions[name] = statement
             })
         })
    }
}
```

最后运行单元测试 `jest module`，全部 4 个单元测试通过。

完整代码可查看 [v0.5 版本](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.5)。

## 效果

- [v0.5](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.5)
  - 执行 `jest module` 可以看到除4个单元测试通过

## 总结

- 增加 module，在 module 中解析出 import，export 与所有的定义。利用了 analyse 中的 `_defines` 结果

此文章为2月Day11学习笔记，内容来源于极客时间《前端进阶训练营》
