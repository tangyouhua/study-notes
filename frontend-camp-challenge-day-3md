# 手写 mini-vuex

> 极客时间前端进阶训练营笔记—打卡-Day3，2023-1-18

大纲

- [手写 mini-vuex](#手写-mini-vuex)
  - [目标](#目标)
    - [概念介绍](#概念介绍)
      - [Vue 状态管理](#vue-状态管理)
      - [Vuex 是什么](#vuex-是什么)
  - [步骤](#步骤)
    - [配置环境](#配置环境)
    - [编写 v0.0 版本](#编写-v00-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.0
  - 理解 Vuex 功能与解决的问题
  - 搭建初始项目，了解需要手写实现的 API

### 概念介绍

涉及的概念介绍：

#### Vue 状态管理

Vue 中的[状态管理][doc-vuejs-state-management]

> 理论上来说，每一个 Vue 组件实例都已经在“管理”它自己的响应式状态了。
>
> 它是一个独立的单元，由以下几个部分组成：
>
> - 状态：驱动整个应用的数据源；
> - 视图：对状态的一种声明式映射；
> - 交互：状态根据用户在视图中的输入而作出相应变更的可能方式。
>
> 然而，当我们有多个组件共享一个共同的状态时，就没有这么简单了：
>
> - 多个视图可能都依赖于同一份状态。
> - 来自不同视图的交互也可能需要更改同一份状态。

#### Vuex 是什么

Vue3 目前用的 [Vuex][doc-vuex-introduction] 但已不维护，推荐 [Pinia][doc-pinia-introduction]

> Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式 + 库。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
>
> Vuex 可以帮助我们管理共享状态，并附带了更多的概念和框架。

[doc-vuex-introduction]: https://vuex.vuejs.org/zh/
[doc-pinia-introduction]: https://pinia.vuejs.org/zh/introduction.html

**核心概念**

- State：单一状态树
- Getter：从 store 中的 state 中派生出一些状态
- Mutation：更改 Vuex 的 store 中的状态的唯一方法是提交 mutation
- Action：Action 类似于 mutation，不同在于
  - Action 提交的是 mutation，而不是直接变更状态
  - Action 可以包含任意异步操作
- Module：Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割

## 步骤

1. 建立 GitHub 仓库：[lab-mini-vuex][1]
2. 按照 [Vite 文档][2]建立项目
3. 配置环境（Vite + Vue）
4. 实现各版本要求

### 配置环境

**安装项目依赖**

```shell
npm install vuex # 安装版本4.1.0
```

### 编写 v0.0 版本

**为什么？**

搭建项目骨架，使用 Vuex 完成基本的功能。

**实现思路**

- 引入 Vuex 库
- 创建 Store 组件，并定义 count
- 调用同步增加、异步增加 count

**核心代码**

新建 Store 组件

```shell
mkdir src/store
```

添加 `index.js` 内容如下：

```js
import { createStore } from "vuex"

// 创建Store实例
const store = createStore({
    state() {
        return {
            count: 1
        }
    },
    mutations: {
        // state从何而来？
        add(state) {
            state.count++
        }
    },
    getters: {
        doubleCounter(state) {
            return state.count * 2
        }
    },
    actions: {
        add({ commit }) {
            setTimeout(() => {
                commit('add')
            }, 1000)
        }
    }
})

export { store }
```

App 注册 store 组件：

main.js

```js
import { store } from './store'

createApp(App).use(store).mount('#app')
```

修改 `App.vue` 模板，支持：

- 点击同步调用 `add`，增加 count
- 点击异步调用 `add`，增加 count

```html
<template>
  <h1>mini-vuex</h1>
  <p @click="$store.commit('add')">count: {{ $store.state.count }}</p>
  <p @click="$store.dispatch('add')">count: {{ $store.state.count }}</p>
</template>
```

## 效果

- [v0.0](https://github.com/tangyouhua/lab-mini-vuex/releases/tag/v0.0) 显示两行数据，从 0 开始
  - 点击第一行同步增加计数，两行内容同步变化
  - 点击第二行异步增加计数，两行内容同步变化

## 总结

- Vuex 是一个可以用作 Vue 项目组件间状态管理的库，提供的 API 可以有效解决跨组件层级数据同步的问题

[1]: https://github.com/tangyouhua/lab-mini-vuex
[2]: https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project
[doc-vuejs-state-management]: https://cn.vuejs.org/guide/scaling-up/state-management.html#simple-state-management-with-reactivity-api
