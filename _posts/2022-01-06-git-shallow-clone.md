---
layout: post
title:  Git shallow clone
date:   2022-01-06 11:30:00 +0800
categories: git
tags: git
published: true
---

* content
{:toc}

## 一、shallow clone

shallow clone 浅拷贝。<br>
通过 `depth` 控制 git history 的数量就叫做 git shallow clone。

```bash
git clone -–depth [depth] [remote-url]
```

git 为什么需要浅拷贝？<br>
众所周知，git 的每一个 commit 节点都是一份完整的 copy，而不是 diff，这就导致了 随着 commit history 的增长 git 工程体积几乎线性膨胀（如果是 diff patch 则远低于线性膨胀）。<br>
有时候上 Github 借鉴别人的代码，我们仅仅想下载到本地运行看下效果，这个时候并不需要 commit history，浅拷贝就可以减少拷贝体积，缩短克隆时间（Github 的速度，原因众所周知）

## 二、unshallow

对于 shallow clone 的代码，随着借鉴的深入，需要追踪 history 了解原作者的设计意图，这个时候 git log 发现本地只有一个 commit 记录。<br>
如何做才可以把完整的 commit history 重新拉下来呢？很简单，拉取对应 git fetch，shallow 的反面 unshallow：

```bash
git fetch --unshallow
```

<!-- https://www.perforce.com/blog/vcs/git-beyond-basics-using-shallow-clones -->
<!-- https://ssoor.github.io/2020/03/25/git-clone-depth/ -->