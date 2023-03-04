# 迷你 Pinia

> 前端进阶训练营笔记-3月打卡-Day4，2023-3-4

本文的目标是设计与动手实现一个迷你 Pinia，在 Pinia 的 demo 中替换为迷你 Pinia 达到同样效果。

## 项目准备

从 GitHub clone 仓库并切换到 `init` 分支：

```Bash
git clone https://github.com/tangyouhua/lab-mini-pinia.git
cd lab-mini-pinia
git checkout init
yarn install
yarn run dev
```

## 实现思路

通过阅读 Pinia 源码可以知道，迷你 Pinia 需要满足下面几个要点：

- 可以在任意地方实现 store，然后统一管理；
- 每个 store 在系统中只有一个实例，可以通过 `Id` 获取；
- 实力中的 `getter`、`action` 需要能够动态刷新；
- Pinia 实例可以作为组件注入。

下面按照这几个思路，通过代码重构的方式开始实现。

说明：类似TDD思想，但是采取手动测试。

## 实现useCounterStore

第一步，将之前实现的 counter store，提取到 useCounterStore 方法：

- `state`：例如 `count`
- `gettter`：例如 doubleCounter()
- `actions`：例如 `inc()`
- store 的 `$patch()`

实现了从具体实现抽取到 store 里，为下一步接受用户的任意参数做准备。

```JavaScript
// src/components/HelloWorld.vue
// import { useCounterStore } from '../../store/counter';
function useCounterStore() {
  const state = reactive({
    count: 0
  })

  const store = reactive({
    ...toRefs(state),
    doubleCounter: computed(() => state.count * 2),
    inc() {
      state.count++
    },
    $patch(partialStateOrMutator) {
      if (typeof partialStateOrMutator === 'object') {
        Object.keys(partialStateOrMutator).forEach(key => {
          state[key] = partialStateOrMutator[key]
        })
      } else {
        // function 
        partialStateOrMutator(state)
      }
    }
  })
  return store
}
```

执行效果：用新的 `useCounterStore` 替换之前的实现，各按钮点击效果一致。

这里需要注意的是：需要用 `toRefs`、`computed` 实现数据的动态刷新。

完整的实现可以查看 **step1** 分支。

## 实现 defineStore

第二步，定义自己的 pinia ，同时把前面实现的 `useCounterStore` 方法加入到 mypinia 中。

具体的步骤：

- 分解 `defineStore` 传入的 `options`
- 对 `state` 加上 `reactive` 与 `computed`
- 对 `getters` 加上 `computed`
- 为 `actions` 增加上下文

```JavaScript
export function defineStore(options) {
  const { state: stateFn, getters, actions } = options;

  const state = reactive(stateFn());

  function useStore() {
    const store = reactive({
      ...toRefs(state),
      ...Object.keys(getters || {}).reduce((computedGetters, name) => {
        computedGetters[name] = computed(() => {
          return getters[name].call(store, store);
        });
        return computedGetters;
      }, {}),
      ...Object.keys(actions || {}).reduce((wrapActions, actionName) => {
        // todo: add try-catch
        // todo: add action listener
        wrapActions[name] = () => actions[acitonName].call(store);
        return wrapActions;
      }, {}),
      // ...
    });
    return store;
  }
  return useStore;
}
```

注意：对于 `actions`  要做得完善，还需要考虑异常处理并允许用户对它进行监听。

在 HelloWorld 中进行替换，达到平等替代的效果。

```Vue
// import { useCounterStore } from '../../store/counter';
import { useCounterStore } from "../../store/mycounter";
```

完整的实现可以查看 **step2** 分支。

## 实现 store 管理

扩展之前的实现，为 `defineStore` 增加 `id`，通过 `Map` 保存与获取。

```JavaScript
const stores = new Map();
function useStore() {
    // create a store if not exists
    if (!stores.has(id)) {
      stores.set(id, /* store by defineStore() */)
    }
    return stores.get(id);
}

```

在 mycounter 中增加 `id`，达到平等替代的效果。

```JavaScript
export const useCounterStore = defineStore("counter", {
```

完整的实现可以查看 **step3** 分支。

此文章为3月Day4学习笔记，内容基于极客时间前端训练营。
