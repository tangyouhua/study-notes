# Jest 单元测试（2）

> 前端进阶训练营笔记-打卡-Day21，2023-2-5

## 是什么

[Jest](https://www.jestjs.cn/) 是一个 JavaScript 测试框架，旨在确保任何 JavaScript 代码的正确性。它为你提供了。它为你提供了 易于理解、熟悉且功能丰富的 API 来编写测试用例，并快速地反馈结果。

Jest 架构介绍：[Jest Architecture 视频](https://www.jestjs.cn/docs/architecture)。

异步测试：

- Jest 支持测试异步函数，例如 `setTimeout`
- 对于耗时的单元测试，可以通过 [Timer Mocks](https://www.jestjs.cn/docs/timer-mocks) 进行快进

## 为什么

众多开源框架通过单元测试提高代码质量，例如开发新功能、重构等。

与此同时，单元测试是一种很好的文档，便于理解和学习。

## 实操案例

案例 2: 异步函数单元测试

首先，编写异步函数，src/delay.js

```js
module.exports = fn => {
  setTimeout(() => fn(), 1000)
}
```

接着，添加异步单元测试，`src/__tests__/delay.spec.js`

```js
const delay = require('../delay')

it('异步测试', done => {
  delay(() => {
    done()
  })
  expect(true).toBe(true)
})
```

然后，执行 `jest delay` 运行单元测试，可以看到该测试花费约 1 秒多的时间

```shell
% jest delay
 PASS  src/__tests__/delay.spec.js
  ✓ 异步测试 (1002 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.27 s
```

实际测试时，每次等待 1 秒或更多的时间可能是不必要的，这里 jest 提供了 [Timer Mock](https://www.jestjs.cn/docs/timer-mocks) 实现了“快进”。

修改之前的单元测试，增加 Mock Timer。

```js
const delay = require('../delay')

it('异步测试', done => {
  jest.useFakeTimers()
  delay(() => {
    done()
  })
  jest.runAllTimers()
  expect(true).toBe(true)
})
```

运行单元测试，可以看到 delay 的等待时间从 1 秒多变成了若干毫秒。

```shell
% jest delay
 PASS  src/__tests__/delay.spec.js
  ✓ 异步测试 (3 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.267 s, estimated 2 s
```

## 参考

- 官网教程
  - [快速入门](https://www.jestjs.cn/docs/getting-started)
  - 支持 [babel](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-babel)、[webpack](https://www.jestjs.cn/docs/webpack)、[vite](https://www.jestjs.cn/docs/getting-started#using-vite)、[Typescript](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-typescript)等
  - 支持 [React](https://www.jestjs.cn/docs/tutorial-react) 以及其它 Web 框架
- [API](https://www.jestjs.cn/docs/api)
- [Jest CLI](https://www.jestjs.cn/docs/cli)
- [Building a JavaScript Testing Framework](https://cpojer.net/posts/building-a-javascript-testing-framework)
