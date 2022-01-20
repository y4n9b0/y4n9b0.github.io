---
layout: post
title:  gradle version
date:   2022-01-20 10:30:00 +0800
categories: gradle
tags: gradle
published: true
---

* content
{:toc}

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
