# Mini Rollup（9）

> 前端进阶训练营笔记-2月打卡-Day14，2023-2-19

手写 RollUp，了解其原理。

## 目标

- v0.8
  - 增加 Rollup 入口，实现 mini Rollup 集成打包测试

### 实现 Rollup

- 准备测试用例
- 提供 rollup 调用：引入 bundle，填写测试输入 entry，输出到打包后 bundle 文件
- 集成测试

#### 测试用例

下面这个测试用例体现了多模块的解析。

```JavaScript
// src/case01/main.js
import { a } from "./a.js"
console.log(a());

// src/case01/a.js
export const a = () => 1

```

#### Rollup

组合之前完成的 Bundle，调用 build 完成打包。

```JavaScript
// lib/rollup.js
const Bundle = require('./bundle')

function rollup(entry, outputFileName) {
    const bundle = new Bundle({ entry })
    bundle.build(outputFileName)
}

module.exports = rollup
```

#### 集成测试case 01

```JavaScript
// index.js
const path = require('path')
const rollup = require('./lib/rollup')
const entry = path.resolve(__dirname, './src/case01/main.js')
rollup(entry, './bundle.js')

```

在测试项目根目录执行 node.js，会生成 bundle.js，内容如下：

```JavaScript
// bundle.js
const a = () => 1
console.log(a());
```

对比原始的用例，可以发现 import 文件中的内容正确地打包到了输出。

注意：之前版本的代码有路径问题，做如下处理后可正确生成。

```JavaScript
if (route) {
    // 正确处理文件后缀
    const code = fs.readFileSync(route.replace(/\.js$/, '') + '.js', 'utf-8').toString()
    const module = new Module({
        code,
        path: route,
        bundle: this
    })
    return module
}
```

完整代码可查看 [v0.8 版本](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.8)。

## 效果

- [v0.8](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.8)
  - 执行 `node index.js` 可以正确生成 bundle.js 内容为 case01 打包后的结果

## 总结

- 通过 rollUp 串联之前编写的所有模块，完成 mini Rollup 的最初版本，理解了通过 ast 解析实现打包工具的主要路径。该版本还有很多待改进的空间

此文章为2月Day14学习笔记，内容来源于极客时间《前端进阶训练营》
