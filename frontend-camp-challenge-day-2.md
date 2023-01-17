# 动手实验 mini-router（4）

> 极客时间前端进阶训练营笔记—打卡-Day2，2023-1-17

## 目标

- 理解 Vue Router 设计思想
  - 构建单页应用 SPA
  - 建立路由和组件映射关系，显示对应组件
  - 路由跳转
- 动手实现
  - v0.3 监听导航事件并响应变化
  - v0.4 实现 RouterView

### 概念介绍

涉及的概念介绍：

#### hashchange事件

> 当 URL 的片段标识符更改时，将触发 [hashchange 事件][api-window-hashchange-event] (跟在＃符号后面的 URL 部分，包括＃符号)

使用示例：

```js
window.addEventListener('hashchange', function() {
  console.log('The hash has changed!')
}, false);
```

或者

```js
function locationHashChanged() {
  if (location.hash === '#cool-feature') {
    console.log("You're visiting a cool feature!");
  }
}

window.onhashchange = locationHashChanged;
```

在 mini-router 中，通过监听 hashchange 事件，保存 current URL 并触发 RouterView 更新。

#### getCurrentInstance()

在 Vue3 之前的版本中，提供了 `getCurrentInstance()` 方法获取当前实例。

> Vue3 认为此方法是一种反模式（参见 ["Why getCurrentInstance is descriped as anti pattern in application code?"][qa-vuejs-getcurrentinstance-to-internal]），现在为内部 API。


在 mini-router 中，通过 `getCurrentInstance()` 方法获取实例中的 `router`

```js
const { proxy: { $router } } = getCurrentInstance()
```

#### RouterOptions

> [RouterOptions][api-vuejs-routeroptions] 创建 Router 时传递的原始配置对象。只读的。

在 mini-router 中，通过 RouterOptions 的 routes 属性对 URL 进行匹配。

```js
const route = $router.options.routes.find(
  (route) => route.path === unref($router.current)
)
```

**注意**：这里用 `unref()` 方法处理响应式数据。

### 编写 v0.3 版本

**为什么？**

监听导航事件并响应变化。

**实现思路**

通过 hashchange 事件监听导航变化并实现 RouterView 更新。

**核心代码**

router.js

```js
// router实例
const router = {
    options, // 保存配置项
    current: ref(window.location.hash.slice(1) || '/'),
    // ...
}

// 监听hashchange事件
window.addEventListener('hashchange', () => {
    // 变化保存到current并触发RouterView更新
    router.current.value = window.location.hash.slice(1)
    console.log(router.current.value)
})
```

### 编写 v0.4 版本

**为什么？**

实现 RouterView，根据点击的链接跳转到对应视图 Home.vue, About.vue。 

**实现思路**

- 根据 hashchange 事件处理时记录的 value，定位到匹配的组件
  - 获取当前实例的 `$router`
  - 根据当前 URL 与 `RouterOptions.routes` 进行组件匹配
- 通过 Vue `h()` 函数，对匹配到的 `<router-view>` 进行渲染
- 如果未匹配成果，则用 `<div>` 进行渲染

**核心代码**

App.vue

```html
<template>
  <!-- 路由出口 -->
  <!-- 将来路由匹配的组件会渲染在这里 -->
  <router-view></router-view>
</template>
```

RouterView.js

```js
export default defineComponent({
    setup() {
        return () => {
            // 获取组件实例
            const { proxy: { $router } } = getCurrentInstance()
            // 1. 获取想要渲染的组件
            // 1.1 获取配置routes
            // 1.2 通过current地址找到匹配的项
            let component;
            const route = $router.options.routes.find(
                (route) => route.path === unref($router.current)
            )
            // 找到匹配的组件
            if (route) {
                component = route.component
                return h(component, 'router-view')
            } else {
                console.warn('no match component')
                return h('div', '')
            }
        }
    }
})
```

## 效果

- [v0.3](https://github.com/tangyouhua/lab-mini-router/releases/tag/v0.3)
  - 能够正常加载，点击超链接会在调试 console 中输出 `#/` 或 `#/about`
- [v0.4](https://github.com/tangyouhua/lab-mini-router/releases/tag/v0.4)
  - 能够正常加载，点击超链接会在链接下方显示 "home page" 或 "about page" 内容

## 总结

- 处理 hashchange 事件，对路由根据 URL 进行匹配，将内容渲染到 `<router-view>`

[doc-vuerouter]: https://router.vuejs.org/zh/introduction.html
[playground-vuerouter]: https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBIb21lIGZyb20gJy4vSG9tZS52dWUnXG5pbXBvcnQgQWJvdXQgZnJvbSAnLi9BYm91dC52dWUnXG5pbXBvcnQgTm90Rm91bmQgZnJvbSAnLi9Ob3RGb3VuZC52dWUnXG5cbmNvbnN0IHJvdXRlcyA9IHtcbiAgJy8nOiBIb21lLFxuICAnL2Fib3V0JzogQWJvdXRcbn1cblxuZXhwb3J0IGRlZmF1bHQge1xuICBkYXRhKCkge1xuICAgIHJldHVybiB7XG4gICAgICBjdXJyZW50UGF0aDogd2luZG93LmxvY2F0aW9uLmhhc2hcbiAgICB9XG4gIH0sXG4gIGNvbXB1dGVkOiB7XG4gICAgY3VycmVudFZpZXcoKSB7XG4gICAgICByZXR1cm4gcm91dGVzW3RoaXMuY3VycmVudFBhdGguc2xpY2UoMSkgfHwgJy8nXSB8fCBOb3RGb3VuZFxuICAgIH1cbiAgfSxcbiAgbW91bnRlZCgpIHtcbiAgICB3aW5kb3cuYWRkRXZlbnRMaXN0ZW5lcignaGFzaGNoYW5nZScsICgpID0+IHtcblx0XHQgIHRoaXMuY3VycmVudFBhdGggPSB3aW5kb3cubG9jYXRpb24uaGFzaFxuXHRcdH0pXG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxhIGhyZWY9XCIjL1wiPkhvbWU8L2E+IHxcbiAgPGEgaHJlZj1cIiMvYWJvdXRcIj5BYm91dDwvYT4gfFxuICA8YSBocmVmPVwiIy9ub24tZXhpc3RlbnQtcGF0aFwiPkJyb2tlbiBMaW5rPC9hPlxuICA8Y29tcG9uZW50IDppcz1cImN1cnJlbnRWaWV3XCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkhvbWUudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+SG9tZTwvaDE+XG48L3RlbXBsYXRlPiIsIkFib3V0LnZ1ZSI6Ijx0ZW1wbGF0ZT5cbiAgPGgxPkFib3V0PC9oMT5cbjwvdGVtcGxhdGU+IiwiTm90Rm91bmQudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+NDA0PC9oMT5cbjwvdGVtcGxhdGU+In0=
[course-vueschool-vue-router-4-for-everyone]: https://vueschool.io/courses/vue-router-4-for-everyone
[lab-vue-router]: https://github.com/tangyouhua/lab-vue-router
[doc-vuejs-components-registration]: https://cn.vuejs.org/guide/components/registration.html
[doc-vuejs-app-config-global-properties]: https://cn.vuejs.org/api/application.html#app-config-globalproperties
[api-vuejs-definecomponent]: https://cn.vuejs.org/api/general.html#definecomponent
[doc-vuejs-slot-content-and-outlet]: https://cn.vuejs.org/guide/components/slots.html#slot-content-and-outlet
[doc-vuerouter]: https://router.vuejs.org/zh/introduction.html
[playground-vuerouter]: https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBIb21lIGZyb20gJy4vSG9tZS52dWUnXG5pbXBvcnQgQWJvdXQgZnJvbSAnLi9BYm91dC52dWUnXG5pbXBvcnQgTm90Rm91bmQgZnJvbSAnLi9Ob3RGb3VuZC52dWUnXG5cbmNvbnN0IHJvdXRlcyA9IHtcbiAgJy8nOiBIb21lLFxuICAnL2Fib3V0JzogQWJvdXRcbn1cblxuZXhwb3J0IGRlZmF1bHQge1xuICBkYXRhKCkge1xuICAgIHJldHVybiB7XG4gICAgICBjdXJyZW50UGF0aDogd2luZG93LmxvY2F0aW9uLmhhc2hcbiAgICB9XG4gIH0sXG4gIGNvbXB1dGVkOiB7XG4gICAgY3VycmVudFZpZXcoKSB7XG4gICAgICByZXR1cm4gcm91dGVzW3RoaXMuY3VycmVudFBhdGguc2xpY2UoMSkgfHwgJy8nXSB8fCBOb3RGb3VuZFxuICAgIH1cbiAgfSxcbiAgbW91bnRlZCgpIHtcbiAgICB3aW5kb3cuYWRkRXZlbnRMaXN0ZW5lcignaGFzaGNoYW5nZScsICgpID0+IHtcblx0XHQgIHRoaXMuY3VycmVudFBhdGggPSB3aW5kb3cubG9jYXRpb24uaGFzaFxuXHRcdH0pXG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxhIGhyZWY9XCIjL1wiPkhvbWU8L2E+IHxcbiAgPGEgaHJlZj1cIiMvYWJvdXRcIj5BYm91dDwvYT4gfFxuICA8YSBocmVmPVwiIy9ub24tZXhpc3RlbnQtcGF0aFwiPkJyb2tlbiBMaW5rPC9hPlxuICA8Y29tcG9uZW50IDppcz1cImN1cnJlbnRWaWV3XCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkhvbWUudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+SG9tZTwvaDE+XG48L3RlbXBsYXRlPiIsIkFib3V0LnZ1ZSI6Ijx0ZW1wbGF0ZT5cbiAgPGgxPkFib3V0PC9oMT5cbjwvdGVtcGxhdGU+IiwiTm90Rm91bmQudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+NDA0PC9oMT5cbjwvdGVtcGxhdGU+In0=
[course-vueschool-vue-router-4-for-everyone]: https://vueschool.io/courses/vue-router-4-for-everyone
[lab-vue-router]: https://github.com/tangyouhua/lab-vue-router
[doc-vuejs-components-registration]: https://cn.vuejs.org/guide/components/registration.html
[doc-vuejs-app-config-global-properties]: https://cn.vuejs.org/api/application.html#app-config-globalproperties
[api-vuejs-definecomponent]: https://cn.vuejs.org/api/general.html#definecomponent
[doc-vuejs-slot-content-and-outlet]: https://cn.vuejs.org/guide/components/slots.html#slot-content-and-outlet
[api-window-hashchange-event]: https://developer.mozilla.org/zh-CN/docs/Web/API/Window/hashchange_event
[qa-vuejs-getcurrentinstance-to-internal]: https://github.com/vuejs/docs/issues/1422#issuecomment-1032120675
[api-vuejs-routeroptions]: https://router.vuejs.org/zh/api/#routeroptions
