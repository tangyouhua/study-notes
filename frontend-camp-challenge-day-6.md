# 手写 mini-vuex (4)

> 极客时间前端进阶训练营笔记—打卡-Day6，2023-1-21

大纲

- [手写 mini-vuex](#手写-mini-vuex)
  - [目标](#目标)
    - [概念介绍](#概念介绍)
      - [Object.defineProperty()](#objectdefineproperty)
      - [Vue computed 计算属性](#vue-computed-计算属性)
  - [步骤](#步骤)
    - [编写 v0.3 版本](#编写-v03-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.3
  - 实现 Store 实例的 getters

### 概念介绍

涉及的概念介绍：

#### Object.defineProperty()

> [Object.defineProperty()][api-js-object-defineproperty] 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

示例：

```js
const object1 = {};

Object.defineProperty(object1, 'property1', {
  value: 42,
  writable: false
});

object1.property1 = 77;
// Throws an error in strict mode

console.log(object1.property1);
// Expected output: 42
```

在 mini-vuex 项目中，使用 Object.defineProperty() 只提供 `get()` 方法，实现 `getters` 对象只读。

```js
const getters = {}
Object.defineProperty(getters, key, {
  get() { return bValue; },
  // set(newValue) { bValue = newValue; },
});
```

#### Vue computed 计算属性

> [计算属性][doc-vuejs-computed]有以下特点，
> 
> - 用来描述依赖响应式状态的复杂逻辑
> - 若我们将同样的函数定义为一个方法而不是计算属性，两种方式在结果上确实是完全相同的，然而，不同之处在于计算属性值会基于其响应式依赖被缓存。

要了解计算属性的用法，可以尝试这个[5分钟教程][tut-vuejs-computed]:

- 通过计算属性自动计算输入的字符长度、实现列表反转；
- 使用计算属性需要注意的事项，例如不要修改原有的数据 `[...arr].reverse()`。

在 mini-vuex 项目中，利用计算属性的缓存特性实现 `getters` 的读取效率优化。

## 步骤

1. 建立 GitHub 仓库：[lab-mini-vuex][1]
2. 按照 [Vite 文档][2]建立项目
3. 配置环境（Vite + Vue）
4. 实现各版本要求

### 编写 v0.3 版本

**为什么？**

替换 Vuex 的 getters 为自己的实现。

**实现思路**

- 在 mini-vuex/store/index.js 中实现自己的 getters
  - 在 `store` 中创建 `getters` 对象
  - 根据调用传入的 key 在 `getters` 中查找
  - 如能找到，通过 `call()` 调用；未能找到给出错误提示
- 需求：`store` 内的 `getters` 不允许用户设置
  - 用 `Object.defineProperty` 实现只读
- 优化：每次调用都查找 `getters` 效率低
  - 用 Vue `computed` 计算属性实现优化 

**核心代码**

查看已实现的 `doubleCounter()` 函数。

```js
const store = createStore {
  // ...
  getters: {
    doubleCounter(state) {
        return state.count * 2
    }
  }
}
```

调用 `getters` 中的 doubleCounter 显示双倍结果。

src/mini-vuex/App.vue

```htm
<template>
  <p>doubleCounter: {{ $store.getters.doubleCounter }}</p>
</template>
```

实现自己的 `getter()`

src/mini-vuex/index.js

```js
// 定义store.getters
store.getters = {}

// 遍历用户定义getters
Object.keys(options.getters).forEach(key => {
    // 定义计算属性
    // 优化getters读效率
    const result = computed(() => {
        // 函数 key,例如 doubleCounter
        const getter = options.getters[key]
        if (getter) {
            return getter.call(store, store.state)
        } else {
            console.error('unknown getter type:' + key)
            return ''
        }
    })
    // 动态定义store.getters.xxx
    // 值来自于用户定义的getter函数的返回值
    Object.defineProperty(store.getters, key, {
        // 只读
        get() {
            return result
        }
    })
})
```

## 效果

- [v0.3](https://github.com/tangyouhua/lab-mini-vuex/releases/tag/v0.3)
  - 点击第一或第二行，两行内容同步变化，同时第三行显示双倍数值
  - 修改 App.vue 中的 getters 函数名，如未定义，则第三行不显示数值

## 总结

- 涉及 getters 这样的 key/value 存储与查找，需要考虑
  - key 不存在的处理
  - 数据量大的情况下，优化查找的效率
  - 存储的数据结构本身要做到只读，即不允许外部人为修改或替换

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
[api-js-object-defineproperty]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
[tut-vuejs-computed]: https://vueschool.io/lessons/computed-properties-in-vue-3
[doc-vuejs-computed]: https://cn.vuejs.org/guide/essentials/computed.html
