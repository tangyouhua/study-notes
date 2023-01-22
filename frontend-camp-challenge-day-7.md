# 手写 mini-vuex (5)

> 极客时间前端进阶训练营笔记—打卡-Day7，2023-1-22

大纲

- [手写 mini-vuex](#手写-mini-vuex)
  - [目标](#目标)
    - [概念介绍](#概念介绍)
      - [Vue 侦听器与 watch API](#vue-侦听器与-watch-api)
  - [步骤](#步骤)
    - [编写 v0.4 版本](#编写-v04-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.4
  - 实现严格模式

### 概念介绍

涉及的概念介绍：

#### Vue 侦听器与 watch API

> [侦听器][doc-vuejs-watchers]：在选项式 API 中，我们可以使用 watch 选项在每次响应式属性发生变化时触发一个函数。

示例：

```js
export default {
  data() {
    return {
      question: '',
      answer: 'Questions usually contain a question mark. ;-)'
    }
  },
  watch: {
    // 每当 question 改变时，这个函数就会执行
    question(newQuestion, oldQuestion) {
      if (newQuestion.includes('?')) {
        this.getAnswer()
      }
    }
  },
  methods: {
    async getAnswer() {
      this.answer = 'Thinking...'
      try {
        const res = await fetch('https://yesno.wtf/api')
        this.answer = (await res.json()).answer
      } catch (error) {
        this.answer = 'Error! Could not reach the API. ' + error
      }
    }
  }
}
```

template

```htm
<p>
  Ask a yes/no question:
  <input v-model="question" />
</p>
<p>{{ answer }}</p>
```

**深层侦听器**

> watch 默认是浅层的：被侦听的属性，仅在被赋新值时，才会触发回调函数——而嵌套属性的变化不会触发。如果想侦听所有嵌套的变更，你需要深层侦听器：
>
> [watch][api-vuejs-watch] 用于声明在数据更改时调用的侦听回调。

- `deep`：如果源是对象或数组，则强制深度遍历源，以便在深度变更时触发回调。详见深层侦听器。
- `flush`：调整回调的刷新时机。详见回调的触发时机及 `watchEffect()`。

在 mini-vuex 项目中，利用 watch API 深层监听器监视 `store.state` 变化实现严格模式。

在 watch 回调函数中根据当前 `options.strict` 以及 `commit` 内部变量给出对应提示。

## 步骤

1. 建立 GitHub 仓库：[lab-mini-vuex][1]
2. 按照 [Vite 文档][2]建立项目
3. 配置环境（Vite + Vue）
4. 实现各版本要求

### 编写 v0.4 版本

**为什么？**

提供 `strict: true` 选项，当开启此选项时，不允许用户只能调用 `commit()` 或 `dispatch()` 修改数据。

**实现思路**

- 为 `store` 提供 `strict: true`
- 修改 `commit()` 方法，通过 `_withCommit()` 进行提交
- 监听 `store.state`，如发现有提交未完成，且是严格模式则给出警告，拒绝本次提交

**核心代码**

Store 提供 `strict` 选项：

src/store/index.js

```js
export function createStore(options) {
  //...
  _commit: false, // 提交的标识符，如果通过commit方式修改状态，则设置为true
  //...
}
```

mini-vuex 提供 `_withCommit()` 方法，对提交进行封装：

src/mini-vuex/index.js

```js
export function createStore(options) {
  //...
  _withCommit(fn) { // fn就是用户设置的mutation执行函数
      this._commit = true
      fn()
      this._commit = false
  }
  //...
}
```

更新 `commit()` 方法，使用 `_withCommit()` 提交：

src/mini-vuex/index.js

```js
function commit(type, payload) {
    // ...
    // 要使用withCommit方式提交
    this._withCommit(() => {
        entry.call(this.state, this.state, payload)
    })
}
```

监听 store.state 变化，根据 `strict` 模式进行检查：

```js
function createStore(options) {
    // ...
    // strict模式
    if (options.strict) {
        // 监听store.state变化
        watch(store.state, () => {
            if (!store._commit) {
                console.warn('please use commit to mutate state')
            }
        }, {
            deep: true,
            flush: 'sync'
        })
    }
}
```

## 效果

- [v0.4](https://github.com/tangyouhua/lab-mini-vuex/releases/tag/v0.4)
  - 点击第一或第二行，两行内容同步变化，同时第三行显示双倍数值
  - 测试 `state` 严格模式下警告：例如 App.vue 中 `<p @click="$store.state.count++">`。点击第一行，会给出警告 "please use commit to mutate state"

## 总结

- 实现 strict 选项，保证使用 Store 时能够按照确定的方式提交，而不是可以外部随意修改

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
[doc-vuejs-watchers]: https://cn.vuejs.org/guide/essentials/watchers.html
[api-vuejs-watch]: https://cn.vuejs.org/api/options-state.html#watch
