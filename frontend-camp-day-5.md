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
- 完成 v0.1，v0.2 版本功能

## 步骤

1. 建立 GitHub 仓库：[lab-mini-vue][1]
2. 按照 [Vite 文档][2]建立项目
3. 配置环境（Vite + Vue + Vitest）

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

## 效果

## 总结

[1]: https://github.com/tangyouhua/lab-mini-vue
[2]: https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project
