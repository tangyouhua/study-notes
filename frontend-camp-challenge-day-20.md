# Jest 单元测试（1）

> 前端进阶训练营笔记-打卡-Day20，2023-2-4

## 是什么

[Jest](https://www.jestjs.cn/) 是一个 JavaScript 测试框架，旨在确保任何 JavaScript 代码的正确性。它为你提供了。它为你提供了 易于理解、熟悉且功能丰富的 API 来编写测试用例，并快速地反馈结果。

Jest 架构介绍：[Jest Architecture 视频](https://www.jestjs.cn/docs/architecture)。

## 为什么

众多开源框架通过单元测试提高代码质量，例如开发新功能、重构等。

与此同时，单元测试是一种很好的文档，便于理解和学习。

## 实操案例

首先，安装 Jest：`npm i jest -g`

接着，在项目目录下初始化包：`npm init`。

- Jest 会在 `__tests__` 目录下查找文件
- 测试文件通常以 `<filename>.spec.js` 命名

创建代码：src/add.js

```js
const add = (a, b) => a + b
module.exports = add
```

然后编写测试代码：`__tests__/add.spec.js`

- `describe` 创建一组测试
- `it` 创建一个测试用例
  - 描述内容对测试用例进行了自解释
  - `expect` 进行了断言

```js
const add = require('../add')

// a test unit
describe('test add()', () => {
  it('add(1, 2) === 3', () => {
    // assert
    expect(add(1, 2)).toBe(3)
  }) 

  it('add(2, 2) === 4', () => {
    // assert
    expect(add(2, 2)).toBe(4)
  }) 
})
```

编写完毕，在项目根文件夹下执行测试：

- `jest`：执行所有测试
- `jest add`：执行 `add.spec` 测试
- `jest --watch`：实时增量测试，保存代码即给出结果
  - 注意：需要初始化为 git 仓库

```shell
% jest
PASS  src/__tests__/add.spec.js
  test add()
    ✓ add(1, 2) === 3 (1 ms)
    ✓ add(2, 2) === 4

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        0.27 s
```

## 参考

- 官网教程
  - [快速入门](https://www.jestjs.cn/docs/getting-started)
  - 支持 [babel](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-babel)、[webpack](https://www.jestjs.cn/docs/webpack)、[vite](https://www.jestjs.cn/docs/getting-started#using-vite)、[Typescript](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-typescript)等
  - 支持 [React](https://www.jestjs.cn/docs/tutorial-react) 以及其它 Web 框架
- [API](https://www.jestjs.cn/docs/api)
- [Jest CLI](https://www.jestjs.cn/docs/cli)
- [Building a JavaScript Testing Framework](https://cpojer.net/posts/building-a-javascript-testing-framework)
