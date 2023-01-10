# 动手实验 mini-vue（1）

> 极客时间前端进阶训练营笔记—Day5，2023-1-10

## 大纲

- v0.1
  - 创建实例 `createApp(App)`
  - `app.mount('#app')
- v0.2
  - 创建渲染器 renderer：`createRender()`
  - 提供 render：`renderer.render`
- v0.3
  - 引入 reactive 机制
  - 实现依赖搜集
- v0.4
  - 引入虚拟 DOM：`createVNode`
  - patch 算法

## 目标

- 建立实验项目 lab-mini-vue
- 完成 v0.1 版本

## 步骤

1. 建立 GitHub 仓库：[lab-mini-vue][1]
2. 按照 [Vite 文档][2]建立项目
3. 配置环境（Vite + Vue + Vitest）
4. 编写 v0.1 版本

### 配置环境
**安装项目依赖**

```shell
npm i vitest happy-dom -D
```

配置 vite.config.js，增加 `test` 配置

```js
export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true, // jest like语法
    environment: 'happy-dom', // 模拟DOM环境
  }
})
```

package.json 增加 "test"

```json
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest" // test指定为vitest
  },
```

添加单元测试 `src/components/HelloWorld.test.js`，内容如下，

```js
test('should first', () => {
    expect(true).toBe(true)
});
```

**验证环境**

执行下面命令，启动项目

```shell
pnpm run dev
```

执行下面命令，启动测试

```shell
pnpm run test
```

### 编写 v0.1 版本

- 新建 mini-vue
  - 提供 `createApp()`，返回 App 实例，实现了 `mount()` 方法
  - 实例的 `mount()` 方法将传入的组件渲染结果增加到宿主元素
- 手动实现 `render()` *后续由编译器生成*

关键代码：mainjs

```js
import { createApp } from './mini-vue' // 引入自己的实现

createApp({
    data() { /*...*/ },
    render() {
        // 手动实现渲染函数，返回渲染结果
        // todo: 此处后续由编译器生成
        const h3 = document.createElement('h3')
        h3.textContent = this.title
        return h3
    }
}).mount('#app')
```

关键代码：mini-vue/index.js

```js
// 创建App实例
export function createApp(rootComponent) {
    //接收根组件，返回App实例
    return {
        mount(selector) {
            // console.log('mount!')
            // 1.获取宿主
            const container = document.querySelector(selector)
            // 2.渲染视图
            // call指定数据上下文
            const el = rootComponent.render.call(rootComponent.data())
            // 3.追加到宿主
            container.appendChild(el)
        }
    }
}
```

## 效果

- 环境搭建成功：[tag v0.0][3]
- [v0.1][4]：加载正确，显示 H3 标题 **Hello, mini-vue!**

## 总结

[1]: https://github.com/tangyouhua/lab-mini-vue
[2]: https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project
[3]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.0
[4]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.1
