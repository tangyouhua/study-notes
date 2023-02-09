# Jest 单元测试（5）

> 前端进阶训练营笔记-2月打卡-Day3，2023-2-8

## 是什么

[Jest](https://www.jestjs.cn/) 是一个 JavaScript 测试框架，旨在确保任何 JavaScript 代码的正确性。它为你提供了。它为你提供了 易于理解、熟悉且功能丰富的 API 来编写测试用例，并快速地反馈结果。

Jest 架构介绍：[Jest Architecture 视频](https://www.jestjs.cn/docs/architecture)。

异步测试：

- Jest 支持测试异步函数，例如 `setTimeout`
- 对于耗时的单元测试，可以通过 [Timer Mocks](https://www.jestjs.cn/docs/timer-mocks) 快进

Mock 测试：

- Mock 外部依赖，避免单元测试引入其他的依赖，比如 axois
- Mock 函数调用，通过此功能可以确认函数是否被调用、调用的次数等

Dom 测试：

- 单元测试模拟浏览器中的环境，通过引入 [jsdom](https://github.com/jsdom/jsdom) 模拟
- 支持模拟浏览器参数 Options，例如 url, contentType 等

快照测试：[Snapshot testing](https://www.jestjs.cn/docs/snapshot-testing)

- 对于前端页面通过序列化进行保存
- 开发修改代码或重构以后，在测试中进行比对

## 为什么

众多开源框架通过单元测试提高代码质量，例如开发新功能、重构等。

与此同时，单元测试是一种很好的文档，便于理解和学习。

## 实操案例

案例：快照测试

首先，编写测试代码，`src/__tests__/snapshot.spec.js`

```js
const {generateDiv} = require('../dom')
require('../jsdom-config')

test('Dom的快照测试', () => {
  generateDiv()
  expect(document.getElementsByClassName('c1')).toMatchSnapshot()
})
```

接着，运行测试 jest snapshot。这时会在 `src/__test__` 下新建 `__snapshots__` 目录。

第一次运行测试，会生成 snapshot.spec.js.snap 文件。内容为序列化后的 Dom。

```js
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`Dom的快照测试 1`] = `
HTMLCollection [
  <div
    class="c1"
  />,
]
`;
```

模拟实际开发可能出现的情况：修改页面，重构代码。

修改 src/dom.js，模拟页面修改：

```js
exports.generateDiv = () => {
  const div = document.createElement('div')
  div.className = 'c2' // c1 => c2
  document.body.appendChild(div)
}
```

再次运行测试，会报告错误：

```diff
$ jest snapshot
 FAIL  src/__tests__/snapshot.spec.js
  × Dom的快照测试 (7 ms)
                                                                          
  ● Dom的快照测试                                                         
                                                                          
    expect(received).toMatchSnapshot()

    Snapshot name: `Dom的快照测试 1`

    - Snapshot  - 5
    + Received  + 1

    - HTMLCollection [
    -   <div
    -     class="c1"
    -   />,
    - ]
    + HTMLCollection []

      4 | test('Dom的快照测试', () => {
      5 |   generateDiv()
    > 6 |   expect(document.getElementsByClassName('c1')).toMatchSnapshot()
```

修改 src/dom.js，模拟代码重构：

```js
exports.generateDiv = () => {
  const mydiv = document.createElement('div')
  mydiv.className = 'c1'
  document.body.appendChild(mydiv)
}
```

再次运行测试，测试通过。说明本次代码重构对网页内容没有带来变化。

快照测试更多内容可参考 [Expect.toMatchSnapshot API](https://www.jestjs.cn/docs/expect#tomatchsnapshotpropertymatchers-hint) 与 [Snapshot Testing 教程](https://www.jestjs.cn/docs/snapshot-testing)。

## 参考

- 官网教程
  - [快速入门](https://www.jestjs.cn/docs/getting-started)
  - 支持 [babel](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-babel)、[webpack](https://www.jestjs.cn/docs/webpack)、[vite](https://www.jestjs.cn/docs/getting-started#using-vite)、[Typescript](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-typescript)等
  - 支持 [React](https://www.jestjs.cn/docs/tutorial-react) 以及其它 Web 框架
- [API](https://www.jestjs.cn/docs/api)
- [Jest CLI](https://www.jestjs.cn/docs/cli)
- [Building a JavaScript Testing Framework](https://cpojer.net/posts/building-a-javascript-testing-framework)
- [jsdom](https://github.com/jsdom/jsdom)

此文章为2月Day3学习笔记，内容来源于极客时间《前端进阶训练营》
