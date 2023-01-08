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

[1]: https://cn.vuejs.org/api/general.html#nexttick
[2]: https://github.com/vuejs/core/blob/6aaf8efefffdb0d4b93f178b2bb36cd3c6bc31b8/packages/runtime-core/src/scheduler.ts#L53
