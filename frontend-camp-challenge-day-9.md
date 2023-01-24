# TDD Vue (2)

> 极客时间前端进阶训练营练习项目—打卡-Day9，2023-1-24

大纲

- [TDD Vue](#tdd-vue)
  - [目标](#目标)
    - [概念介绍](#概念介绍)
      - [响应式 reactive() 与副作用函数 effect()](#响应式-reactive-与副作用函数-effect)
      - [Mock 函数 jest.fn()](#mock-函数-jestfn)
      - [模拟定时器函数 jest.useFakeTimers()](#模拟定时器函数-jestusefaketimers)
  - [步骤](#步骤)
    - [配置环境](#配置环境)
    - [编写 v0.1 版本](#编写-v01-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.1
  - 编写单元测试，手动实现 reactive() 和 effect()

### 概念介绍

涉及的概念介绍：

#### 响应式 reactive() 与副作用函数 effect()

> 我们（Vue）是可以追踪对象属性的读写的。
> 
> 在 Vue 3 中则使用了 [Proxy][api-js-proxy] 来创建响应式对象。下面的伪代码将会说明它们是如何工作的：

在 TDD Vue 项目中，根据 Vue 3 官方文档[“深入响应式系统”][doc-vuejs-how-reactivity-works-in-vue]章节的伪代码实现。

![](https://cn.vuejs.org/assets/render-pipeline.03805016.png)

上图为官方文档[“渲染机制--渲染管线“][doc-vuejs-render-pipeline]的示意图。

1. 首先会跟踪 track 副作用函数：map(obj, fn)
2. 接着响应式状态发生变化
3. 触发 trigger 副作用函数重新渲染
4. 返回虚拟 DOM
5. 更新 mount/patch 真实 DOM

#### Mock 函数 jest.fn()

> [jest.fn()][doc-jestjs-fnimpl] 返回一个新的、未使用的 mock 函数。可以提供默认实现（可选）。

```js
const mockFn = jest.fn();
mockFn();
expect(mockFn).toHaveBeenCalled();

// 提供默认mock实现
const returnsTrue = jest.fn(() => true);
console.log(returnsTrue()); // true;
```

在 TDD Vue 项目中，通过 `jest.fn()` 测试 effect 调用。

#### 模拟定时器函数 jest.useFakeTimers()

> [jest.useFakeTimers()][api-jestjs-jestusefaketimersfaketimersconfig] 指定 Jest 使用假的全局日期、性能、时间和定时器 API。 假定时器实现由 @sinonjs/fake-timaters 支持。
>
> 可以在任何地方调用 jest.useFakeTimers() 或 jest.useRealTimers()(如顶层，it内部等)。 请记住，这是一个 全局操作 ，并将影响到同一文件中的其他测试。 在同一测试文件内再次调用 jest.useFakeTimers() 会重置内部状态（如计时器计数），并使用提供的选项重新载入假定时器替换原生的API。

示例：

```js
test('advance the timers automatically', () => {
  jest.useFakeTimers({advanceTimers: true});
  // ...
});

test('do not advance the timers and do not fake `performance`', () => {
  jest.useFakeTimers({doNotFake: ['performance']});
  // ...
});

test('uninstall fake timers for the rest of tests in the file', () => {
  jest.useRealTimers();
  // ...
});
```

在 TDD Vue 项目中，通过 `jest.useFakeTimers()` 和 `jest.runAllTimers()` 测试 scheduler 调用。

`jest.useFakeTimers()` 的原理是，替换延时函数，以减少不必要的等待。

> [jest.runAllTimer()][api-jestjs-jestrunalltimers] exhausts both the macro-task queue (i.e., all tasks queued by setTimeout(), setInterval(), and setImmediate()) and the micro-task queue (usually interfaced in node via process.nextTick).
>
> When this API is called, all pending macro-tasks and micro-tasks will be executed. If those tasks themselves schedule new tasks, those will be continually exhausted until there are no more tasks remaining in the queue. If those tasks themselves schedule new tasks, those will be continually exhausted until there are no more tasks remaining in the queue.
>
> This is often useful for synchronously executing setTimeouts during a test in order to synchronously assert about some behavior that would only happen after the setTimeout() or setInterval() callbacks executed. See the Timer mocks doc for more information. See the Timer mocks doc for more information.

执行 `setTimeout`, `setInterval`, `setImmediate` 中排队的微任务。

### 编写 v0.1 版本

**为什么？**

手动实现响应式 reactive()，副作用函数 effect()。

**实现思路**

分析单元测试，根据官网的伪代码思路手工实现并通过单元测试。

**核心代码**

新建骨架代码

src/reactivity/reactive.js

```js
export function reactive(obj) {
  // 对数据实现响应式
  return new Proxy(...)
}
```

src/reactivity/effect.js

```js
// 注册副作用函数
export function effect(fn, options = {}) {...}
// 跟踪响应式对象状态变化
export function track(target, key) {...}
// 触发副作用调用
export function trigger(target, key) {...}
```

**单元测试**

\_\_test\_\_/reactivity.spec.js

reactive()

- 原始对象
- 调用 reactive() 跟踪对象变化
- 比对原始对象与代理对象中的属性

```js
test('reactive() ', () => {
    const original = { foo: 'foo' }
    const observed = reactive(original)
    observed.foo = 'foo~~~'
    expect(original.foo).toBe('foo~~~')
});
```

effect

- 建立响应式对象 counter
- 注册 fnSpy 函数为副作用
  - 检查调用次数为 1 次
  - dummy 赋值为 count.num，值为 0
- 修改 counter.num 为 1
  - 检查调用次数为 2 次
  - dummy 赋值为 count.num，值为 1 

```js
it('effect', () => {
    let dummy
    const counter = reactive({ num: 0 })
    const fnSpy = jest.fn(() => {
        dummy = counter.num
    })
    effect(fnSpy)

    expect(fnSpy).toHaveBeenCalledTimes(1)
    expect(dummy).toBe(0)

    counter.num = 1
    expect(fnSpy).toHaveBeenCalledTimes(2)
    expect(dummy).toBe(1)
})
```

支持响应式多属性对象

- 建立响应式对象 counter：包含 num1, num2 两个属性
- 注册响应式函数：返回 dummy 值，dummy = num1 + num1 + num2
  - 默认 dummy 等于 0
  - 修改 counter 两个属性为 7
  - dummy 等于 21 

```js
it('should observe multiple properties', () => {
    let dummy
    const counter = reactive({ num1: 0, num2: 0 })
    effect(() => (dummy = counter.num1 + counter.num1 + counter.num2))

    expect(dummy).toBe(0)
    counter.num1 = counter.num2 = 7
    expect(dummy).toBe(21)
})
```

验证副作用函数调用正确：

- 建立响应式对象 observe：包含 foo、bar 属性
- 新建 fnSpy mock 函数：返回 observe.foo 属性
- 注册 fnSpy 为副作用函数
- 修改 observe 中的 bar、foo 属性
  - mock 函数 fnSpy 被调用两次

```js
it('effect should linked to the exact key', () => {
    const observe = reactive({ foo: 'foo', bar: 'bar' })
    const fnSpy = jest.fn(() => {
        observe.foo
    });

    effect(fnSpy)
    observe.bar = 'barrr'
    observe.foo = 'foooo'
    expect(fnSpy).toHaveBeenCalledTimes(2)
});
```

调度执行：

- 创建响应式对象 obj：属性 foo 默认值 1
- 注册 effect 副作用函数：arr 数组加入 obj.foo 对象，等待执行
  - 首次注册，arr.push(1)
  - 触发响应式状态变化 obj.foo++，arr.push(2)，等待调度
  - arr.push("over")
  - 启动调度
- 验证效果：arr 调度结果 2 会在 over 之后

```js
it("调度执行", () => {
    const obj = reactive({ foo: 1 });
    const arr = [];
    jest.useFakeTimers(); // 开启模拟定时器
    effect(() => arr.push(obj.foo), {
        scheduler(fn) {
            setTimeout(fn);
        },
    });
    obj.foo++;
    arr.push("over");

    jest.runAllTimers(); // 等待所有定时器执行
    expect(arr[0]).toBe(1);
    expect(arr[1]).toBe("over");
    expect(arr[2]).toBe(2);
});
```

**核心代码**

实现响应式

- 使用 Proxy 代理实现响应式
- 拦截 set/get/deletePropery
- 注册 track, trigger 跟踪和触发副作用函数

src/reactivity/reactive.js

```js
// reactive返回传入obj的代理对象，值更新时使app更新
export function reactive(obj) {
    return new Proxy(obj, {
        get(target, key) {
            const result = Reflect.get(target, key)
            track(target, key)
            return result
        },
        set(target, key, value) {
            const result = Reflect.set(target, key, value)
            trigger(target, key)
            return result
        },
        deleteProperty(target, key) {
            const result = Reflect.deleteProperty(target, key)
            trigger(target, key)
            return result
        }
    })
}
```

支持副作用函数

- 用 WeakMap 存储响应式数据与副作用函数关系
  - target, `{key, {fn1...fnN}}`
- track 存储
- trigger（循环并）调用副作用

```js
//effect.js
export let activeEffect
// effect(fn, options)
// 注册副作用函数
export function effect(fn, options = {}) {
    // 封装一个effectFn用于扩展功能
    const effectFn = () => {
        activeEffect = effectFn
        fn()
    }
    effectFn.options = options // 增加选项以备trigger时使用
    effectFn()
}

// WeakMap: key支持对象
const targetMap = new WeakMap()

// track(targe, key)
// 跟踪副作用函数
export function track(target, key) {
    if (activeEffect) {
        let depsMap = targetMap.get(target)

        if (!depsMap) {
            targetMap.set(target, (depsMap = new Map()))
        }

        let deps = depsMap.get(key)
        if (!deps) {
            // fn作为value不重复
            depsMap.set(key, (deps = new Set()))
        }

        deps.add(activeEffect)
    }
}

// trigger(target, key)
// 触发响应式函数: scheduler()异步调用, dep() 同步调用
export function trigger(target, key) {
    const depsMap = targetMap.get(target)

    if (depsMap) {
        const deps = depsMap.get(key)

        //增加scheduler判断
        deps && deps.forEach(dep => {
            if (dep.options.scheduler) {
                dep.options.scheduler(dep)
            } else {
                dep()
            }
        })
    }
}
```

## 效果

- [v0.1](https://github.com/tangyouhua/lab-tdd-vue/releases/tag/v0.1)
  - 执行 `pnpm test` 所有响应式与副作用函数用例测试通过
  - 测试用例调用手写的 reactive() 与 effect() 实现 
 
## 总结

- 阅读并理解单元测试，熟悉 jest.fn, timer 用法，应用源代码分析课程与官网伪代码编写自己的实现

[1]: https://github.com/tangyouhua/lab-tdd-vue
[doc-tdd]: https://zh.wikipedia.org/wiki/%E6%B5%8B%E8%AF%95%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91
[doc-jestjs]: https://jestjs.io/zh-Hans/docs/getting-started
[api-js-proxy]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[doc-vuejs-how-reactivity-works-in-vue]: https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#how-reactivity-works-in-vue
[doc-vuejs-render-pipeline]: https://cn.vuejs.org/guide/extras/rendering-mechanism.html#render-pipeline
[doc-jestjs-fnimpl]: https://jestjs.io/zh-Hans/docs/jest-object#jestfnimplementation
[api-jestjs-jestusefaketimersfaketimersconfig]: https://jestjs.io/zh-Hans/docs/jest-object#jestusefaketimersfaketimersconfig
[api-jestjs-jestrunalltimers]: https://jestjs.io/zh-Hans/docs/jest-object#jestrunalltimers
