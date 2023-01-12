# 动手实验 mini-vue（3）

> 极客时间前端进阶训练营笔记—Day7，2023-1-12

## 大纲

- [x] v0.1
  - 创建实例 `createApp(App)`
  - `app.mount('#app')`
- [x] v0.2
  - 创建渲染器 renderer：`createRender()`
  - 提供 render：`renderer.render`
- [x] v0.3
  - 引入 reactive 机制
  - 实现依赖搜集
- v0.4
  - 引入虚拟 DOM：`createVNode`
  - patch 算法

## 目标

- 完成 v0.3 版本

## 步骤

1. 编写 v0.3 版本
2. 增加 reactiviy

### 编写 v0.3 版本

**是什么？**

> 响应性：Vue 会自动跟踪 JavaScript 状态并在其发生变化时响应式地更新 DOM。（[Vue 简介][6]）

响应式工作流程

1. 声明响应式状态
2. 依赖搜集
3. 视图更新函数

**实现思路**

- 提供 `reactive()` 函数：基于 [Proxy 对象][7]实现功能
- 提供 `effect()` 函数：基于 [WeakMap][8] 存储响应依赖关系

**核心代码**

main.js `createApp()` 加载组件两秒后改变响应式数据 `data()`：

```js
createApp({
    data() {
        return {
            title: 'Hello, mini-vue!'
        }
    },
    // ...
    mounted() {
        setTimeout(() => {
            this.title = 'This is a new title!' // 改变数据 data.title
        }, 2000)
    }
}).mount('#app')
```

通过 `reactive()` 声明 `data()` 为响应式数据：

```js
export function createRenderer(options) {
    const render = (rootComponent, selector) => {
        const container = options.querySelector(selector)
        const observed = reactive(rootComponent.data()) // 声明 data 为 响应式数据
        // 定义（副作用）更新函数
        const componentUpdateFn = () => {
            const el = rootComponent.render.call(observed)
            options.setElementText(container, '') // 清除上一次内容
            options.insert(el, container)
        }
        effect(componentUpdateFn)  // 设置激活的副作用
        componentUpdateFn() // 初始化执行一次
        // 挂载钩子 mounted 传入响应式数据
        if (rootComponent.mounted) {
            rootComponent.mounted.call(observed)
        }
    }
    return {
        render,
        createApp: createAppAPI(render)
    }
}
```

reactive() reactivity/index.js 实现响应式机制。

> 注意：这里相比 Vue2，采用了 Proxy 对象，能够支持 delete property 功能

第一版、用 Proxy 包装数据，拦截 get/set/delete property：

```js
// 接收需要做响应式处理的对象obj，返回一个代理对象
export function reactive(obj) {
    return new Proxy(obj, {
        get(target, key) {
            return Reflect.get(target, key)
        },
        set(target, key, value) {
            return Reflect.set(target, key, value)
        },
        deleteProperty(target, key) {
            return Reflect.deleteProperty(target, key)
        },
    })
}
```

第二版、`track()` 跟踪依赖关系，`trigger()` 触发依赖事件：

```js
export function reactive(obj) {
    return new Proxy(obj, {
        get(target, key) {
            const value = Reflect.get(target, key)
            // 依赖跟踪
            track(target, key)
            return value
        },
        set(target, key, value) {
            const result = Reflect.set(target, key, value)
            // 依赖触发
            trigger(target, key)
            return result
        },
        deleteProperty(target, key) {
            const result = Reflect.deleteProperty(target, key)
            // 依赖触发
            trigger(target, key)
            return result
        },
    })
}
```

数据响应依赖管理：

```js
// 创建一个Map保存依赖关系 {target: {key: [fn1, fn2]}}
const targetMap = new WeakMap()
```

`track()` 跟踪数据响应依赖：

```js
function track(target, key) {
    if (activeEffect) {
        let depsMap = targetMap.get(target)
        // 首次depsMap是不存在的，需要创建
        if (!depsMap) {
            targetMap.set(target, (depsMap = new Map()))
        }

        // 获取depsMap中key对应的Set
        let deps = depsMap.get(key)
        if (!deps) {
            depsMap.set(key, (deps = new Set()))
        }

        // 添加当前激活的副作用
        deps.add(activeEffect)
    }
}
```

`trigger()` 触发数据响应：

```js
function trigger(target, key) {
    const depsMap = targetMap.get(target)
    if (depsMap) {
        const deps = depsMap.get(key) // 可能注册多个函数
        if (deps) {
            deps.forEach(dep => {
                dep()
            })
        }
    }
}
```

建立测试用例：设计测试用例涵盖，

- 修改原对象，更新代理对象
- 修改代理对象，更新原对象
- 支持新增、删除属性

```js
test('reactive should work', () => {
    const original = { foo: 'foo' }
    const observed = reactive(original)

    // 代理对象是全新的对象
    expect(observed).not.toBe(original)

    // 能够访问所代理对象的属性
    expect(observed.foo).toBe('foo')

    // 能够修改所代理对象的属性
    observed.foo = 'foooooooo~'
    expect(original.foo).toBe('foooooooo~')

    // 能够新增所代理对象的属性
    observed.bar = 'bar'
    expect(original.bar).toBe('bar')

    // 能够删除所代理对象的属性
    delete observed.bar
    expect(original.bar).toBe(undefined)
})
```

## 效果

- [v0.3][9]
  - 页面标题 data.title 为响应式数据，页面加载 2 秒后显示标题 **This is a new title!**
  - 实现 reactive 声明数据为响应式，管理、触发响应式数据依赖

## 总结

- reactivity 利用 Proxy 实现响应式接口

[1]: https://github.com/tangyouhua/lab-mini-vue
[2]: https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project
[3]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.0
[4]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.1
[5]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.2
[6]: https://cn.vuejs.org/guide/introduction.html#what-is-vue
[7]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[8]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
[9]: https://github.com/tangyouhua/lab-mini-vue/releases/tag/v0.3
