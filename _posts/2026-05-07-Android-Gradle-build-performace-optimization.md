---
layout: post
title: Android Gradle 构建性能优化
date: 2026-05-07 17:30:00 +0800
categories: Android
tags: Gradle
published: true
---

* content
{:toc}

## 1. Android Build 大致链路:

```txt
Gradle
  ↓
Configuration Phase
  ↓
Task Graph
  ↓
Execution Phase
  ↓
Kotlin / Java Compile
  ↓
KAPT / KSP
  ↓
Resource Merge
  ↓
Dex
  ↓
R8 / Proguard
  ↓
APK / AAB
```

## 2. 真正的优化目标其实只有三个：

* 减少 task 数量 - 少干活
* 减少 task 耗时 - 干得更快
* 避免重复执行 - 不重复干

## 3. 第一阶段：Build Profiling（先测量）

很多人一上来就“优化”，这是不对的，第一步永远是：Profiling（测量）。只有测量了才能知道哪些地方该优化，对症下药，而不是像只无头苍蝇一样盲目地“优化”。

## 4. 性能分析工具链

### 4.1 [Build scan](https://gradle.com/scans/gradle/){:target="_blank"}

```bash
./gradlew assembleDebug --scan
```
    
会生成：
* task timeline
* critical path
* cache hit
* configuration time

### 4.2 [gradle-profiler](https://github.com/gradle/gradle-profiler){:target="_blank"}

```bash
gradle-profiler --benchmark assembleDebug
```
    
用于：
* 多轮 benchmark
* warmup
* 对比优化前后

gradle-profiler 还支持使用指定 profiler，比如 Build scan（相当于包含了 Build scan 功能）：
```bash
gradle-profiler --profile buildscan assembleDebug
```

## 5. 性能瓶颈

### 5.1 Configuration Phase

configuration 阶段 Gradle 会先：
* 解析 build.gradle / build.gradle.ktx
* 创建 task
* 构建 dependency graph

大项目里会非常慢，因为：
* module 太多
* plugin 太多
* afterEvaluate 滥用
* 动态 task 创建

解决方案：开启 Configuration Cache
```txt
org.gradle.configuration-cache=true
```
开启 Configuration Cache 后，task graph 会被序列化缓存下来，下一次 build 的时候，符合重用条件就直接恢复 task graph。

在日志里一般会看到类似提示：
```txt
Reusing configuration cache.
```
说明这次没重新 configuration。如果是
```txt
Calculating task graph as configuration cache cannot be reused because ...
```
说明失效了，而且会告诉你原因。
要排查为什么没命中，Gradle 官方建议看 configuration cache 的 HTML 报告，它会列出配置阶段读取了哪些输入。
* [Gradle Configuration Cache 官方文档](https://docs.gradle.org/current/userguide/configuration_cache.html){:target="_blank"}
* [Configuration Cache 启用与失效](https://docs.gradle.org/current/userguide/configuration_cache_enabling.html){:target="_blank"}
* [Configuration Cache 调试文档](https://docs.gradle.org/current/userguide/configuration_cache_debugging.html){:target="_blank"}

开启 Configuration Cache 的最大难点是很多插件不兼容：
* 自定义 task
* AGP 老版本
* 非 lazy API

一般都会涉及到全链路改插件、迁移 Gradle Provider API。

// todo: gradle provider

### 5.2 KAPT

KAPT 之所以慢，是因为
```txt
Kotlin
  ↓
生成 Java Stub
  ↓
Java Annotation Processing
  ↓
再回 Kotlin
```
涉及：
* JVM
* AST
* Javac
* Kotlin Compiler

非常重

解决方案：替换为基于 Kotlin 的 KSP（Kotlin Symbol Processing）

## 6. Incremental Compilation（增量编译）

核心思想：改一个文件，不重新编全部。
开启：
```txt
kotlin.incremental=true
```

// todo: ABI 改动 / 非 ABI 改动
// todo: module A 改动，依赖 module A 的 module B 是否需要编译？以及 module B 依赖 module A 的方式 implementation/api，是否会引起依赖 module B 的模块编译

## 7. Build Cache

### 7.1 Local Cache

```txt
org.gradle.caching=true
```

### 7.2 Remote Cache

[Remote Cache](https://y4n9b0.github.io/2026/01/27/Optimizing-Gradle-Builds/#remote-cache){:target="_blank"}

## 8. Module 化

单 module 之所以慢，是因为改一个文件全项目都需要重新编译。<br>
但 module 不是越多越好，过多会导致：
* configuration 爆炸
* dependency graph 爆炸

个人觉得比较合理的模块化工程（自顶向下）：
* 业务模块：biz1、biz2、biz3 等，跟随产品需求频繁改动
* 基础模块：网络、日志、埋点、UI 组件等，业务关联度低，偏稳定，改动频次低，适合发布到 maven 仓库
* 工具模块：各种 util、扩展 之类，特点是业务无关，任何一个项目皆可使用，适合发布到 maven 仓库

## 9. dependency resolution

[dependency resolution](https://y4n9b0.github.io/2026/01/27/Optimizing-Gradle-Builds/#optimize-dependency-resolution){:target="_blank"}
