# 手写 mini-vuex (3)

> 极客时间前端进阶训练营笔记—打卡-Day5，2023-1-20

大纲

- [手写 mini-vuex](#手写-mini-vuex)
  - [目标](#目标)
    - [概念介绍](#概念介绍)
      - [Vue 状态管理](#vue-状态管理)
      - [Vuex 是什么](#vuex-是什么)
      - [Vue 响应式](#vue-响应式)
      - [Vuex 中的 Store 实例与 State 属性](#vuex-中的-store-实例与-state-属性)
  - [步骤](#步骤)
    - [编写 v0.2 版本](#编写-v02-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.2
  - 实现 Store 实例的 commit 和 dispatch 方法

### 概念介绍

涉及的概念介绍：

#### bind() 方法与 call() 方法

> [bind()][api-js-bind] 方法创建一个新的函数，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

示例

```js
const module = {
  x: 42,
  getX: function() {
    return this.x;
  }
};

const unboundGetX = module.getX;
console.log(unboundGetX()); // The function gets invoked at the global scope
// Expected output: undefined

const boundGetX = unboundGetX.bind(module);
console.log(boundGetX());
// Expected output: 42
```

在 mini-vuex 项目中，调用 dispatch 方法需要支持 `setTimeout` 以及 Promise 这些用法。

所以实现 dispatch 时，需要考虑绑定上下文为 Store。这时会用到 `call()` 方法

> [call()][api-js-call] 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。

示例

```js
function Product(name, price) {
  this.name = name;
  this.price = price;
}

function Food(name, price) {
  Product.call(this, name, price);
  this.category = 'food';
}

console.log(new Food('cheese', 5).name);
// Expected output: "cheese"
```

在 mini-vuex 项目中，实现 commit 和 dispatch 方法时，把 Store 作为 `call()`。

## 步骤

1. 建立 GitHub 仓库：[lab-mini-vuex][1]
2. 按照 [Vite 文档][2]建立项目
3. 配置环境（Vite + Vue）
4. 实现各版本要求

### 编写 v0.2 版本

**为什么？**

替换 Vuex 的 commit 与 dispatch 为自己的实现。

**实现思路**

- 在 mini-vuex/store/index.js 中实现自己的 commit 与 dispatch
- 根据传入的 options 中的 `mutations` 与 `actions`
  - 记录到 `store`
  - 根据调用 `commit` 与 `dispatch` 传入的 `type` 进行查找
  - 如能找到，通过 `call()` 调用；未能找到给出错误提示

**核心代码**

src/mini-vuex/index.js

记录 `mutations` 与 `actions`

```js
// Store实例
const store = {
    // 响应式state
    _state: reactive(options.state()),
    // 访问保护
    get state() {
        return this._state
    },
    set state(v) {
        console.error('please use replaceState() to reset state')
    },
    _mutations: options.mutations, // 记录mutations
    _actions: options.actions, // 记录actions
}
```

实现 commit() 方法：从 mutations 中查找并调用

```js
// commit(type, payload)
function commit(type, payload) {
    // 获取type对应的mutation
    const entry = this._mutations[type]
    if (!entry) {
        console.error(`unknown mutation type: ${type}`)
        return
    }
    entry.call(this.state, this.state, payload)
}
```

实现 dispatch() 方法：从 actions 中查找并调用

```js
// dispatch(type, payload)
function dispatch(type, payload) {
    // 获取用户编写的type对应的action
    const entry = this._actions[type]
    if (!entry) {
        console.error(`unknown action type: ${type}`)
        return
    }
    return entry.call(this, this, payload)
}
```

绑定 commit 与 dispatch 上下文为 store，并赋值给 store 对象：

```js
// 记录：如果直接在 store 中实现两个方法
// 则 strict 模式下 entry.call()的上下文为undefined
store.commit = commit.bind(store)
store.dispatch = commit.bind(store)
```

## 效果

- [v0.2](https://github.com/tangyouhua/lab-mini-vuex/releases/tag/v0.2)：与 v0.0 版本功能一致，浏览器 console 无报错

## 总结

- 替换 Vuex 的 commit 与 dispatch 实现
  - Store 中记录 actions 与 mutations，根据 commit 与 dispatch 传入的参数查找并调用 `call()`
  - 细节要关注调用的上下文，并考虑到用户可能的使用场景 `setTimeout` 与 Promise

[1]: https://github.com/tangyouhua/lab-mini-vuex
[2]: https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project
[doc-vuejs-state-management]: https://cn.vuejs.org/guide/scaling-up/state-management.html#simple-state-management-with-reactivity-api
[doc-vuejs-reactivity-in-depth]: https://cn.vuejs.org/guide/extras/reactivity-in-depth.html
[api-js-proxy]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[api-vuejs-reactivity-core-reactive]: https://cn.vuejs.org/api/reactivity-core.html#reactive
[api-vuex-createstore]: https://vuex.vuejs.org/zh/api/#createstore
[api-vuex-store-state]: https://vuex.vuejs.org/zh/api/#state-1
[api-js-bind]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
[api-js-call]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call
