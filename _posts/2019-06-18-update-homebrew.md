---
layout: post
title:  update homebrew
date:   2019-06-18 10:30:00 +0800
categories: tool
tags: homebrew
published: true
---

* content
{:toc}

## 前言

[Homebrew](https://brew.sh/) 默认的更新源存放在 [GitHub](https://github.com/Homebrew) 上，导致墙内用户访问缓慢，一般来说我们会更倾向选择国内提供的更新源，推荐[中科大](https://mirrors.ustc.edu.cn/)以及[清华大学](https://mirrors.tuna.tsinghua.edu.cn/)提供的更新源。

## 替换更新源

```bash
# 替换 brew.git:
$ cd "$(brew --repo)"
# 中科大:
$ git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
# 清华:
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

# 替换 homebrew-core.git:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
# 中科大:
$ git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
# 清华:
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

# 替换 homebrew-bottles:
# 中科大:
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile
# 清华:
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile

# 应用生效:
$ brew update
```

## 重置更新源

如果想换回官方源，只需要删除环境变量的 HOMEBREW_BOTTLE_DOMAIN 设置并重置 homebrew git remote url：

```bash
# 重置 brew.git:
$ cd "$(brew --repo)"
$ git remote set-url origin https://github.com/Homebrew/brew.git

# 重置 homebrew-core.git:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git remote set-url origin https://github.com/Homebrew/homebrew-core.git

# 应用生效:
$ brew update
```

## 诊断 homebrew

homebrew 安装目录是基于 git 版本管理，如果 homebrew 折腾出了问题，可以很方便的使用 git 命令重置 homebrew：

```bash
# 诊断Homebrew的问题:
$ brew doctor

# 重置brew.git设置:
$ cd "$(brew --repo)"
$ git fetch
$ git reset --hard origin/master

# homebrew-core.git同理:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git fetch
$ git reset --hard origin/master

# 应用生效:
$ brew update  
```

## 升级 homebrew

```bash
# 升级 homebrew
$ brew upgrade

# 清理旧有软件安装包
$ brew cleanup
```
