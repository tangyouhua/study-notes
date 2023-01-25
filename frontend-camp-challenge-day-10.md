# TDD Vue (3)

> 极客时间前端进阶训练营练习项目—打卡-Day10，2023-1-25

大纲

- [TDD Vue](#tdd-vue)
  - [目标](#目标)
  - [步骤](#步骤)
    - [编写 v0.2 版本](#编写-v02-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.2
  - 修改 effect 实现，支持 effectFn() 嵌套调用

### 编写 v0.2 版本

**为什么？**

副作用函数 effect() 实际中会有嵌套调用的情况，需要支持嵌套调用。

**单元测试**

effectFn 嵌套调用测试：

- 新建响应式对象 obj：包含 foo, bar 两个属性
- 通过 jest.fn 注册两个 mock 函数：mockFn1, mockFn2 -> 调用 mockFn1
  - mockFn1 调用后 arr 加入 obj.bar 值
  - mockFn2 调用后 arr 加入 obj.foo 值
- 注册副作用函数 mockFn2
  - 验证 `arr[0]`, `arr[1]` 值
- 修改 `obj.foo`
  - 验证 `arr[3]` 值

```js
it("should be nested", () => {
    const obj = reactive({ foo: "foo", bar: "bar" });
    const arr = []
    const mockFn1 = jest.fn(() => {
        arr.push(obj.bar)
    });
    const mockFn2 = jest.fn(() => {
        // mockFn1嵌套在mockFn2内部，obj.bar -> mockFn1
        effect(mockFn1);
        arr.push(obj.foo)
    });
    // mockFn2包含了mockFn1，obj.foo -> mockFn2
    effect(mockFn2)
    // 因此希望内部mockFn1先执行，arr[0]是bar，arr[1]是foo
    expect(arr[0]).toBe('bar')
    expect(arr[1]).toBe('foo')
    // 修改foo，希望触发mockFn2，再次添加obj.foo
    obj.foo = "fooooo";
    expect(arr[3]).toBe('fooooo');
});
```

**实现思路**

- v0.1 版本单元测试不通过
  - 提示 `arr[3]` 实际值为 `undefined`
  - 期待为修改后的值 fooooo 
- 分析
  - 手写 vue 的逻辑：响应式对象 get 方法会跟踪 track 副作用函数，set 方法会触发 trigger 副作用函数
  - 副作用函数存储逻辑：target 为响应式对象，key 为 property 名字，value 为响应式函数集合 Set
  - effect() 注册副作用函数：记录到 activeEffect，当 track 调用时记录到存储
  - 问题：obj.foo 触发 trigger 副作用时，没有执行期待的副作用
  - 原因：由于 effect(mockFn2) 会调用 effect(mockFn1)，而 v0.1 实现并没有考虑调用栈，所以实际最终副作用函数存储中 obj.foo 对应的副作用有问题\*
  - 验证这个逻辑：Debug 查看 effect.js 中的 deps，发现 obj.foo 对应的 Set 为空
- 解决办法
  - 修改 effect() 注册 activeEffect 部分，每次调用记录堆栈，并跟踪 track

**注意**：这里有问题\*的现象是，按照 foo 去找副作用函数，先是错误地调用了 mockFn1，然后没有正常调用 mockFn2。

**核心代码**

- 在 effect() 中支持嵌套调用 effect()
- 保证 activeEffect 各层次记录的 obj.key -> fn 都保持正确
- 由于是嵌套调用，这里用 Stack 模拟

src/reactivity/effect.js

```js
export let activeEffect
// effect(fn, options)
// 注册副作用函数
const effectStack = [] // 支持嵌套
export function effect(fn, options = {}) {
    // v0.1代码
    // const effectFn = () => {
    //     activeEffect = effectFn
    //     fn()
    // }

    // 封装一个effectFn用于扩展功能
    const effectFn = () => {
        activeEffect = effectFn
        effectStack.push(effectFn)
        fn()
        effectStack.pop()
        activeEffect = effectStack[effectStack.length - 1]
    }
    effectFn.options = options // 增加选项以备trigger时使用
    effectFn()
}
```

遗留问题：

- [ ] 上面的代码解决了这个单元测试的例子，但是执行跟踪 track 是响应式对象 get 函数触发，所以 effect 与 track 的时序关系会多有种情况。

## 效果

- [v0.2](https://github.com/tangyouhua/lab-tdd-vue/releases/tag/v0.2)
  - 嵌套 effect() 单元测试执行通过
 
## 总结

- effect() 会登记副作用函数同时执行一次 effectFn，对于可能出现嵌套调用 effect() 情况使用 Stack 处理，使用 [Jest Runner][extension-vscode-jestrunner] 调试

[1]: https://github.com/tangyouhua/lab-tdd-vue
[doc-tdd]: https://zh.wikipedia.org/wiki/%E6%B5%8B%E8%AF%95%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91
[doc-jestjs]: https://jestjs.io/zh-Hans/docs/getting-started
[api-js-proxy]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[doc-vuejs-how-reactivity-works-in-vue]: https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#how-reactivity-works-in-vue
[doc-vuejs-render-pipeline]: https://cn.vuejs.org/guide/extras/rendering-mechanism.html#render-pipeline
[doc-jestjs-fnimpl]: https://jestjs.io/zh-Hans/docs/jest-object#jestfnimplementation
[api-jestjs-jestusefaketimersfaketimersconfig]: https://jestjs.io/zh-Hans/docs/jest-object#jestusefaketimersfaketimersconfig
[api-jestjs-jestrunalltimers]: https://jestjs.io/zh-Hans/docs/jest-object#jestrunalltimers
[extension-vscode-jestrunner]: https://marketplace.visualstudio.com/items?itemName=firsttris.vscode-jest-runner 
