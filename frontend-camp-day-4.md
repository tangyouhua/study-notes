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

## 怎么实现？

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

[1]: https://cn.vuejs.org/guide/introduction.html#what-is-vue
