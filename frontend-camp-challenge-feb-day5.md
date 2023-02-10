# Webpack 基本概念与实验

> 前端进阶训练营笔记-2月打卡-Day5，2023-2-10

## 是什么


> 官网定义<sup>[1][ref1]</sup>：本质上，webpack 是一个用于现代 JavaScript 应用程序的 静态模块打包工具。当 webpack 处理应用程序时，它会在内部从一个或多个入口点构建一个 依赖图(dependency graph)，然后将你项目中所需的每一个模块组合成一个或多个 bundles，它们均为静态资源，用于展示你的内容。

[ref1]: https://webpack.docschina.org/concepts/

Webpack 有以下核心概念概念

- **entry**：webpack 打包入口，可以在配置中指定。[官网文档](https://webpack.docschina.org/concepts/#entry)
- **output**：指定在哪里输出创建的 bundle。[官网文档](https://webpack.docschina.org/concepts/#output)
- **loader**：加载器，webpack 默认支持 Javascript 和 JSON，对于其他不同文件可以通过 loader 支持。[官网文档](https://webpack.docschina.org/concepts/#loaders)。
- **plugins**：比 loader 更强大的扩展机制。[官方文档](https://webpack.docschina.org/concepts/#plugins)
- **mode**：针对不同环境的优化，比如 production, development, none。[官方文档](https://webpack.docschina.org/concepts/#mode)
- **DevServer**：用于辅助快速开发应用程序，例如修改后可以动态刷新页面、加载本地资源等。[官方文档](https://webpack.docschina.org/configuration/dev-server/)

Webpack.config.js 示例配置

```js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js',
  },
  module: {
    // 对import或require()中出现.txt后缀的文件用raw-loader转换后打包
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
    // html-webpack-plugin 为应用程序生成一个 HTML 文件
    // 并自动将生成的所有 bundle 注入到此文件中
    plugins: [new HtmlWebpackPlugin({ template: './src/index.html' })],
  },
};
```

## 为什么

为什么前端开发需要模块化打包工具？

- ES Module 环境兼容性问题
- 文件过多，请求频繁
- 除了 Module 外，还有 css、 图片、字体等文件
- 图片等多媒体资源需要进行压缩
- 代码压缩与混淆
- 项目开发的定制化需求

参考文章<sup>[2][ref2]</sup>

[ref2]: https://static.kancloud.cn/cyyspring/webpack/3079669

## 实现原理

总的来说，Webpack 通过读取每个模块、分析依赖关系、使用加载器和插件处理代码、最终生成输出文件的过程，来实现前端代码打包。

- 首先，Webpack 会读取每个模块中的代码，每个模块可以包含 JavaScript、CSS、图片等资源。然后，Webpack 会分析每个模块之间的依赖关系，并将它们组合成一个依赖图。
- 接下来，Webpack 会使用加载器和插件对代码进行处理，比如对 JavaScript 代码进行语法转换、压缩和混淆等。
- 最后，Webpack 会将处理后的代码写入一个或多个输出文件，这些文件可以直接在浏览器中加载并使用。

## 实操案例

准备工作：安装 webpack、webpack-cli 以及相关 loader、plugin 和开发工具。

```shell
yarn init -y
yarn add webpack webpack-cli
yarn add raw-loader -D
yarn add html-webpack-plugin -D
yarn add webpack-dev-server -D
```

建立实验目录与待打包的测试文件：

```shell
mkdir -p lab-webpack/src
cd lab-webpack
```

```js
// src/add.js
export const add = (a, b) => a + b
```

```js
// src/index.js
import {add} from "./add"
console.log(add(3, 3))
```

建立 webpack 配置文件：

```js
// webpack.config.js
module.exports = {
  // entry point
  entry: "./src/index.js",
  mode: "development"
}
```

查看打包结果：可以看到生成了 dist 目录以及 main.js，内容如下

```shell
% cat ./dist/main.js 
(()=>{"use strict";console.log(6)})();%  
```

加接下来，实验 loader：引入 hello.txt，通过 webpack 进行打包。

修改 index.js 引入 hello.txt，并且修改 Webpack.config.js 加入 raw loader。

```js
// src/index.js
import {add} from "./add"
import hello from "./hello.txt"
console.log(add(3, 3))
console.log(hello)
```
```js
// webpack.config.js
module.exports = {
  // entry point
  entry: "./src/index.js",
  mode: "development",
  module: {
    // 对import或require()中出现.txt后缀的文件
    // 用raw-loader转换后打包
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  }
}
```

准备 src/hello.txt，内容为 `"Hello webpack"`。

执行打包并查看运行结果（生成的 main.js 不易理解）：

```shell
% node ./dist/main.js 
6
"Hello webpack"
```

最后，实验通过 plugin 处理 html 文件。

修改 webpack.config.js 加载 plugin，这里用到的是 html-webpack-plugin，并且加入 ./src/index.html 文件。

```js
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
  // entry point
  entry: "./src/index.js",
  mode: "development",
  module: {
    // 对import或require()中出现.txt后缀的文件
    // 用raw-loader转换后打包
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ],
  // webpack 5版本要加上配置才能修改保存后即时刷新
  devServer: {
    watchFiles: ["./src/index.html"],
    compress: true,
    port: 8080
  }
}
```

```html
<!-- ./src/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello Webpack</title>
</head>
<body>
    <h1>Hello Webpack</h1>
</body>
</html>
```

执行打包并查看 dist/ 内容，可以看到 index.html 已加入目录：

```shell
% npx webpack
% ls ./dist
index.html      main.js
```

启动 dev server：

```shell
% npx webpack serve
```

打开 <http://localhost:8080>，修改 src/index.html 中的内容，可以看到页面会即时更新。

## 开发资源

- [Webpack Plugins 列表](https://webpack.docschina.org/plugins)
- [官网上手教程](https://webpack.docschina.org/guides/getting-started/)

此文章为2月Day5学习笔记，内容来源于极客时间《前端进阶训练营》
