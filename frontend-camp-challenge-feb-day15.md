# 持续集成（1）

> 前端进阶训练营笔记-2月打卡-Day15，2023-2-20

理解持续集成基本概念，动手实践。

## 目标

理解持续集成基本概念。

## 是什么

持续集成包含两部分，持续集成与持续发布。

- 前者是通过某些触发条件，结合代码管理工具，将项目打包成待发布的产品。
- 后者是通过自动化的脚本，将产品（程序）部署到目标环境。

前端持续集成涉及到以下方面：

- 持续集成阶段：代码静态检查 Eslint、编译与构建（Webpack 或 Rollup）、回归单元测试、版本验证测试、集成测试、端到端测试；
- 持续交付阶段：部署到应用服务器（Nginx）、部署到软件包管理器（Npm）、Webhook消息推送。

## 为什么

提高开发与发布的效率。

## 怎么做

通常会采用持续集成服务器，结合项目的实际情况通常有以下选择：

- Github Action：官方文档 [https://docs.github.com/en/actions](https://docs.github.com/en/actions)
- GitLab CI：官方文档 [https://docs.gitlab.com/ee/ci/](https://docs.gitlab.com/ee/ci/)
- Jenkins
- Circle CI

### 案例实战

下面以 GitHub Action 为例，介绍典型流程：

1. 开发示例项目：[https://github.com/su37josephxia/lego-react](https://github.com/su37josephxia/lego-react)
2. 编写发布脚本：[https://github.com/su37josephxia/lego-react/blob/main/.github/workflows/publish-aliyun-ecs.yml](https://github.com/su37josephxia/lego-react/blob/main/.github/workflows/publish-aliyun-ecs.yml)
3. 配置发布所需的变量，例如 ACCESS_TOKEN、SERVER_IP、TARGET
4. 开发修改代码，提交 main 分支或者提交 Pull Request 到 main 分支
5. 开始持续构建与发布，可以在项目 Action 页面查看：[https://github.com/su37josephxia/lego-react/actions](https://github.com/su37josephxia/lego-react/actions)
6. 如果构建成功，可通过部署服务器的公开地址查看修改结果

### 脚本分析

- 名称：name
- 触发条件：on
- jobs：
    - 构建：基于 ubuntu-latest 镜像，按照 steps 编写的步骤进行构建。
    - 部署：通过 [https://github.com/easingthemes/ssh-deploy](https://github.com/easingthemes/ssh-deploy) 项目通过 ssh 方式进行部署
        - 前提：在目标阿里云服务器上，创建好 ssh 公钥私钥对；
        - 注意：对于多用户使用的服务器，需要为 actions 单独创建，避免覆盖 .ssh/ 目录，影响其他用户。

```YAML
name: Publish to Aliyun

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # 下载代码
    - uses: actions/checkout@v2
    # 安装Nodejs
    - uses: actions/setup-node@v2
      with:
        node-version: 14
    - run: yarn
    - run: yarn build
    # 部署到阿里云
    - name: Deploy to Aliyun
      uses: easingthemes/ssh-deploy@v2.1.1
      env:
        SSH_PRIVATE_KEY: ${{ secrets.ACCESS_TOKEN }}
        ARGS: "-avzr --delete"
        SOURCE: "build/"
        REMOTE_HOST: ${{ secrets.SERVER_IP }}
        REMOTE_USER: "root"
        TARGET: ${{ secrets.TARGET }}
```

## 总结

- 持续集成包括集成、发布两个环节，有 GitHub Action、GitLab CI 等工具，需要结合项目实际需求动手实践。

此文章为2月Day15学习笔记，内容来源于极客时间《前端进阶训练营》
