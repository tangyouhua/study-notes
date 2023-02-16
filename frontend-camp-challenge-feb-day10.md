# Mini Rollup（5）

> 前端进阶训练营笔记-2月打卡-Day10，2023-2-15

手写 RollUp，了解其原理。

## 目标

- v0.4
  - 增加 analyse 解析程序为作用域

## 步骤

### 编写 v0.4 版本

针对下面的例子，在解析出 ast 的基础上，实现 analyse 解析全局变量、函数变量、依赖：

- `_scope`：全局作用域
- `_defines`：函数定义
- `_dependsOn`：依赖

```js
const a = 1
function f() {
  const b = 2
}
```

首先，采用单元测试驱动，编写代码骨架：

```js
// lib/analyse.js
module.exports = function analyse(ast, magicString, module) {...}
```

为单条语句编写单元测试：

```js
// lib/__tests__/analyse.spec.js
describe('测试analyse', () => {
    it('_scope _defines', () => {
        const { ast, magicString } = getCode('const a = 1')
        analyse(ast, magicString)
        expect(ast._scope.contains('a')).toBe(true)
        expect(ast._scope.findDefiningScope('a')).toEqual(ast._scope)
        expect(ast.body[0]._defines).toEqual({ a: true })
    });
});
```

这里用到之前实验中的 acorn, magic-string 分别用来将代码解析为 ast，将 node 节点序列化为代码：

```shell
npm i acorn -D
npm i magic-string -D
```

提供 `getCode()` 工具方法：

```js
// lib/__tests__/analyse.spec.js
function getCode(code) {
    return {
        ast: acorn.parse(code, {
            locations: true,
            ranges: true,
            sourceType: 'module',
            ecmaVersion: 7
        }),
        magicString: new MagicString(code)
    }
}
```

接下来，思考如何实现 `analyse()`

- 利用 walk 以及 enter(node), leave(node) 可遍历 ast 树节点
- 将代码加入 <https://astexplorer.net/> 查看 ast 树结构
  - VariableDeclaration 节点：节点下的所有变量加入 scope
  - FunctionDeclaration 节点：
    - 本身就是作用域，需要与 parent Scope 关联；
    - 对节点下的所有变量加入 scope
  - Indentifier 节点：所有的标识符记录到依赖信息

开始动手。

先为语句增加需要的属性：

```js
// lib/analyse.js
function analyse(ast, magicString, module) {
  // ...
  let scope = new Scope({})
  ast._scope = scope

  // ...
  Object.defineProperties(statement, {
      _defines: { value: {} },
      _dependsOn: { value: {} }
  })
}
```

接着，用 walk 遍历 ast 节点：

```js
// lib/analyse.js
function analyse(ast, magicString, module) {
  // ...
  ast.body.forEach(statement => {
      walk(statement, {
          enter(node) {
              switch (node.type) {
                  case 'FunctionDeclaration': //...
                  case 'VariableDeclaration': //...
              }
          },
          leave(node) {}
      })
  })

  ast.body.forEach(statement => {
      walk(statement, {
          enter(node) {
              if (node.type === 'Identifier') //...
          }
      }
  }
}
```

最后，就是构造作用域，正确填写对应的属性。

```js
function analyse(ast, magicString, module) {
  //...
  // 添加作用域
  function addToScope(declaration) {
      const name = declaration.id.name
      scope.add(name)
      if (!scope.parent) {
          statement._defines[name] = true
      }
  }
}
```

在项目的根目录运行 `jest analyse` 一个单元测试通过。

补充 `_dependsOn` 测试用例：

```js
// lib/__tests__/analyse.spec.js
describe('_dependentsOn', () => {
    it('function语句', () => {
        const { ast, magicString } = getCode(
            `const a = 1
             function f() {
                 const b = 2
             }`)
        analyse(ast, magicString)
        expect(ast.body[0]._dependsOn).toEqual({ a: true })
        expect(ast.body[1]._dependsOn).toEqual({ f: true, b: true })
    });
});
```

增加下面的实现：

```js
// lib/analyse.js
function analyse(ast, magicString, module) {
  // ...
  ast.body.forEach(statement => {
      walk(statement, {
          enter(node) {
              if (node.type === 'Identifier') {
                  statement._dependsOn[node.name] = true
              }
          }
      })
  })
}
```

在项目的根目录运行 `jest analyse` 两个单元测试通过。

完整代码可查看 [v0.4 版本](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.4)。

## 效果

- [v0.4](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.4)
  - 执行 `jest analyse` 可以看到除2个单元测试通过

## 总结
- 增加 analyse，在 ast 基础上增加 `_scope` 全局作用域、`_defines` 函数定义、`_dependsOn` 依赖，供后续打包与 Treeshaking 使用

此文章为2月Day10学习笔记，内容来源于极客时间《前端进阶训练营》
