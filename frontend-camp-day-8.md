# 动手实验 mini-vue（4）

> 极客时间前端进阶训练营笔记—Day8，2023-1-13

## 大纲

- [x] v0.1
  - 创建实例 `createApp(App)`
  - `app.mount('#app')`
- [x] v0.2
  - 创建渲染器 renderer：`createRender()`
  - 提供 render：`renderer.render`
- [x] v0.3
  - 引入 reactive 机制
  - 实现依赖搜集
- [x] v0.4
  - 引入虚拟 DOM：`createVNode`
  - patch 算法

## 目标

- 完成 v0.4 版本

### 编写 v0.4 版本

**为什么？**

componentUpdateFn() 每次都是全量更新，

- 全量更新工作效率不高：实现简化版 VNode patch 算法
- 不能重复利用现有节点：引入虚拟 DOM

**实现思路**

使用[虚拟 DOM][doc-vuejs-virtual-dom]

- 找到真正的变化点
- 在第二次更新的时候合并变化点

虚拟 DOM 介绍：

> - 虚拟 DOM (Virtual DOM，简称 VDOM) 是一种编程概念，意为将目标所需的 UI 通过数据结构“虚拟”地表示出来，保存在内存中，然后将真实的 DOM 与之保持同步。
> - 这个概念是由 React 率先开拓，随后在许多不同的框架中都有不同的实现，当然也包括 Vue。
> - 与其说虚拟 DOM 是一种具体的技术，不如说是一种模式，所以并没有一个标准的实现。
> - 一个运行时渲染器将会遍历整个虚拟 DOM 树，并据此构建真实的 DOM 树。这个过程被称为**挂载 (mount)**。
> - 如果我们有两份虚拟 DOM 树，渲染器将会有比较地遍历它们，找出它们之间的区别，并应用这其中的变化到真实的 DOM 上。这个过程被称为**更新 (patch)**，又被称为“比对”(diffing) 或“协调”(reconciliation)。
> - 虚拟 DOM 带来的主要收益是它让开发者能够灵活、声明式地创建、检查和组合所需 UI 的结构，同时只需把具体的 DOM 操作留给渲染器去处理。

**核心代码**

改造标题数据 `data()` 支持数组：

```js
createApp({
    data() {
        return {
            title: ['Hello', 'mini-vue!', 'again'] // 标题数组
        }
    },
    render() {
        // 支持数组
        // 返回VNode
        if (Array.isArray(this.title)) {
            return createVNode('h3', {}, this.title.map(t => createVNode('p', {}, t)))
        } else {
            return createVNode('h3', {}, this.title)
        }
    },
    mounted() {
        setTimeout(() => {
            this.title = ['This is a new title!', 'This is the second title!'] // 更新标题数组
        }, 2000)
    }
}).mount('#app')
```

提供 `createVNode()` 方法创建 VNode：

```js
export function createVNode(type, props, children) {
    // 返回虚拟DOM
    return {type, props, children}
}
```

更新 `createApp()` 调用传入的挂载 `mount()`方法：

```js
// 创建App实例
export function createApp(rootComponent) {
    //接收根组件，返回App实例
    const app = ensureRenderer().createApp(rootComponent)
    const mount = app.mount
    app.mount = function (selectorOrContainer) {
        const container = document.querySelector(selectorOrContainer)
        mount(container)
    }
    return app
}
```

为渲染器 renderer 对 vnode 执行更新 `patch()`:

```js
const render = (vnode, container) => {
    // 如果存在vnode则为mount或patch，否则unmount
    if (vnode) {
        patch(container._vnode || null, vnode, container)
    }

    container._vnode = vnode
}
```

`patch()` 的主要实现逻辑

1. 判断元素类型：原生节点 Element 或者 组件 Component
2. 针对元素提供 `mount()` 与 `patch()`

在 `mount()` 阶段根据 Node 类型，处理文本和需要递归的节点类型。

```js
const processElement = (n1, n2, container) => {
    if (n1 == null) {
        // 创建阶段
        mountElement(n2, container)
    } else {
        // 更新阶段
        patchElement(n1, n2)
    }
}

const mountElement = (vnode, container) => {
    const el = (vnode.el = hostCreateElement(vnode.type))

    // TODO: 处理properties

    // children如果为文本
    if (typeof vnode.children === 'string') {
        el.textContent = vnode.children
    } else {
        // 数组需要递归创建
        vnode.children.forEach(child => patch(null, child, el))
    }

    // 插入元素
    hostInsert(el, container)
}
```

在 `patch()` 阶段比较并找出变化点并更新，

```js
const patchElement = (n1, n2) => {
    // 获取要更新的元素节点
    const el = n2.el = n1.el

    // 更新type相同的节点，实际上还要考虑key
    if (n1.type === n2.type) {
        // 获取双方子元素
        const oldCh = n1.children
        const newCh = n2.children

        // 根据双方子元素情况做不同处理
        if (typeof oldCh === 'string') {
            if (typeof newCh === 'string') {
                // 对比双方文本，如果变化则更新
                if (oldCh !== newCh) {
                    hostSetElementText(el, newCh)
                }
            } else {
                // 替换文本为一组子元素
                hostSetElementText(el, '')
                newCh.forEach(v => patch(null, v, el))
            }
        } else {
            if (typeof newCh === 'string') {
                // 之前是子元素数组，变化之后是文本内容
                hostSetElementText(el, newCh)
            } else {
                // 变化前后都是子元素数组
                updateChildren(oldCh, newCh, el)
            }
        }
    }
}
```

简化版本的更新算法：

```js
const updateChildren = (oldCh, newCh, parentElm) => {
    // A B C D E
    // A C D E
    // 获取较短的数组的长度
    const len = Math.min(oldCh.length, newCh.length)
    for (let i = 0; i < len; i++) {
        patch(oldCh[i], newCh[i])
    }
    // 获取较长数组中剩余的部分
    if (newCh.length > oldCh.length) {
        // 新数组较长，剩余的批量创建追加
        newCh.slice(len).forEach(child => patch(null, child, parentElm))
    } else {
        // 老数组较长，剩余的批量删除
        oldCh.slice(len).forEach(child => hostRemove(child.el))
    }
}
```

## 效果

- [v0.4][10]
  - 页面标题 data.title 支持数组，页面加载 2 秒后进行标题更新
  - 标题更新逻辑1 `[a, b] + [c, d]` => `[c, d]`
  - 标题更新逻辑 `[a, b, c] + [d, e]` => `[d, e]`
  - 标题更新逻辑 `[a, b] + [c, d, e]` => `[c, d, e]`

## 总结

- 处理 mount 与 patch 有很多细节要考虑，mini-vue 练习了最基本的逻辑
- 在实际动手过程中，建议对整个机制和基本概念有完整的了解，效果会更好

[1]: https://github.com/tangyouhua/lab-mini-vue
[2]: https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project
[3]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.0
[4]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.1
[5]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.2
[6]: https://cn.vuejs.org/guide/introduction.html#what-is-vue
[7]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[8]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
[9]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.3
[10]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.3
[doc-vuejs-virtual-dom]: https://cn.vuejs.org/guide/extras/rendering-mechanism.html#virtual-dom
