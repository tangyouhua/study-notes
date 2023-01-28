# TDD Vue (6)

> 极客时间前端进阶训练营练习项目—打卡-Day13，2023-1-28

大纲

- [TDD Vue](#tdd-vue)
  - [目标](#目标)
  - [步骤](#步骤)
    - [编写 v0.5 版本](#编写-v05-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.5
  - 增加 examples，理解从原始 HTML，使用 Vue3，使用手写 Reactivity 的变化

### 编写 v0.5 版本

**为什么？**

通过一个示例，对比不同方法的区别，理解原生 HTML，Vue3 与手写的效果与代码上的区别。

**示例功能**

- 输入框输入文本，按钮文字跟随输入变化
- 点击按钮，按钮文字与文本框文字内容反转

**实现思路**

监听输入框 keyup 事件，监听按钮 click 事件

- 01 原生 HTML，在事件处理函数中直接处理
- 02 Vue3，输入框与按钮绑定响应式数据
- 03 手写 Vue，reactivity() 声明响应式数据，effect() 注册更新函数，在输入框与按钮监听事件中修改响应式数据，调用副作用函数更新

**核心代码**

原生 HTML

examples/01-native.html

```js
// data
const data = {
    message: 'Hello Vue3!!'
}

// view
const input = document.querySelector('input')
const button = document.querySelector('button')

input.addEventListener('keyup', function () {
    data.message = this.value
    update()
})
button.addEventListener('click', function () {
    data.message = data.message.split('').reverse().join('')
    update()
})

// update view
function update() {
    button.innerHTML = data.message
    input.value = data.message
}

// init
update()
```

使用 Vue3

examples/02-vue.html

```html
<div id="app">
    <!-- view -->
    <input v-model="data.message" />
    <button @click="onclick">{{data.message}}</button>
</div>
```

```js
Vue.createApp({
    setup() {
        // data
        const data = Vue.reactive({
            message: 'Hello Vue3!!'
        })
        function onclick() {
            data.message = data.message.split('').reverse().join('')
        }
        return {
            data,
            onclick
        }
    }
}).mount('#app')
```

手写 Vue 响应式与副作用函数

- 这里引入了 build/reactivity.js
- 该文件对 src/reactivity 目录下的代码进行了打包封装
- [ ] 此版本未提供完整的封装，待和下一个示例一起完成

examples/03-reactivity.html

```js
const { reactive, effect } = Reactivity
// use reactive data
const data = reactive({
    message: 'Hello Vue3!!'
})

const button = document.querySelector('button')
const input = document.querySelector('input')

function update() {
    button.innerHTML = data.message
    input.value = data.message
}

// register update as effect fn
effect(update)

input.addEventListener('keyup', function () {
    data.message = this.value
})
button.addEventListener('click', function () {
    data.message = data.message.split('').reverse().join('')
})
```

## 效果

- [v0.5](https://github.com/tangyouhua/lab-tdd-vue/releases/tag/v0.5)
  - 打开 examples 下面 01-native.html, 02-vue.html, 03-reactivity.html，示例功能表现一致

## 总结

- 通过一个简单示例，对比各种实现方案，了解了如何手工对 src 代码进行打包

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
