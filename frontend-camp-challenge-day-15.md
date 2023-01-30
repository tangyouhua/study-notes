# Monorepo

## 是什么

Monorepo（mono 独立 repo 仓库）是针对项目采用独立仓库的一种版本仓库结构。

在前端项目中，Monorepo 通常指用一个代码仓库管理多个 package 的情况。

## 为什么

相比较用多个代码仓库来管理项目中的 package，Monorepo 有以下的优势与缺点：

优点

- 工作流一致
- 项目基建成本低
  - 例如某个需求要改多个 package 不需要到各自的仓库去修改、测试、发布或者 npm link
- 降低依赖管理复杂度
- 团队协作更容易

更多优点可参见[这里][doc-monorepo-features]。

## 备选方案

Monorepo 工具很多，包括

- Bazel
- Gradle
- Lage
- Lerna
- Nx
- Pants
- Rush
- Turborepo

详细的功能对比参见[这里][doc-monorepo-tools]。

## 实现案例

这里通过 pnpm 实现 Monorepo。

- 创建 monorepo 工作区
- packages 目录分别存放：smarty-admin 包，@smarty-admin/utils 包

目录结构

```shell
.
├── package-lock.json
├── package.json
├── packages
│   ├── admin
│   │   ├── package.json
│   │   └── test.js
│   └── utils
│       ├── index.js
│       └── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
└── scripts
    └── preinstall.js
```

执行命令如下：

1. 初始化外层包

```shell
pnpm init
```

2. 确保使用pnpm作为包管理

package.json 增加 preinstall 脚本

```json
"scripts": {
  "preinstall": "node ./scripts/preinstall.js",
  "test": "echo \"Error: no test specified\" && exit 1"
},
```

脚本负责检查执行路径是否包含 pnpm，否则给出警告并终止执行。内容如下：

```js
if (!/pnpm/.test(process.env.npm_execpath || '')) {
    console.warn(`\u001b[33mThis repository requires using pnpm as the package management]` +
        ` for scripts to work properly.\u001b[39m\n]`)
    process.exit(1)
}
```

3. 建立工作空间

新建 pnpm-workspace.yaml，内容如下：

```yaml
packages:
  # all packages in subdirs of packages/ and components
  - "packages/**"
```

4. 配置 package

新建 packages/admin，packages/utils 目录

```shell
cd packages/admin
npm init # 包名 smarty-admin
```

```shell
cd packages/utils
npm init # 包名 @smarty-admin/utils
```

为 utils 包添加 index.js 为后面测试准备。

```js
module.exports = 'This is utils package'
```

6. 安装最外层包

```shell
pnpm i vue -w
```

7. 安装子包 utils 依赖

```shell
pnpm i vue -r --filter smarty-admin
```

8. 添加子包 admin 依赖

```shell
pnpm i react -r --filter @smarty-admin/utils
```

9. 为smarty-admin安装本地包utils

```shell
pnpm i @smarty-admin/utils -r --filter smarty-admin
```

10. 测试本地包

为 smarty-admin 包增加 test.js，打印 utils 中的变量。

test.js

```js
const utils = require('@smarty-admin/utils')
console.log('utils: ', utils)
```

执行测试：

```shell
node test.js
utils:  This is utils package
```

[示例项目仓库](https://github.com/tangyouhua/lab-monorepo-smarty)### Monorepo

## 官方文档

- [Monorepo 是什么][doc-monorepo-what-is-a-monorepo]
- [为什么采用 Monorepo][doc-monorepo-why-a-monorepo]：对比 Polyrepo
- [Monorepo 有什么特点][doc-monorepo-features]

[doc-monorepo-why-a-monorepo]: https://monorepo.tools/#why-a-monorepo
[doc-monorepo-what-is-a-monorepo]: https://monorepo.tools/#what-is-a-monorepo
[doc-monorepo-features]: https://monorepo.tools/#monorepo-features
[doc-monorepo-tools]: https://monorepo.tools/#monorepo-tools
