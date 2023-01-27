# TDD Vue (5)

> 极客时间前端进阶训练营练习项目—打卡-Day12，2023-1-27

大纲

- [TDD Vue](#tdd-vue)
  - [目标](#目标)
  - [步骤](#步骤)
    - [编写 v0.4 版本](#编写-v04-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.4
  - 增加 generate 代码生成，实现 AST 转换为 Javascript

### 编写 v0.4 版本

**为什么？**

编译器是 Vue 的核心功能，主要分为：

1. 解析 template 为 AST；
2. 对 AST 进行转换（可选）；
3. 将 AST 生成为 Javascript 代码。

v0.4 版本将第 1 步解析的结果 AST 生成 Javascript 代码，从而实现 Vue template 的最终效果，即所有代码最终都是 Javascript 函数。

**单元测试**

生成文本元素

- 输入：Element `div`, 包含子节点 Text `foo`
- 输出：Javascript 代码
  - `return this._c('div',null,'foo')`

```js
it('generate element with text', () => {
    const ast = [
        {
            type: 'Element',
            tag: 'div',
            props: [],
            isUnary: false,
            children: [{ type: 'Text', content: 'foo' }]
        }
    ]
    const code = generate(ast)
    expect(code).toMatch(`return this._c('div',null,'foo')`)
})
```

生成包含表达式的元素

- 输入：Element `div`, 包含子节点 Interpolcation, 插值表达式内容为 `foo`
- 输出：Javascript 代码
  - `return this._c('div',null,this.foo)`

```js
it('generate element with expression', () => {
    const ast = [
        {
            type: 'Element',
            tag: 'div',
            props: [],
            isUnary: false,
            children: [
                {
                    type: 'Interpolation',
                    content: { type: 'Expression', content: 'foo' }
                }
            ]
        }
    ]
    const code = generate(ast)
    expect(code).toMatch(`return this._c('div',null,this.foo)`)
})
```

生成包含多个子节点的元素

- 输入：Element `div`, 包含两个子节点
  - 子节点 Text，文本内容为 `foo`
  - 子节点 Element `span`, 该直接点包含文本 Text, 内容为 `bar`
- 输出：Javascript 代码
  - `return this._c('div',null,[this._v('foo'),this._c('span',null,'bar')])`

```js
it('generate element with multi children', () => {
    const ast = [
        {
            type: 'Element',
            tag: 'div',
            props: [],
            isUnary: false,
            children: [
                { type: 'Text', content: 'foo' },
                {
                    type: 'Element',
                    tag: 'span',
                    props: [],
                    isUnary: false,
                    children: [{ type: 'Text', content: 'bar' }]
                }
            ]
        }
    ]
    const code = generate(ast)
    expect(code).toMatch(
        `return this._c('div',null,[this._v('foo'),this._c('span',null,'bar')])`
    )
})
```

**实现思路**

- 核心是手工实现 generate() 方法：即提供一个从 AST（抽象语法树）生成 Javascript 代码的生成器
  - 遍历 AST 节点与子节点
  - 采用[模板字符串][doc-jslang-template-literals]生成代码，主要用到 `this._v()` 与 `this._c()` 函数
  - 返回当前代码
- 遍历所有节点，递归结束

**核心代码**

```js
export function generate(ast) {
    const code = genNode(ast[0])
    return `return ${code}`
}

// 根据节点类型调用对应处理
function genNode(ast) {
    if (ast.type === 'Element') {
        return genElement(ast)
    } else if (ast.type === 'Text') {
        return genText(ast)
    } else if (ast.type === 'Interpolation') {
        return genText(ast.content)
    }
    return ''
}

// 生成元素
// 组合 tag,children,props
function genElement(el) {
    const tag = `'${el.tag}'`
    const children = genChildren(el)
    const props = genProps(el)
    const code = `this._c(${tag},${props}${children ? `,${children}` : ""})`
    return code
}

// 递归处理子节点
function genChildren(el) {
    const children = el.children

    if (children.length > 0) {
        // 1.处理文本
        // 2.处理插值节点
        // 3.genNode处理子节点
        return `[${children.map((c) => genNode(c)).join(',')}]`
    }
}

// 生成属性
function genProps(el) {...}

// 生成文本
function genText(text) {...}
```

## 效果

- [v0.4](https://github.com/tangyouhua/lab-tdd-vue/releases/tag/v0.4)
  - 所有代码生成单元测试通过

## 总结

- 递归 AST 节点（子节点），根据 AST 中解析出的类型生成代码

[1]: https://github.com/tangyouhua/lab-tdd-vue
[doc-tdd]: https://zh.wikipedia.org/wiki/%E6%B5%8B%E8%AF%95%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91
[doc-jestjs]: https://jestjs.io/zh-Hans/docs/getting-started
[api-js-proxy]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[doc-vuejs-how-reactivity-works-in-vue]: https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#how-reactivity-works-in-vue
[doc-vuejs-render-pipeline]: https://cn.vuejs.org/guide/extras/rendering-mechanism.html#render-pipeline
[doc-jestjs-fnimpl]: https://jestjs.io/zh-Hans/docs/jest-object#jestfnimplementation
[api-jestjs-jestusefaketimersfaketimersconfig]: https://jestjs.io/zh-Hans/docs/jest-object#jestusefaketimersfaketimersconfig
[api-jestjs-jestrunalltimers]: https://jestjs.io/zh-Hans/docs/jest-object#jestrunalltimers
[extension-vscode-jestrunner]: https://marketplace.visualstudio.com/items?itemName=firsttris.vscode-jest-runner
[doc-jslang-template-literals]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Template_literals
