# 动手实验 mini-router（3）

> 极客时间前端进阶训练营笔记—打卡-Day1，2023-1-16

## 目标

- 理解 Vue Router 设计思想
  - 构建单页应用 SPA
  - 建立路由和组件映射关系，显示对应组件
  - 路由跳转
- 动手实现
  - v0.2 实现 RouterLink 和 RouterView 组件

### 概念介绍

涉及的概念介绍：

#### defineComponent()

> [defineComponent()][api-vuejs-definecomponent] 在定义 Vue 组件时提供类型推导的辅助函数。

```ts
function defineComponent(
  component: ComponentOptions | ComponentOptions['setup']
): ComponentConstructor
```

在 mini-router 中，通过 `defineComponent()` 为 RouterLink 与 RouterView 组件提供了 `setup()` 函数。

#### 插槽内容与出口

> [插槽（Slots）内容与出口][doc-vuejs-slot-content-and-outlet]在某些场景中，我们可能想要为子组件传递一些模板片段，让子组件在它们的组件中渲染这些片段。

示例：

这里有一个 `<FancyButton>` 组件，可以像这样使用：

```html
<FancyButton>
  Click me! <!-- 插槽内容 -->
</FancyButton>
```

而 `<FancyButton>` 的模板是这样的：

```html
<button class="fancy-btn">
  <slot></slot> <!-- 插槽出口 -->
</button>
```

`<slot>` 元素是一个插槽出口 (slot outlet)，标示了父元素提供的插槽内容 (slot content) 将在哪里被渲染。

最终渲染出的 DOM 是这样：

```html
<button class="fancy-btn">Click me!</button>
```

在 mini-router 中，通过插槽与插槽出口实现了下面的转换：

```html
<router-link to="/">Go to Home | </router-link>
```

转换为，

```html
<a href="/">Go to Home | </a>
```

### 编写 v0.2 版本

**为什么？**

完成 RouterLink 与 RouterView 组件，实现 `<router-link>` 标签转换为 HTML。

**实现思路**

利用 Vue Component 相关的 API

- 重写 `defineComponent()` 中的 `setup()` 方法实现转换
- 调用 Vue `h()` 函数
  - RouterLink 转换为 `<a href="#yyy">xxx</a>`
  - RouterView 转换为 `<div>`

主要注意的是：

```html
<router-link to="yyy">
```

这里的 `yyy` 可能是响应式变量，在转换时需要通过 `unref()` 处理。

**核心代码**

RouterLink.js 转换 `<router-link>` 标签为超链接。

```js
export default defineComponent({
    // 定义 to 属性
    props: {
        to: {
            type: String,
            required: true
        }
    },
    setup(props, { slots }) {
        return () => {
            const to = unref(props.to) // 处理 to 为响应式数据
            return h( // 转换为 <a href="#yyy">xxx</a>
                'a',
                {
                    href: '#' + to
                },
                slots.default() // xxx
            )
        }
    }
})
```

RouterView.js 转换 `<router-view>` 标签为 `<div>`。

```js
import { defineComponent, h } from "vue"

export default defineComponent({
    setup() {
        return () => h('div', 'router-view')
    }
})
```

## 效果

- [v0.2](https://github.com/tangyouhua/lab-mini-router/releases/tag/v0.2)
  - 能够正常加载，并看到超链接 `Go to home` 为 `#/`, `Go to about` 为 `#/about`

## 总结

- 组件插槽内容与出口可用来获取 template 信息，利用 `h()` 转换路由链接

[doc-vuerouter]: https://router.vuejs.org/zh/introduction.html
[playground-vuerouter]: https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBIb21lIGZyb20gJy4vSG9tZS52dWUnXG5pbXBvcnQgQWJvdXQgZnJvbSAnLi9BYm91dC52dWUnXG5pbXBvcnQgTm90Rm91bmQgZnJvbSAnLi9Ob3RGb3VuZC52dWUnXG5cbmNvbnN0IHJvdXRlcyA9IHtcbiAgJy8nOiBIb21lLFxuICAnL2Fib3V0JzogQWJvdXRcbn1cblxuZXhwb3J0IGRlZmF1bHQge1xuICBkYXRhKCkge1xuICAgIHJldHVybiB7XG4gICAgICBjdXJyZW50UGF0aDogd2luZG93LmxvY2F0aW9uLmhhc2hcbiAgICB9XG4gIH0sXG4gIGNvbXB1dGVkOiB7XG4gICAgY3VycmVudFZpZXcoKSB7XG4gICAgICByZXR1cm4gcm91dGVzW3RoaXMuY3VycmVudFBhdGguc2xpY2UoMSkgfHwgJy8nXSB8fCBOb3RGb3VuZFxuICAgIH1cbiAgfSxcbiAgbW91bnRlZCgpIHtcbiAgICB3aW5kb3cuYWRkRXZlbnRMaXN0ZW5lcignaGFzaGNoYW5nZScsICgpID0+IHtcblx0XHQgIHRoaXMuY3VycmVudFBhdGggPSB3aW5kb3cubG9jYXRpb24uaGFzaFxuXHRcdH0pXG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxhIGhyZWY9XCIjL1wiPkhvbWU8L2E+IHxcbiAgPGEgaHJlZj1cIiMvYWJvdXRcIj5BYm91dDwvYT4gfFxuICA8YSBocmVmPVwiIy9ub24tZXhpc3RlbnQtcGF0aFwiPkJyb2tlbiBMaW5rPC9hPlxuICA8Y29tcG9uZW50IDppcz1cImN1cnJlbnRWaWV3XCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkhvbWUudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+SG9tZTwvaDE+XG48L3RlbXBsYXRlPiIsIkFib3V0LnZ1ZSI6Ijx0ZW1wbGF0ZT5cbiAgPGgxPkFib3V0PC9oMT5cbjwvdGVtcGxhdGU+IiwiTm90Rm91bmQudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+NDA0PC9oMT5cbjwvdGVtcGxhdGU+In0=
[course-vueschool-vue-router-4-for-everyone]: https://vueschool.io/courses/vue-router-4-for-everyone
[lab-vue-router]: https://github.com/tangyouhua/lab-vue-router
[doc-vuejs-components-registration]: https://cn.vuejs.org/guide/components/registration.html
[doc-vuejs-app-config-global-properties]: https://cn.vuejs.org/api/application.html#app-config-globalproperties
[api-vuejs-definecomponent]: https://cn.vuejs.org/api/general.html#definecomponent
[doc-vuejs-slot-content-and-outlet]: https://cn.vuejs.org/guide/components/slots.html#slot-content-and-outlet
