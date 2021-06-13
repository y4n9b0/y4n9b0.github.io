---
layout: post
title:  组件化工程版本控制
date:   2020-09-07 11:00:00 +0800
categories: Android
tags: component-based
published: true
---

* content
{:toc}

## 前言

## 一、组件化的三种结构

* 单 git 工程、多 module 模块
* 多 git 工程、多 module 模块（repo）
* 多 git 工程、独立 module 模块

## 二、 git submodule

### 1. 如何添加 submodule

```bash
# xx - 用户名， oo - 仓库名
git submodule add https://github.com/xx/oo.git
```

**Note**

* 需要注意的是这里尽量使用带 .git 后缀的 url，理论上 [https://github.com/xx/oo](https://github.com/xx/oo) 也可以，但后续使用会被告警并重定向至 [https://github.com/xx/oo.git](https://github.com/xx/oo.git)
* 默认情况下，子模块会将子项目放到一个与仓库同名的目录中。 如果想要放到其他地方，可以在命令结尾添加一个不同的路径作为参数。
* 添加成功后运行 `git status`，可以发现当前路径下不仅生成了 `oo` 目录，还多了一个 `.gitmodules` 文件，该文件记录当前工程所有子模块的链接配置。要重点注意的是，该文件也像 `.gitignore` 文件一样受到版本控制。 它会和该项目的其他部分一同被拉取推送，这样克隆该项目的人才会知道去哪获得子模块。

</br>

### 2. 克隆包含 submodule 的 git 工程

```bash
# 克隆主工程（这里的主工程仅针对子模块而言，并非组件化里的主工程）
git clone https://github.com/xx/MainProject

# 初始化本地配置文件
git submodule init

# 更新子模块所有数据
git submodule update
```

</br>
其中子模块的初始化和更新命令可以合二为一。如果子模块还嵌套包含子模块，则需要添加 `--recursive` 参数。

```bash
git submodule update --init
# or
git submodule update --init --recursive
```

</br>
当然，还有一种更简单的方式替代以上所有步骤，在 git clone 的时候传递参数 --recurse-submodules，它会自动初始化并更新仓库中的每一个子模块，包括可能存在的嵌套子模块。

```bash
# 克隆主工程，初始化所有子模块（包含嵌套子模块）并拉取数据
git clone https://github.com/xx/MainProject --recurse-submodules
```

</br>

### 3. submodule 只读工作模式

这是在项目中使用子模块的最简模型，就是只使用子项目并不时地获取更新，而并不在检出中进行任何更改。

</br>
进入子模块，手动拉取远程更新

```bash
git pull
# or
git fetch && git merge
```

</br>
如果不想在子目录中手动抓取与合并，那么还有种更容易的方式

```bash
git submodule update --remote
```

如果工程有很多子模块的话，默认会尝试更新所有子模块。只想更新特定子模块，可以传递对应的子模块名字作为参数。

```bash
# oo - 子模块名
git submodule update --remote oo
```

</br>
默认更新并检出子模块仓库的 master 分支。 可以通过设置 `.gitmodules` 指定其他分支。

```bash
# oo - 子模块名， dev - 分支名
git config -f .gitmodules submodule.oo.branch dev
```

如果不用 `-f .gitmodules` 选项，那么设置会保存在本地的 `.git/config` 文件中，而 `.git/config` 文件的配置只会为本地用户生效。但是在仓库中保留跟踪信息更有意义一些，因为其他人也可以得到同样的效果。

手动拉取子模块更新后，还需要把子模块的更新记录提交到当前主工程的远程仓库。

</br>
需要注意的是，如果只在主工程中使用 `git pull`，只会拉取到子模块更新的记录，并不会检出子模块的更新。
可以为 git pull 命令添加 --recurse-submodules 选项（从 Git 2.14 开始）。 这会让 Git 在拉取后运行 git submodule update，将子模块置为正确的状态。 此外，如果想让 Git 总是以 --recurse-submodules 拉取，可以将配置选项 submodule.recurse 设置为 true （从 Git 2.15 开始可用于 git pull）。此选项会让 Git 为所有支持 --recurse-submodules 的命令使用该选项（除 clone 以外）。

```bash
git pull --recurse-submodules

# 全局设置 git 命令默认添加参数 --recurse-submodules
git config --global submodule.recurse true
```

</br>

### 4. submodule 协作模式

如果我们在主项目中提交并推送但并不推送子模块上的改动，其他尝试检出我们修改的人会遇到麻烦， 因为他们无法得到依赖的子模块改动。那些改动只存在于我们本地的拷贝中。

可以让 Git 在推送到主项目前检查所有子模块是否已推送。 git push 命令接受可以设置为 “check” 或 “on-demand” 的 --recurse-submodules 参数。 如果任何提交的子模块改动没有推送那么 “check” 选项会直接使 push 操作失败。

```bash
git push --recurse-submodules=check

# 设置对所有推送都执行子模块检查
git config push.recurseSubmodules check
```

另一个选项是使用 “on-demand” 值，它会尝试帮助你推送子模块（如果子模块因为某些原因推送失败，主项目也会推送失败）。

```bash
git push --recurse-submodules=on-demand

# 设置对所有推送都执行子模块推送
git config push.recurseSubmodules on-demand
```

</br>

### 子模块更新 url

方式一：

```bash
# 将新的 URL 复制到本地配置中
$ git submodule sync --recursive

# 从新 URL 更新子模块
$ git submodule update --init --recursive
```

方式二：手动改 .gitmodules 和 .git/modules/oo/config （oo - 子模块名）

**Note** 新版本的 git，子模块会将它们的所有 git 数据保存在顶级项目的 .git 目录中

</br>

### 子模块技巧

* 子模块遍历：foreach 子模块命令

    ```bash
    git submodule foreach 'git stash'
    git submodule foreach 'git checkout -b featureA'
    git diff && git submodule foreach 'git diff'
    ```

* 如果你设置了配置选项 status.submodulesummary，git status 也会显示你的子模块的更改摘要

    ```bash
    # 设置 git status 显示子模块更改摘要
    git config status.submodulesummary 1
    ```

* git diff --submodule，查看子模块新添加提交记录，也可以通过设置 diff.submodule 为 “log” 来将其作为默认行为

    ```bash
    # 设置 git diff 显示子模块更改记录
    git config --global diff.submodule log
    ```

</br>

### git submodule update --init 和 --remote的区别

**Todo** [https://www.cnblogs.com/hustcpp/p/11851244.html](https://www.cnblogs.com/hustcpp/p/11851244.html)

## 三、BuildSrc

* 版本统一配置
* lint检查

<!-- https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97 -->
<!-- https://juejin.im/post/6844903615346245646 -->
<!-- https://www.jrebel.com/blog/using-buildsrc-custom-logic-gradle-builds -->
