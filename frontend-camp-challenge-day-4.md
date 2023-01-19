# 手写 mini-vuex

> 极客时间前端进阶训练营笔记—打卡-Day4，2023-1-19

大纲

- [手写 mini-vuex](#手写-mini-vuex)
  - [目标](#目标)
    - [概念介绍](#概念介绍)
      - [Vue 状态管理](#vue-状态管理)
      - [Vuex 是什么](#vuex-是什么)
      - [Vue 响应式](#vue-响应式)
      - [Vuex 中的 Store 实例与 State 属性](#vuex-中的-store-实例与-state-属性)
  - [步骤](#步骤)
    - [编写 v0.1 版本](#编写-v01-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.1
  - 创建 mini-vuex
  - 创建 Store 实例，提供响应式的 state

### 概念介绍

涉及的概念介绍：

#### Vue 响应式

> Vue 最标志性的功能就是其低侵入性的响应式系统。组件状态都是由响应式的 JavaScript 对象组成的。当更改它们时，视图会随即自动更新。
>
> [深入 Vue 响应式系统][doc-vuejs-reactivity-in-depth]

实现原理：通过追踪对象属性的读写提供响应式支持，Vue3 采用了 [Proxy][api-js-proxy] 实现。

在 mini-vuex 项目中，对于 Store 对象中的 `state` 通过 Vue 的 [reactive()][api-vuejs-reactivity-core-reactive] 做响应式支持。


#### Vuex 中的 Store 实例与 State 属性

Vuex 中的 Store 适用于构建一个中大型单页应用，可以帮助更好地在组件外部管理状态。

- 提供 [createStore()][api-vuex-createstore] 方法：创建一个 store 实例
- `state`：[API 文档][api-vuex-store-state]
  - 是 Vuex store 实例的根 state 对象
  - 如果传入返回一个对象的函数，其返回的对象会被用作根 state

> 参考 [Store 实例属性][api-vuex-store-state]文档。

在 mini-vuex 项目中，需要根据需求自行实现 Store 与 state 属性。

## 步骤

1. 建立 GitHub 仓库：[lab-mini-vuex][1]
2. 按照 [Vite 文档][2]建立项目
3. 配置环境（Vite + Vue）
4. 实现各版本要求

### 编写 v0.1 版本

**为什么？**

替换 Vuex 为 mini-vuex。

**实现思路**

- 创建 mini-router 组件，提供 `createStore()` 方法
- 对 Store 实例中的 `state` 提供响应式支持，并通过 `get`/`set` 方法对直接赋值进行拦截（后面提供 `commit`/`dispatch`）


**核心代码**

创建 mini-vuex 目录：

```shell
mkdir src/mini-vuex
```

src/mini-vuex/index.js 提供 createStore 方法：

- 创建组件，实现 `install()`
- 返回 `store` 实例
- 通过 `reactive()` 让 _state 支持响应式
- 提供 `get state()`, `set state(v)` 拦截直接访问 state 对象

```js
export function createStore(options) {
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
    }
    // 插件实现要求的install方法
    store.install = function (app) {
        const store = this
        // 注册$router
        app.config.globalProperties.$store = store
    }

    return store
}
```

使用 mini-vuex 组件：

src/store/index.js 替换如下

```js
import { createStore } from "../mini-vuex"
```

## 效果

- [v0.1](https://github.com/tangyouhua/lab-mini-vuex/releases/tag/v0.1)
  - 能够正常显示，点击第一行数据会报告 "\_ctx.$store.commit is not a function"，下一个版本实现
  - 如果直接修改 store，例如 App.vue 中 `<p @click="$store.state={}">` 会给出错误提示 "please use replaceState() to reset state"，证明组件加载和拦截 `set` 成功

## 总结

- 理解 Vuex 中的 Store 实例与 state 属性，提供自己的实现
  - `reactive()` 对 state 支持响应式
  - 拦截 get/set 确保用户对 Store 执行可预期的操作

[1]: https://github.com/tangyouhua/lab-mini-vuex
[2]: https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project
[doc-vuejs-state-management]: https://cn.vuejs.org/guide/scaling-up/state-management.html#simple-state-management-with-reactivity-api
[doc-vuejs-reactivity-in-depth]: https://cn.vuejs.org/guide/extras/reactivity-in-depth.html
[api-js-proxy]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[api-vuejs-reactivity-core-reactive]: https://cn.vuejs.org/api/reactivity-core.html#reactive
[api-vuex-createstore]: https://vuex.vuejs.org/zh/api/#createstore
[api-vuex-store-state]: https://vuex.vuejs.org/zh/api/#state-1
