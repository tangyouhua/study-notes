# NPM 包管理器

> 极客时间前端进阶训练营练习项目—打卡-Day14，2023-1-29

## 是什么

包管理器是一种电脑中自动安装、配置、卸载和升级软件包的工具组合。

NPM（Node Package Manager） 是 Node 包管理器，包括：

- 存放第三方包的代码库*
- 管理本地安装包机制
- 定义包依赖关系的标准

**注意**：除了 Npm 仓库，还可以自己搭建私有仓库，例如 [Github Packages][pkg-mgr-github-packages]，[Verdaccio][pkg-mgr-verdaccio-packages]。

[pkg-mgr-github-packages]: https://github.com/features/packages
[pkg-mgr-verdaccio-packages]: https://verdaccio.org/

安装模式

- 本地模式：只在工作目录下工作，不会修改整个系统范围
- 全局模式：适合总是被全局加载的公共模块

配置

- 环境变量：对于全局安装的模块，可以把这个安装目录设为系统 PATH 环境变量的一部分。从而可以使用系统的环境变量
- 设置镜像源：解决网络速度与可访问性，参考 <https://npmmirror.com/>


## 基本命令

```shell
# 注册用户
npm adduser

# 登录
npm login

# 初始化项目
## 可指定用户
npm init --scope

# 安装/更新
npm install
## 本地模式
npm install vue
## 全局模式
npm install nodemon -g
## 指定版本
npm install vue@3.0.0
## 分支版本
npm install vue@3.0.x
## 小于某一个版本
npm install vue@"<3.0.x"
## 指定范围
npm install vue@">-1.0.0 <2.0.0"

# 卸载模块
npm uninstall vue

# 更新模块
npm update vue

# 发布模块
## 默认为 private package
npm publish --access public
## 发布新版本
### 修改 package.json 后
npm publish

# 查看发布结果
npm view

# 查看包依赖
npm ls
# 查看过期
npm outdated

# Dist Tag: 默认会拉取最新 lastest, 如果不希望误用可打上 tag
# 添加 Dist Tag
npm dist-tag add
# 删除 Dist Tag
npm dist-tag rm 
```

## 官方文档

- [npm 命令行][doc-npmjs]
- [配置 npm][doc-npmjs-configuration]
- [使用 npm][doc-npmjs-using]

## 教程与练习

- [how-to-npm][pkg-npmjs-how-to-npm] 是一个不错的上手练习：需要注册一个 npm 账号，一些涉及账号登陆的练习可能不通过但不影响学习。
- [创建与发布包][tut-npmjs-create-publish-package]

## 常见问题

[doc-npmjs]: https://docs.npmjs.com/cli/v9/commands
[doc-npmjs-configuration]: https://docs.npmjs.com/cli/v9/configuring-npm
[doc-npmjs-using]: https://docs.npmjs.com/cli/v9/using-npm
[pkg-npmjs-how-to-npm]: https://www.npmjs.com/package/how-to-npm
[tut-npmjs-create-publish-package]: https://docs.npmjs.com/creating-and-publishing-scoped-public-packages
