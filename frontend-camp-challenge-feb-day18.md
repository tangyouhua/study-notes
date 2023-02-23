# git版本管理

> 前端进阶训练营笔记-2月打卡-Day18，2023-2-23

git版本管理主要涉及到几个方面，一个是日常使用，一个是从理解版本管理git的概念角度来学习。

## 日常使用

在官方的 every day git 教程里，展示了不同角色日常可能需要了解的 git 命令。

教程链接：[https://git-scm.com/docs/giteveryday](https://git-scm.com/docs/giteveryday)

### 独立开发

```Bash
git init # 创建新仓库
git log # 查看记录
git switch # 切换分支
git branch # 切换分支
git add # 管理 index 文件
git diff # 查看过程中的修改
git status # 查看过程中的修改
git commit # 推进当前分支
git restore # 撤销修改
git merge # 合并本地分支
git rebase # 维护分支的描述
git tag # 标记某次提交
```

### 项目开发

```Bash
git clone # 从上游分支拉取到本地仓库
git pull # 从origin保持与上游最新修改同步
git fetch # 从origin保持与上游最新修改同步
git push # 分享仓库
git request-pull # 创建upstream待拉取的变更摘要 
```

### 集成

```Bash
git pull # 从信任的同事合并修改
git format-patch # 为贡献者发送准备好的修改
git revert # 回退糟糕的修改
git push # 发布最新版本
```

## 版本管理方面面面

官方文档：[https://git-scm.com/docs](https://git-scm.com/docs)

todo：此部分的命令待补充注释，并在后续的文章中介绍实际案例。

### 安装与配置

```Bash
git #  内容管理跟踪
git config # 设置与获取仓库或全局配置
git help # 显示git帮助信息

```

### 获取与创建项目

```Bash
git init # 创建空的仓库或者重新初始化已有仓库
git clone # 
```

### 基本镜像

```Bash
git add 
git status
git diff
git commit
git notes
git restore
git reset
git rm
git mv
```

### 分支与合并

```Bash
git branch
git checkout
git switch
git merge
git mergetool
git log
git stash
git tag
git worktree
```

### 分享与更新项目

```Bash
git fetch
git pull
git push
git remote
git submodule
```

### 监视与比较

```Bash
git show
git log
git diff
git difftool
git range-diff
git shortlog
git describe

```

### 补丁

```Bash
git apply
git cherry-pick
git diff
git rebase
git revert
```

### 调试

```Bash
git bisect
git blame
git grep
```

### 管理

```Bash
git clean
git gc
git fsck
git reflog
git filter-branch
git instaweb
git archive
git bundle
```

### 垂直命令

```Bash
git cat-file
git check-ignoreg
git checkout-index
git commit-tree
git count-objects
git diff-index
git for-each-ref
git hash-object
git ls-files
git ls-tree
git merge-base
git read-tree
git rev-list
git rev-parse
git show-ref
git symbolic-ref
git update-index
git update-ref
git verify-pack
git write-tree
```

## git 命令行的命名规范

官方文档：[https://git-scm.com/docs/gitcli](https://git-scm.com/docs/gitcli)

### 命名规范

- 选项在前，参数在后
- 版本号在前，路径在后
- 如果版本与路径不好区分，可以在中间加上 —，例如 git diff — HEAD
- 基于上面的规则，— 不能用来作为选项与版本的分隔符
- 许多命令支持在路径中使用通配符，例如 git restore *.c
- 使用英文句号表示当前路径

### 万能选项

```Bash
-h # 打印命令用法
--help-all # 列出完整的选项列表

```

### 选项取反

```Bash
--no-
# 示例
git branch --no-track

```

### 聚合多个短选项

```Bash
# 示例
git rm -rf
git clean -fdx
```

### 简写长选项

```Bash
git commit --amen # 替代 --amend
```

### 使用建议

- 推荐：git foo，不推荐：git-foo
- 推荐：git foo -a -b，不推荐：git foo -ab
- 选项需要一个参数
    - 推荐：git foo -oArg，不推荐： git foo -o Arg；
    - 推荐：git foo —long-opt=Arg，不推荐：git foo —long-opt Arg
- 参数中有版本时，注意与工作目录下的文件名称不要冲突
    - 推荐：git log -l HEAD —，不推荐：git log -l HEAD
- 建议不要使用简写，因为后面可能会有新的命令与该简写冲突


此文章为2月Day18学习笔记，内容来源于 git 官方文档。
