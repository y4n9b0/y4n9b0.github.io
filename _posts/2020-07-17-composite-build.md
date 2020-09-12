---
layout: post
title:  Composite build
date:   2020-07-17 14:30:00 +0800
categories: compile
tags: gradle
published: true
---

* content
{:toc}

## 前言

组件化工程进行联合调试是一件非常麻烦的事情：

1. 组件工程修改代码并提交
2. 组件工程打包并上传 maven 仓库
3. 主工程更新组件工程在 maven 仓库中的版本号
4. 主工程编译、安装、联调
5. 如果组件工程本次修改未能达到预期效果，则又回到步骤1从头来过

以上操作不仅步骤繁多、耗时耗力，而且会造成大量无意义的 commit history。
Gradle 提供了一种叫复合构建的机制可以帮助我们缓解这种困局。

## 一、什么是复合构建

复合构建是指包含其他构建的构建。复合构建和多模块工程构建有点类似，不同的是复合构建包含的是单个完整的工程。

复合构建能做什么：

* 联合那些独立开发的构建，比如修复应用依赖的某个 library 的一个 bug。
* 把一个很大的多模块工程拆解成很多相互隔离的小工程，这些工程可以独立工作或者按需协同工作。

复合构建（composite build）里被包含的构建（included build），不共享任何配置给复合构建 或者其他被包含的构建，每一个被包含的构建的配置和执行都是孤立的。

被包含的构建通过依赖替换影响其他构建，默认的隐式替换规则是 被包含构建的工程使用 `${project.group}` 作为 `groupId`、`${project.name}` 作为 `artifactId` 去匹配其他构建的依赖。由于工程目录原因，可能导致隐式替换规则不满足需求，这时我们也可以在当前工程的 `settings.gradle` 文件里显式的声明替换规则，比如：

```groovy
includeBuild('../utils') { // 声明 included build 工程所在的目录，这里使用的是相对当前工程的位置
    dependencySubstitution {
        // 使用 utils 工程的 lib 模块替换其他构建 groupId=com.step2hell、artifactId=utils 的二进制依赖
        substitute module('com.step2hell:utils') with project(':lib')
    }
}
```

## 二、复合构建的三种方式

### 1. 命令行声明复合构建

使用 gradle 参数 --include-build 指定被包含的构建工程

```bash
gradle clean assembleDebug --include-build ../utils
```

### 2. 在 settings.gradle 声明

```groovy
includeBuild '../utils'
```

如果还需要显式依赖替换，则改写成如下方式：

```groovy
includeBuild('../utils') { // 声明 included build 工程所在的目录，这里使用的是相对当前工程的位置
    dependencySubstitution {
        // 使用 utils 工程的 lib 模块替换其他构建 groupId=com.step2hell、artifactId=utils 的二进制依赖
        substitute module('com.step2hell:utils') with project(':lib')
    }
}
```

### 3. 定义一个单独的混合构建工程

```groovy
// settings.gradle
rootProject.name = 'composite'

includeBuild 'app'
includeBuild 'utils'
```

通过 `dependsOn` 把 task 委托给被包含的构建执行

```groovy
// build.gradle
tasks.register('assembleDebug') {
    dependsOn gradle.includedBuild('app').task(':assembleDebug')
}
```

这种方式的好处在于 不需要改动真正的工程代码，需要依赖线上二进制包时执行 app 工程，需要依赖源码工程进行调试时执行 composite 工程。麻烦之处在于需要把所有可能用到的 task 都进行委托。

## 三、复合构建的限制

* 构建的 `rootProject.name` 不能相同
* gradle plugin 版本号必须保持一致
* 如果多个复合构建包含同一个构建，那么多个复合构建并行运行可能会导致冲突。gradle 在调用期间不会共享被包含的构建工程锁。

<!-- https://docs.gradle.org/current/userguide/composite_builds.html -->
<!-- https://blog.gradle.org/introducing-composite-builds -->