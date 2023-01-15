# 动手实验 mini-router（2）

> 极客时间前端进阶训练营笔记—Day10，2023-1-15

## 目标

- 理解 Vue Router 设计思想
  - 构建单页应用 SPA
  - 建立路由和组件映射关系，显示对应组件
  - 路由跳转
- 动手实现
  - v0.1 创建 mini-router 插件

### 概念介绍

涉及的概念介绍：

#### 组件注册

Vue 提供了实例方法用来进行[组件注册][doc-vuejs-components-registration]，支持全局注册、局部注册。

> 我们可以使用 Vue 应用实例的 app.component() 方法，让组件在当前 Vue 应用中全局可用。

下面是本次实验用到的全局注册，组件采用了 Javascript 实现：

```js
import RouterLink from './RouterLink'

app.component('RouterLink', RouterLink)
```

#### app.config.globalProperties

> 全局属性对象 [app.config.globalProperties][doc-vuejs-app-config-global-properties] 是一个用于注册能够被应用内所有组件实例访问到的全局属性的对象。

**类型定义**

```ts
interface AppConfig {
  globalProperties: Record<string, any>
}
```

**用法**

```ts
app.config.globalProperties.msg = 'hello'
```

这使得 `msg` 在应用的任意组件模板上都可用，并且也可以通过任意组件实例的 `this` 访问到：

```ts
export default {
  mounted() {
    console.log(this.msg) // 'hello'
  }
}
```

### 编写 v0.1 版本

**为什么？**

新建 mini-router 插件，提供 `<router-link>`, `<router-view>` 标签。

在 Vue Router 项目基础上修改代码，改为使用 mini-router。

**实现思路**

创建插件

- createRouter
- 返回一个对象，实现 `install` 方法
- 完成任务
  - 注册两个组件 `RouterLink`, `RouterView`
  - 注册 `$router` 和 `$route`

**核心代码**

新建 src/mini-router 目录：

```shell
src/mini-router
├── RouterLink.js
├── RouterView.js
├── index.js
└── router.js
```

main.js 导入新创建的组件：

```js
//导入路由插件
import router from './mini-router'
```

编写 mini-router/index.js：

- 拷贝 router.js 内容到这里
- 修改部分如下

```js
// 1.导入页面组件
import Home from '../views/Home.vue'
import About from '../views/About.vue'
import { createRouter } from './router' // 改为应用当前目录下router.js

// 2.定义路由：每个路由对应一个组件
const routes = [
    { path: '/', component: Home },
    { path: '/about', component: About },
]

const router = createRouter({
    // history: createWebHashHistory(), // 临时注释此项
    routes
})

export default router
```

编写 mini-router/router.js

```js
import RouterLink from './RouterLink' // 引入RouterLink.js
import RouterView from './RouterView' // 引入RouterView.js

export function createRouter(options) {
    // router实例
    const router = {
        install(app) {
            const router = this

            // 1.注册两个全局组件
            app.component('RouterLink', RouterLink)
            app.component('RouterView', RouterView)

            // 2.注册$router
            app.config.globalProperties.$router = router //注册$router=router
        }
    }
    return router
}
```

## 效果

- [v0.1](https://github.com/tangyouhua/lab-vue-router/releases/tag/v0.1)
  - 能够加载两个组件
  - 此时还未完成插件功能，调试模式下会报告错误：“router/RouterLink.js' does not provide an export named 'default'”

## 总结

- Vue Router 是 Vue 默认的路由机制，用来实现 SPA 应用
- Router 核心内容为定义路由、管理页面以及路由跳转

[doc-vuerouter]: https://router.vuejs.org/zh/introduction.html
[playground-vuerouter]: https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBIb21lIGZyb20gJy4vSG9tZS52dWUnXG5pbXBvcnQgQWJvdXQgZnJvbSAnLi9BYm91dC52dWUnXG5pbXBvcnQgTm90Rm91bmQgZnJvbSAnLi9Ob3RGb3VuZC52dWUnXG5cbmNvbnN0IHJvdXRlcyA9IHtcbiAgJy8nOiBIb21lLFxuICAnL2Fib3V0JzogQWJvdXRcbn1cblxuZXhwb3J0IGRlZmF1bHQge1xuICBkYXRhKCkge1xuICAgIHJldHVybiB7XG4gICAgICBjdXJyZW50UGF0aDogd2luZG93LmxvY2F0aW9uLmhhc2hcbiAgICB9XG4gIH0sXG4gIGNvbXB1dGVkOiB7XG4gICAgY3VycmVudFZpZXcoKSB7XG4gICAgICByZXR1cm4gcm91dGVzW3RoaXMuY3VycmVudFBhdGguc2xpY2UoMSkgfHwgJy8nXSB8fCBOb3RGb3VuZFxuICAgIH1cbiAgfSxcbiAgbW91bnRlZCgpIHtcbiAgICB3aW5kb3cuYWRkRXZlbnRMaXN0ZW5lcignaGFzaGNoYW5nZScsICgpID0+IHtcblx0XHQgIHRoaXMuY3VycmVudFBhdGggPSB3aW5kb3cubG9jYXRpb24uaGFzaFxuXHRcdH0pXG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxhIGhyZWY9XCIjL1wiPkhvbWU8L2E+IHxcbiAgPGEgaHJlZj1cIiMvYWJvdXRcIj5BYm91dDwvYT4gfFxuICA8YSBocmVmPVwiIy9ub24tZXhpc3RlbnQtcGF0aFwiPkJyb2tlbiBMaW5rPC9hPlxuICA8Y29tcG9uZW50IDppcz1cImN1cnJlbnRWaWV3XCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkhvbWUudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+SG9tZTwvaDE+XG48L3RlbXBsYXRlPiIsIkFib3V0LnZ1ZSI6Ijx0ZW1wbGF0ZT5cbiAgPGgxPkFib3V0PC9oMT5cbjwvdGVtcGxhdGU+IiwiTm90Rm91bmQudnVlIjoiPHRlbXBsYXRlPlxuICA8aDE+NDA0PC9oMT5cbjwvdGVtcGxhdGU+In0=
[course-vueschool-vue-router-4-for-everyone]: https://vueschool.io/courses/vue-router-4-for-everyone
[lab-vue-router]: https://github.com/tangyouhua/lab-vue-router
[doc-vuejs-components-registration]: https://cn.vuejs.org/guide/components/registration.html
[doc-vuejs-app-config-global-properties]: https://cn.vuejs.org/api/application.html#app-config-globalproperties
