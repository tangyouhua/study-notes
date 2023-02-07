# Jest 单元测试（4）

> 前端进阶训练营笔记-2月打卡-Day2，2023-2-7

## 是什么

[Jest](https://www.jestjs.cn/) 是一个 JavaScript 测试框架，旨在确保任何 JavaScript 代码的正确性。它为你提供了。它为你提供了 易于理解、熟悉且功能丰富的 API 来编写测试用例，并快速地反馈结果。

Jest 架构介绍：[Jest Architecture 视频](https://www.jestjs.cn/docs/architecture)。

异步测试：

- Jest 支持测试异步函数，例如 `setTimeout`
- 对于耗时的单元测试，可以通过 [Timer Mocks](https://www.jestjs.cn/docs/timer-mocks) 进行快进

Mock 测试：

- Mock 外部依赖，避免单元测试引入其他的依赖，比如 axois
- Mock 函数调用，通过此功能可以确认函数是否被调用、调用的次数等

Dom 测试：

- 单元测试模拟浏览器中的环境，通过引入 [jsdom](https://github.com/jsdom/jsdom) 模拟
- 支持模拟浏览器参数 Options，例如 url, contentType 等

## 为什么

众多开源框架通过单元测试提高代码质量，例如开发新功能、重构等。

与此同时，单元测试是一种很好的文档，便于理解和学习。

## 实操案例

案例：DOM 测试

首先，新建 src/dom.js 编写功能代码

```js
exports.generateDiv = () => {
  const div = document.createElement('div')
  div.className = 'c1'
  document.body.appendChild(div)
}
```

此时，编写测试代码，`src/__tests__/dom.spec.js`，执行测试 `jest dom` 会报告 undefined 对象

```js
const {generateDiv} = require('../dom')

it('Dom测试', () => {
  generateDiv()
})
```

```shell
% jest dom
 FAIL  src/__tests__/dom.spec.js
  × Dom测试 (1 ms)
                                                                         
  ● Dom测试                                                              
                                                                         
    The error below may be caused by using the wrong test environment, see https://jestjs.io/docs/configuration#testenvironment-string.
    Consider using the "jsdom" test environment.

    ReferenceError: document is not defined
```

根据提示信息，安装 jsdom: `npm install jsdom --save-dev`

接着，编写 jsdom 工具代码 src/jsdom-config.js

- 创建 JSDOM 对象，设置默认参数
- 设置 window 以及 window.document, window.navigator 到全局对象

> Dom 对象的 API 详见 MDN [Window 对象模型](https://developer.mozilla.org/zh-CN/docs/Web/API/Window)。

```js
const jsdom = require('jsdom')
const {JSDOM} = jsdom

const dom = new JSDOM("<!DOCTYPE html><head/><body></body>", {
  url: "http://localhost",
  referrer: "https://example.com",
  contentType: "text/html",
  userAgent: "Mellblomenator/9000",
  includeNodeLocations: true,
  storageQuota: 10000000,
});

global.window = dom.window
global.document = window.document
global.navigator = window.navigator
```

这时，引入 jsdom-config，dom 测试代码就可以用 dom mock 运行。增加 dom 相关的测试，如下：

`src/__tests__/dom.spec.js`，

```js
const {generateDiv} = require('../dom')
require('../jsdom-config')

it('Dom测试', () => {
  generateDiv()
  expect(document.getElementsByClassName('c1').length).toBe(1)
})
```

运行 `jest dom` 单元测试通过。

## 参考

- 官网教程
  - [快速入门](https://www.jestjs.cn/docs/getting-started)
  - 支持 [babel](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-babel)、[webpack](https://www.jestjs.cn/docs/webpack)、[vite](https://www.jestjs.cn/docs/getting-started#using-vite)、[Typescript](https://www.jestjs.cn/docs/getting-started#%E4%BD%BF%E7%94%A8-typescript)等
  - 支持 [React](https://www.jestjs.cn/docs/tutorial-react) 以及其它 Web 框架
- [API](https://www.jestjs.cn/docs/api)
- [Jest CLI](https://www.jestjs.cn/docs/cli)
- [Building a JavaScript Testing Framework](https://cpojer.net/posts/building-a-javascript-testing-framework)
- [jsdom](https://github.com/jsdom/jsdom)

此文章为2月Day2学习笔记，内容来源于极客时间《前端进阶训练营》
