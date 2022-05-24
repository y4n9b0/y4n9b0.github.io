---
layout: post
title:  Multiple SSH keys
date:   2022-02-02 22:00:00 +0800
categories: ssh
tags: ssh
published: true
---

* content
{:toc}

最近开了个 github 小号，由于 github 弃用 http 验证方式，所以必须使用 ssh 验证。原本以为很简单的事儿，过程却出乎意料。

## ssh config

## ssh agent

获取 gradle 主版本号：
```groovy
// 需要显式导包，否则部分 gradle 版本无法找到 GradleVersion 类
import org.gradle.util.GradleVersion

// 获取 gradle 的主版本号
def static getGradleMajorVersion() {
    final semVerList = GradleVersion.current().version.tokenize('.')
    try {
        return semVerList[0].toInteger()
    } catch (Exception e) {
        println("wtf: ${e.message}")
    }
    return 0
}
```

最后说点题外话，github 账号有 rename 功能，出于某些原因我并没有改名，而是选择开个新号。

<!-- https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account -->
<!-- https://docs.gitlab.com/ee/ssh/index.html -->
<!-- https://stackoverflow.com/questions/21160774/github-error-key-already-in-use -->
<!-- https://stackoverflow.com/questions/2419566/best-way-to-use-multiple-ssh-private-keys-on-one-client -->
<!-- https://gist.github.com/jexchan/2351996 -->
<!-- https://zhuanlan.zhihu.com/p/110413836 -->
<!-- https://www.cnblogs.com/greencollar/p/14363535.html -->