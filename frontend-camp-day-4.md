# Vue3 编译器

> 极客时间前端进阶训练营笔记—Day4，2023-1-9

## 是什么？

编译器是一种程序，将一种语言翻译为另外一种目标语言。

示例：

- 将 C 程序翻译成机器代码
- 将 Java 程序翻译成字节码
- 将模板代码翻译形成可执行代码

## 为什么？

在 Vue3 中，有以下场景需要使用编译器：

> [声明式渲染][1]：Vue 基于标准 HTML 拓展了一套模板语法，使得我们可以声明式地描述最终输出的 HTML 和 JavaScript 状态之间的关系。

- 声明式渲染
- 单文件组件 SFC

## 怎么实现？

**模板转 render 函数**

通过编译器实现转换，例如下面这段模板，

```html
<p>{{counter}}</p>
```

会转换为，

```js
(function anonymous(
) {
const _Vue = Vue
const { createElementVNode: _createElementVNode } = _Vue

const _hoisted_1 = /*#__PURE__*/_createElementVNode("h1", null, "Vue3 Compiler", -1 /* HOISTED */)

return function render(_ctx, _cache) {
  with (_ctx) {
    const { createElementVNode: _createElementVNode, toDisplayString: _toDisplayString, Fragment: _Fragment, openBlock: _openBlock, createElementBlock: _createElementBlock } = _Vue

    return (_openBlock(), _createElementBlock(_Fragment, null, [
      _hoisted_1,
      _createElementVNode("p", null, _toDisplayString(counter), 1 /* TEXT */)
    ], 64 /* STABLE_FRAGMENT */))
  }
}
})
```

这种做法的优点：

- 开发人员对 HTML 方式的描述更熟悉，开发效率更高
- 优化细节由编译器来实现：上面的代码是经过了编译器优化，手写容易出错

执行流程：

1. parse
2. [transform][8]：例如增加了 style 属性，在 [style.ts][7] 中会进行对应转换
3. generate

**执行时机**

- 预编译
  - webpack 打包
  - [单文件组件 SFC][2] vue-loader
- 运行时
  - [template 选项][3]
  - 挂载阶段：参见[组合式 API：生命周期钩子][4]

**性能优化**

Vue3 执行编译期性能优化。

- [`compileToFunction()` 方法][5] 进行编译
  - [调用 `new Function()` 转换代码为  render 函数][5]
  - [调用 `compile()` 函数][6]

SFC 编译工具--线上工具：https://sfc.vuejs.org/

- 查看 SFC 转换后的 JS 代码，尝试修改模板，可以帮助理解编译的规则
- 添加 style 可查看生成的 CSS 代码，例如添加 `<style scoped> h1{} </style>` 会生成 `h1[data-v-472cff63]{}`


**编译器优化策略**

通过 Vue Template Explorer 工具查看 template 生成的 JS：https://vue-next-template-explorer.netlify.app/

例如下面的 template：

```html
<div id="app">
    <h1>Vue3 Compiler</h1>
    <p>{{counter}}</p>
</div>
```

会翻译为，

```js
(_ctx, _push, _parent, _attrs) => {
  const _cssVars = { style: { color: _ctx.color }}
  _push(`<div${
    _ssrRenderAttrs(_mergeProps(_cssVars, _attrs, { id: "app" }))
  } scope-id><h1 scope-id>Vue3 Compiler</h1><p scope-id>${
    _ssrInterpolate(_ctx.counter)
  }</p></div>`)
}
```

有以下的优化策略：

- 空间换时间：静态部分缓存
- 精确更新节点：对 template 部分进行分析，只对变更的部分执行更新，实现机制 `patchFlags`，`shapeFlags`

[1]: https://cn.vuejs.org/guide/introduction.html#what-is-vue
[2]: https://cn.vuejs.org/guide/scaling-up/sfc.html
[3]: https://cn.vuejs.org/api/options-rendering.html#template
[4]: https://cn.vuejs.org/api/composition-api-lifecycle.html
[5]: https://github.com/vuejs/core/blob/0739f8909a0e56ae0fa760f233dfb8c113c9bde2/packages/vue/src/index.ts#L80
[5]: https://github.com/vuejs/core/blob/0739f8909a0e56ae0fa760f233dfb8c113c9bde2/packages/vue/src/index.ts#L16
[6]: https://github.com/vuejs/core/blob/0739f8909a0e56ae0fa760f233dfb8c113c9bde2/packages/compiler-dom/src/index.ts#L40
[7]: https://github.com/vuejs/core/blob/9c304bfe7942a20264235865b4bb5f6e53fdee0d/packages/compiler-dom/src/transforms/transformStyle.ts
[8]: https://github.com/vuejs/core/blob/09bb3e996ef17967022243a519d7dcc2921dd049/packages/compiler-core/src/compile.ts#L96
