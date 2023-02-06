# Jest 单元测试（3）

> 前端进阶训练营笔记-2月打卡-Day1，2023-2-6

## 是什么

[Jest](https://www.jestjs.cn/) 是一个 JavaScript 测试框架，旨在确保任何 JavaScript 代码的正确性。它为你提供了。它为你提供了 易于理解、熟悉且功能丰富的 API 来编写测试用例，并快速地反馈结果。

Jest 架构介绍：[Jest Architecture 视频](https://www.jestjs.cn/docs/architecture)。

异步测试：

- Jest 支持测试异步函数，例如 `setTimeout`
- 对于耗时的单元测试，可以通过 [Timer Mocks](https://www.jestjs.cn/docs/timer-mocks) 进行快进

Mock 测试：

- Mock 外部依赖，避免单元测试引入其他的依赖，比如 axois
- Mock 函数调用，通过此功能可以确认函数是否被调用、调用的次数等

## 为什么

众多开源框架通过单元测试提高代码质量，例如开发新功能、重构等。

与此同时，单元测试是一种很好的文档，便于理解和学习。

## 实操案例

案例 1：模拟 axios

首先，编写 fetch.js 通过 `axios.get` 请求数据

```js
const axios = require('axios')
exports.getData = () => axios.get('/abc/def')
```

接着，编写单元测试 fetch.spec.js。

- 这里用到了 Jest 的 [Manual Mock](https://www.jestjs.cn/docs/manual-mocks)，支持 Mock 用户 Module 和 Node Module。
- 下面的例子中使用了 [mockFn.mockResolvedValueOnce(value)](https://www.jestjs.cn/docs/mock-function-api#mockfnmockresolvedvaluevalue)，模拟每次 axios.get 调用的返回值

```js
const {getData} = require('../fetch')
const axios = require('axios')
jest.mock('axios')
it('Test fetch', async () => {
  axios.get.mockResolvedValueOnce('123')
  axios.get.mockResolvedValueOnce('456')
  const data1 = await getData()
  expect(data1).toBe('123')
  const data2 = await getData()
  expect(data2).toBe('456')
})
```

案例 2：测试函数调用

在 `src/__tests__/delay.spec.js` 中增加测试用例。下面的例子中使用了 [.toBeCalledTimes(number)](https://www.jestjs.cn/docs/expect#tohavebeencalledtimesnumber) 对 Mock 函数调用次数进行断言。

```js
it('异步测试', done => {
  let mockFn = jest.fn()
  jest.useFakeTimers()
  delay(() => {
    mockFn()
    done()
  })
  jest.runAllTimers()
  expect(true).toBe(true)
  expect(mockFn).toBeCalled()
})
```

如果函数没有调用，比如注释掉 mockFn()，jest 会给出下面报错：

```shell
● 异步测试
                                                          
    expect(jest.fn()).toBeCalled()

    Expected number of calls: >= 1
    Received number of calls:    0

      19 |   jest.runAllTimers()
      20 |   expect(true).toBe(true)
    > 21 |   expect(mockFn).toBeCalled()
         |                  ^
      22 | })
```

当然，也可以测试 Mock 函数的调用次数。例如 mockFn() 调用两次，在单元测试中加入下面的断言：

```js
expect(mockFn).toBeCalledTimes(2)
```

## 参考

- 官网教程
  - [快速入门](https://www.jestjs.cn/docs/getting-started)
  - 支持 [babel](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-babel)、[webpack](https://www.jestjs.cn/docs/webpack)、[vite](https://www.jestjs.cn/docs/getting-started#using-vite)、[Typescript](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-typescript)等
  - 支持 [React](https://www.jestjs.cn/docs/tutorial-react) 以及其它 Web 框架
- [API](https://www.jestjs.cn/docs/api)
- [Jest CLI](https://www.jestjs.cn/docs/cli)
- [Building a JavaScript Testing Framework](https://cpojer.net/posts/building-a-javascript-testing-framework)

此文章为2月Day1学习笔记，内容来源于极客时间《前端进阶训练营》
