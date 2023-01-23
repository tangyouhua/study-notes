# TDD Vue (1)

> 极客时间前端进阶训练营练习项目

大纲

- [TDD Vue](#tdd-vue)
  - [目标](#目标)
    - [概念介绍](#概念介绍)
      - [单元测试驱动开发 TDD](#单元测试驱动开发-tdd)
      - [Jest 测试框架](#jest-测试框架)
  - [步骤](#步骤)
    - [配置环境](#配置环境)
    - [编写 v0.0 版本](#编写-v00-版本)
  - [效果](#效果)
  - [总结](#总结)

## 目标

- v0.0
  - 搭建初始项目，了解 TDD 工作方式与用到的工具

### 概念介绍

涉及的概念介绍：

#### 单元测试驱动开发 TDD

> [测试驱动开发（英语：Test-driven development，缩写为TDD）][doc-tdd] 是一种软件开发过程中的应用方法，由极限编程中倡导，以其倡导先写测试程序，然后编码实现其功能得名。测试驱动开发始于20世纪90年代。测试驱动开发的目的是取得快速反馈并使用“illustrate the main line”方法来构建程序。
>
> 测试驱动开发是戴两顶帽子思考的开发方式：先戴上实现功能的帽子，在测试的辅助下，快速实现其功能；再戴上重构的帽子，在测试的保护下，通过去除冗余的代码，提高代码品质。测试驱动着整个开发过程：首先，驱动代码的设计和功能的实现；其后，驱动代码的再设计和重构。

在 TDD Vue 项目中，测试驱动开发的工作流程是：

1. 根据 Vue 的核心功能，先编写单元测试，例如 `reactivity()` 的几个单元测试
2. 将原始项目中的 Vue API 替换为自己的实现，单元测试可能失败
3. 修改自己的实现，所有单元测试通过
4. 继续步骤 2

#### Jest 测试框架

> [Jest][doc-jestjs] 是一款优雅、简洁的 JavaScript 测试框架。
>
> Jest 支持 Babel、TypeScript、Node、React、Angular、Vue 等诸多框架！

举个例子，我们先写一个两数相加的函数。 首先，创建 sum.js 文件︰

```js
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

然后，创建名为 sum.test.js 的文件。 此文件中将包含我们的实际测试︰

```js
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

在 TDD Vue 项目中，项目已配置好了 jest，可以通过：

```shell
jest sum
```

或者

```shell
pnpm test
```

查看测试结果。如果单元测试不通过，会报告错误信息与失败的位置。

Jest 提供了丰富的测试 API，可以通过上面的链接查找和练习。

## 步骤

1. 建立 GitHub 仓库：[lab-tdd-vuex][1]
2. 克隆仓库并安装依赖
3. 编写第一个单元测试并运行通过
4. 实现各版本要求

### 配置环境

**安装项目依赖**

```shell
pnpm install # 后安装依赖项
```

### 编写 v0.0 版本

**为什么？**

理解课程提供的 template 模板，目录结构与相关配置。

**实现思路**

分析项目目录结构、所用技术与核心配置。

下面是执行了依赖安装后的目录结构，`node_modules/` 不做分析：
```shell
.
├── README.md
├── __test__
│   └── test.spec.js
├── babel.config.json
├── jest.config.js
├── package.json
├── pnpm-lock.yaml
├── src
│   └── a.js
└── yarn.lock
```

**核心代码**

项目主要用到了 jest 做单元测试，包含以下内容：

jest.config.js

```js
module.exports = {
  testEnvironment: "jsdom", //使用jsdom模拟dom
};
```

实现功能代码：

src/a.js

```js
export function add(a, b) {
  return a + b
}
```

编写单元测试代码：

`__test__` 目录及单元测试脚本 `test.spec.js：

- 测试文件以 `.spec.js` 为后缀，并保存在 `__test__` 目录下
- 通过 jest API 编写单元测试，例如 `describe`, `test`, `expect`, `toBe` 等

```js
import { add } from "../src/a";

describe("jest test", () => {
  test("foo", () => {
    expect(add(1, 1)).toBe(2);
  });
});
```

## 效果

- [v0.0](https://github.com/tangyouhua/lab-tdd-vue/releases/tag/v0.0)
  - 执行 `pnpm test` 可调用 jest watch，实现边开发边执行单元测试
 
## 总结

- 测试驱动开发

[1]: https://github.com/tangyouhua/lab-tdd-vue
[doc-tdd]: https://zh.wikipedia.org/wiki/%E6%B5%8B%E8%AF%95%E9%A9%B1%E5%8A%A8%E5%BC%80%E5%8F%91
[doc-jestjs]: https://jestjs.io/zh-Hans/docs/getting-started
