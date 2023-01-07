# Vue3 响应式原理（2）

> 极客时间前端进阶训练营笔记—Day2，2023-1-7

**是什么？**

- 响应式数据 `Ref`：把一个值变为响应式能够处理的对象，当值发生变化时，能够侦测（`track`）到变化，并根据响应式机制执行对应的副作用（Effect）。
  - `computed` 函数本质是一种 `Ref`，类型为 `ComputedRefImpl`
- 响应式副作用 `ReactiveEffect`：通常是某种计算，例如更新 DOM 等。
- 响应式监视：`watch` 监视响应式对象，执行回调（callback）

**为什么？**

要实现响应式的机制，在开发者的角度，只要做了：

- 标记某个值为响应式数据
- 注册值变化时需要进行的处理：例如 `computed` 或者 ReactiveEffect

就能够：

- 根据业务需要修改值
- 看到自动计算的结果，或者 DOM 更新并渲染

通过监视，能够获取数据变化的跟踪，获取调用前后值（`oldVal`, `newVal`），根据需要加入自己的处理。

**怎么实现？**

在学习中，除了源代码导读，还可以通过官网的[《深入响应式系统》][1]内容辅助理解。

思路是“注册-追踪-检查-调用”核心流程如下：

1. 注册：调用 `Ref()` 注册响应式入局
2. 追踪：`createRef()` 时，会记录该关系（dependency）到 `dep[]`。在数据读写时，拦截访问（Proxy）
3. 检查：拦截到读取数据时，`track()` 响应式关系
4. 调用：拦截到写数据时，`trigger()` 注册的副作用（一个或多个）
5. 监视（可选）：如果有 `watch`，会在 `trigger()` 调用时，通过 `scheduler` 执行回调函数

在 Vue3 中主要通过 [Proxy][2] 来实现。相比较 Vue2 使用的 [Object.defineProperty()][3] 有一些便利与优势。

但是同时也带来了对浏览器版本的限制：IE11 及以下版本不兼容。

官网的文档中，还讨论了响应式调试与与外部系统集成的集中方式。这些作为动手练习与后续学习的事项。

- [ ] 动手实现响应式调试
- [ ] 学习响应式如何与外部状态系统集成，包括**不可变数据**、**状态机**、**RxJS**
- [ ] 了解 Vue3.1 与 compact 包？对 IE11 等的兼容性

[1]: https://cn.vuejs.org/guide/extras/reactivity-in-depth.html
[2]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[3]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
