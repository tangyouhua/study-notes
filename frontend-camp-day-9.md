# 动手实验 mini-router（1）

> 极客时间前端进阶训练营笔记—Day9，2023-1-14

大纲

- [动手实验 mini-router](#动手实验-mini-router)
  - [目标](#目标)
    - [概念介绍](#概念介绍)
    - [配置环境](#配置环境)
    - [编写 v0.0 版本](#编写-v00-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- 理解 Vue Router 设计思想
  - 构建单页应用 SPA
  - 建立路由和组件映射关系，显示对应组件
  - 路由跳转
- 动手实现
  - v0.0 搭建项目，感受 vue-router 使用
  - v0.1 创建 mini-router 插件
  - v0.2 实现 RouterLink 和 RouterView 组件
  - v0.3 监听导航事件并响应变化
  - v0.4 实现 RouterView

### 概念介绍

涉及的概念介绍：

> Vue Router 是 Vue.js 的官方路由。它与 Vue.js 核心深度集成，让用 Vue.js 构建单页应用变得轻而易举。功能包括：
>
> - 嵌套路由映射
> - 动态路由选择
> - 模块化、基于组件的路由配置
> - 路由参数、查询、通配符
> - 展示由 Vue.js 的过渡系统提供的过渡效果
> - 细致的导航控制
> - 自动激活 CSS 类的链接
> - HTML5 history 模式或 hash 模式
> - 可定制的滚动行为
> - URL 的正确编码
>
> 摘自 [Vue Router 官网介绍][doc-vuerouter]

通过[演练场][playground-vuerouter]快速了解基本概念的效果，例如下面这个简单的路由例子：

```js
<script>
import Home from './Home.vue'
import About from './About.vue'
import NotFound from './NotFound.vue'
const routes = {
  '/': Home,
  '/about': About
}
export default {
  data() {
    return {
      currentPath: window.location.hash
    }
  },
  computed: {
    currentView() {
      return routes[this.currentPath.slice(1) || '/'] || NotFound
    }
  },
  mounted() {
    window.addEventListener('hashchange', () => {
		  this.currentPath = window.location.hash
		})
  }
}
</script>
<template>
  <a href="#/">Home</a> |
  <a href="#/about">About</a> |
  <a href="#/non-existent-path">Broken Link</a>
  <component :is="currentView" />
</template>
```

更丰富的效果可以查看 [Vue Router4 视频教程][course-vueschool-vue-router-4-for-everyone]。
### 配置环境

描述配置环境步骤。

1. 新建项目仓库 [lab-vue-router][lab-vue-router]
2. 创建项目并添加依赖

创建项目 lab-vue-router

```shell
pnpm create vite #选择 vue, javascript
npm install
npm run dev
pnpm i vue-router #安装 vue rounter
```

### 编写 v0.0 版本

**为什么？**

v0.0 的目标是建立 Vue 项目，通过路由机制搭建一个最简单的单页面示例。

在后续版本中，在这个单页面应用的代码基础上，引入迷你 router 实现。达到类似效果的同时，理解 Vue Router 的核心机制。

**实现思路**

- 定义 `/`, `/about` 路由
- 提供 Home, About 视图
- 实现跳转

**核心代码**

1. App.vue 引入router

```html
<template>
  <img src="/vite.svg" class="logo" alt="Vite logo" />
  <HelloWorld msg="Hello Vue 3 + Vite" />

  <p>
    <!-- 使用router-link导航 -->
    <router-link to="/">Go to Home | </router-link>
    <router-link to="/about">Go to About</router-link>
  </p>

  <!-- 路由出口 -->
  <!-- 将来路由匹配的组件会渲染在这里 -->
  <router-view></router-view>
</template>
```

2. 创建 views 目录，定义 Home.vue, About.vue

```html
<template>
    <div>
        home page
    </div>
</template>
```

3. 创建 router 目录，导入页面组件并定义路由

```js
// 1.导入页面组件
import Home from '../views/Home.vue'
import About from '../views/About.vue'
import { createWebHashHistory, createRouter } from 'vue-router'

// 2.定义路由：每个路由对应一个组件
const routes = [
    { path: '/', component: Home },
    { path: '/about', component: About },
]

const router = createRouter({
    history: createWebHashHistory(),
    routes
})

export default router
```

## 效果

- [v0.0](https://github.com/tangyouhua/lab-vue-router/releases/tag/v0.0)
  - 显示 `Go to Home | Go to About` 链接
  - 点击链接，下方视图区对应显示为 `home page` 或 `about page`

## 总结

- Vue Router 是 Vue 默认的路由机制，用来实现 SPA 应用
- Router 核心内容为定义路由、管理页面以及路由跳转

[doc-vuerouter]: https://router.vuejs.org/zh/introduction.html
[playground-vuerouter]: https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBIb21lIGZyb20gJy4vSG9tZS52dWUnXG5pbXBvcnQgQWJvdXQgZnJvbSAnLi9BYm91dC52dWUnXG5pbXBvcnQgTm90Rm91bmQgZnJvbSAnLi9Ob3RGb3VuZC52dWUnXG5cbmNvbnN0IHJvdXRlcyA9IHtcbiAgJy8nOiBIb21lLFxuICAnL2Fib3V0JzogQWJvdXRcbn1cblxuZXhwb3J0IGRlZmF1bHQge1xuICBkYXRhKCkge1xuICAgIHJldHVybiB7XG4gICAgICBjdXJyZW50UGF0aDogd2luZG93LmxvY2F0aW9uLmhhc2hcbiAgICB9XG4gIH0sXG4gIGNvbXB1dGVkOiB7XG4gICAgY3VycmVudFZpZXcoKSB7XG4gICAgICByZXR1cm4gcm91dGVzW3RoaXMuY3VycmVudFBhdGguc2xpY2UoMSkgfHwgJy8nXSB8fCBOb3RGb3VuZFxuICAgIH1cbiAgfSxcbiAgbW91bnRlZCgpIHtcbiAgICB3aW5kb3cuYWRkRXZlbnRMaXN0ZW5lcignaGFzaGNoYW5nZScsICgpID0+IHtcblx0XHQgIHRoaXMuY3VycmVudFBhdGggPSB3aW5kb3cubG9jYXRpb24uaGFzaFxuXHRcdH0pXG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxhIGhyZWY9XCIjL1wiPkhvbWU8L2E+IHxcbiAgPGEgaHJlZj1cIiMvYWJvdXRcIj5BYm91dDwvYT4gfFxuICA8YSBocmVmPVwiIy9ub24tZXhpc3RlbnQtcGF0aFwiPkJyb2tlbiBMaW5rPC9hPlxuICA8Y29tcG9uZW50IDppcz1cImN1cnJlbnRWaWV3XCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkhvbWUudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+SG9tZTwvaDE+XG48L3RlbXBsYXRlPiIsIkFib3V0LnZ1ZSI6Ijx0ZW1wbGF0ZT5cbiAgPGgxPkFib3V0PC9oMT5cbjwvdGVtcGxhdGU+IiwiTm90Rm91bmQudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+NDA0PC9oMT5cbjwvdGVtcGxhdGU+In0=
[course-vueschool-vue-router-4-for-everyone]: https://vueschool.io/courses/vue-router-4-for-everyone
[lab-vue-router]: https://github.com/tangyouhua/lab-vue-router
