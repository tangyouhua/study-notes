# Vue3 nextTick 原理与 patch 细节

> 极客时间前端进阶训练营笔记—Day3，2023-1-8

## nextTick

**是什么？**

> 等待下一次 DOM 更新刷新工具的方法。
> 
> 官方文档 [nextTick API][1]

**为什么？**

因为响应式数据刷新 DOM 是**异步执行**。

比如下面这段代码：

- 第一次输出 `p1.value` 是同步执行，这时结果为 1（变量更新 DOM 未更新）；
- 第二次，在 `nextTick()` 中输出 `p1.value`，这时结果为 2.

```js
<div id="app">
    <h1>Vue3 nextTick API</h1>
    <p ref="p1">{{counter}}</p>
</div>
<script src="./node_modules/vue/dist/vue.global.js"></script>
<script>
    const app = Vue.createApp({

        setup() {
            const counter = Vue.ref(1)
            const p1 = Vue.ref(null)

            Vue.onMounted(() => {
                counter.value++
                console.log(p1.value.innerHTML)
                Vue.nextTick(() => {
                    console.log(p1.value.innerHTML)
                })
            })
            return { counter, p1 }
        }
    })
    app.mount('#app')
</script>
```

**怎么实现？**

响应式 API 拦截到数据变化时，会通过异步队列在浏览器执行过程中进行渲染。

`nextTick()` 的效果，调用后，将代执行的任务 `fn` 插入到任务队列合适的位置。这样任务批量执行时，就能够正确地拿到渲染后的结果。

Vue3 中 `nextTick()` 实现：[scheduler.ts][2]

```js
export function nextTick<T = void>(
  this: T,
  fn?: (this: T) => void
): Promise<void> {
  const p = currentFlushPromise || resolvedPromise
  return fn ? p.then(this ? fn.bind(this) : fn) : p
}
```

## patch 细节

**是什么？**

回顾响应式原理：[Vue3 响应式原理（1）][3]

> 采用观察者模式，组件作为观察者，订阅数据的变化。
> 
> 通过 Proxy 拦截数据的 set, get
> 
> 注册数据读写事件需要处理的 handler
> 
> 当发生 set 事件事，根据订阅的情况，在指定的对象上执行 handler
> 
> 实现自动更新

这里的 patch 其实就是更新（虚拟？） DOM 操作。

**为什么？**

为了实现响应式的效果，当数据发生变化时，需要：

- 拦截变化的事件
- 解析出变化的内容
- 更新虚拟 DOM

**怎么实现？**

这里涉及到的细节比较多，主要记录一下实现思路与涉及到的核心数据结构。

思路：

- 将 template 合理规划：区分为静态 static 与动态两个部分
- 对变化的部分，分析有哪些可能的类型，比如 TEXT 等等
- 解析变化的内容：设计高效的算法，对动态部分，变化前后的数据结构能够尽可能以最小的代价（响应时间）进行更新

核心数据结构与算法：

- 通过基于二进制 Flag 的方式，定义 [ShapeFlags][4] 与 [PatchFlags][7]，为执行 patch 做好准备
- [getSequence()][6]：采用[最长自增子序列（Longest increasing subsequence）算法][5] 执行 patch

这里还有很多的实现细节，待后续的学习逐步补充。


[1]: https://cn.vuejs.org/api/general.html#nexttick
[2]: https://github.com/vuejs/core/blob/6aaf8efefffdb0d4b93f178b2bb36cd3c6bc31b8/packages/runtime-core/src/scheduler.ts#L53
[3]: frontend-camp-day-1.md
[4]: https://github.com/vuejs/core/blob/6f663d47e527c96d539b5c8e1786b30dd32bd8e8/packages/shared/src/shapeFlags.ts
[5]: https://en.wikipedia.org/wiki/Longest_increasing_subsequence
[6]: https://github.com/vuejs/core/blob/da2ced15339b6fdb7a1459fa359bb79346a82bc2/packages/runtime-core/src/renderer.ts#L2402
[7]: https://github.com/vuejs/core/blob/9c304bfe7942a20264235865b4bb5f6e53fdee0d/packages/shared/src/patchFlags.ts
