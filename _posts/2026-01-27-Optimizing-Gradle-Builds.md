---
layout: post
title: 优化 Gradle 构建
date: 2026-01-27 16:00:00 +0800
categories: build
tags: Gradle
published: true
---

* content
{:toc}

## Gradle 官方文档

[Build Configuration](https://docs.gradle.org/current/userguide/build_environment.html){:target="_blank"}

[Improve the Performance of Gradle Builds](https://docs.gradle.org/current/userguide/performance.html){:target="_blank"}

## 升级版本

### Gradle

Gradle 更新迭代相当快，不建议直接升到最新版本。我的经验是升级到最新版本的上一版本，这个时候问题暴露得差不多了，bug 也修复得差不多了，可以节省很多踩坑精力。
当然实际升级到什么新版本合适，还得结合各自的项目工程来决定。

### Java

Gradle 运行在 JVM，Java 新版本往往也会带来一些性能提升，项目 Java 版本过于老旧的也可以适当升升级。

### plugin

升级 Gradle 插件，新版本的 Gradle 插件大多都会适配更新的 Gradle 版本。
特别提一下 AGP，AGP、Gradle 和 Java 之间也有版本匹配的，而 Android 项目中 AGP 是版本选择的锚点，正确的升级顺序应为：
AGP → Gradle → Java。

### 启用并行编译

Gradle 默认同一时间只能运行一个 task，实际中大多数工程都会包含多个子工程，子工程又按是否牵扯业务进行分层，业务层依赖工具层，业务层间的工程通过 router 之类的框架解耦，
启用并行编译可以让那些相互不存在依赖的独立子工程同时编译。

```bash
./gradlew <task> --parallel
```

也可以配置在 gradle.properties 中

```
org.gradle.parallel=true
```

并行编译对工程模块化要求较高，如果模块化设计得不够好，模块之间依赖耦合严重，提升可能有限。

另外，基础框架类的子工程可以考虑打包成 .aar 发布到 maven，主工程使用在线依赖，会大幅度提升编译速度：
1. 减少参与构建的 module 数量，多模块 Android / Flutter 工程里，Gradle 主要慢在配置阶段，每个 module 都要：
	* evaluate build.gradle
	* 创建 task graph
	* 参与 variant / dependency 解析
	* Android 插件（AGP）对 module 数量非常敏感

    子模块改成 Maven 依赖后：
	* Gradle 不会 evaluate 子模块
	* 不会创建它们的 task
	* 不会走它们的 variant 计算

    配置阶段会明显变快

2. 命中 Build Cache / Artifact Cache，Maven 依赖是稳定的二进制产物，Gradle / AGP 可以直接复用 .aar，不需要重新编译 Kotlin / Java / AIDL / R / Manifest。

## 复用 gradle daemon 守护进程

```bash
./gradlew <task> --daemon
```

也可以配置在 gradle.properties 中

```
org.gradle.daemon=true
# org.gradle.daemon.idletimeout=(# of idle millis)
```

daemon 进程有空闲超时设置，默认 3 小时，超时会自动关闭，另外在面临内存压力时也会自动关闭。

## 启用 build cache

```bash
./gradlew <task> --build-cache
```

也可以配置在 gradle.properties 中

```
org.gradle.caching=true
```

对于一个特定输入的 task，build cache 会缓存 task 的输出，该 task 下一次运行同样的输入时直接复用缓存的输出。

需要注意的是，FROM-CACHE 和 UP-TO-DATE 不同，UP-TO-DATE 是在没 clean、没改代码的情况下，而 FROM-CACHE 是 clean 了、但没改代码，
一旦改了代码（即改了输入），task 就不会 UP-TO-DATE 也不会命中缓存。

cache task 的前提 task 必须是 cacheable，需要 Task 显式声明 @CacheableTask，自定义 task 切记。
可以通过 `./gradlew <task> --info` 或者 `./gradlew <task> --scan` 来观察 task 是否是 cacheable。

### 跨机器 build cache

build cache 可以跨机器。一般有两种方式：

1. 自建缓存服务器

    找一台公司内网机器，安装 Gradle 官方提供 Docker 镜像
    ```bash
    docker run -p 5071:5071 gradle/build-cache-node
    ```

    ```groovy
    // settings.gradle
    buildCache {
        local {
            enabled = true
        }

        remote(HttpBuildCache) {
            allowInsecureProtocol = true
            // 提供缓存服务器的地址和端口
            url = uri("https://ci-cache-server:5071")
           // 判断是否是 CI，只有 CI 环境才推送服务器，避免个人本地脏 cache 污染远端
           push = System.getenv("CI") != null
        }
    }
    ```

2. Gradle Enterprise（现在叫 [Develocity](https://gradle.com/develocity/product/test-distribution/){:target="_blank"}），官方 SaaS / 私有部署，据说国内很多财大气粗的企业都在使用，包含：
    * Remote Cache
	* Build Scan
	* 构建分析

### 使用 build scan 可视化 build cache
```bash
./gradlew <task> --scan
```

报告输出在官方服务器上，需要验证才能查看。没法直接输出本地查看，这点挺讨厌。

## 启用 [Configuration Cache](https://docs.gradle.org/current/userguide/configuration_cache.html#config_cache){:target="_blank"}

Gradle 的构建流程可以分为三个主要阶段：
1. Initialization，准备构建环境，确定要构建的 Project 以及创建对应的 **Project 对象**。
2. Configuration
    * 解析 build.gradle / settings.gradle 脚本。
	* 创建并配置**所有 task 对象**。
	* 确定 task 的依赖关系。
3. Execution，按依赖顺序执行 task 的 action（doFirst, doLast），检查 UP-TO-DATE / 缓存。

这里的启用配置缓存指的就是第二阶段 Configuration

```bash
./gradlew <task> --configuration-cache
```

也可以配置在 gradle.properties 中

```
org.gradle.configuration-cache=true
```

需要注意的是，启用 Configuration Cache 有风险：
    * 旧插件 / 自定义 task 非常容易不兼容
	* Project、Task 在配置阶段做 IO / 读环境变量 / 单例状态，都会直接报错
	* Android 项目中，AGP + 第三方插件是最大雷区

不要在未通过 --configuration-cache 校验前默认开启到全员环境，建议先在个人机器用 --configuration-cache 跑，逐个修复不兼容问题，通过后再考虑配置在 gradle.properties 全员开启。

## 自定义 task 启用[增量编译](https://docs.gradle.org/current/userguide/incremental_build.html#incremental_build){:target="_blank"}

这是高难度内容，错误声明 Input / Output 会导致：
* 缓存污染
* 产物不一致
* 难以排查的偶现 bug

建议非常熟悉 Gradle 的人才考虑，[writing tasks tutorial](https://docs.gradle.org/current/userguide/writing_tasks_intermediate.html){:target="_blank"}

## 增加 Heap Size

Heap Size 默认是 512MB

```
org.gradle.jvmargs=-Xmx2048M
```

## Only Apply Plugins where they’re needed

使用 `allprojects {}` 或者 `subprojects {}` 配置会对所有子工程生效，即便某些子工程根本不需要。按需配置。

## Optimize dependency resolution

1. 移除无用的 lib 依赖
2. 优化 lib 仓库顺序，Gradle 按仓库顺序搜索 lib，尽可能把拥有使用 lib 多的仓库排在前面
3. 减少仓库数量，尽可能使用聚合仓库。对于国内编译的项目，可以考虑使用阿里云的仓库：
    ```groovy
    maven { url = 'https://maven.aliyun.com/repository/public' }
    maven { url = 'https://maven.aliyun.com/repository/google/' }
    ```
4. 减少动态版本和快照版本
    ```groovy
    implementation 'groupId:artifactId:1.+'
    implementation 'groupId:artifactId:1.0.0-SNAPSHOT'
    ```
    诸如这样的依赖会导致 Gradle 频繁检查仓库是否有 lib 更新，Gradle 默认缓存动态版本 24 小时，可以通过配置来更改：

    ```groovy
    configurations.all {
        resolutionStrategy {
            cacheDynamicVersionsFor 4, 'hours'
            cacheChangingModulesFor 10, 'minutes'
        }
    }
    ```

    正常情况下，你不应该使用动态版本/快照版本（调试除外），特别是 release 分支，这不仅仅是频繁检查仓库的问题，而是两次编译可能使用不同版本依赖导致产物以及最终效果不一致，任何公司/个人都应该严禁 release 分支使用动态版本/快照版本。

5. 移除/优化自定义依赖策略，主要包含两个方面：
    * 强制指定版本号
        ```groovy
        configurations.all {
            resolutionStrategy.eachDependency { details ->
                if (details.requested.group == "com.example" && details.requested.name == "library") {
                    def versionInfo = new URL("https://example.com/version-check").text  // ❌ Remote call during resolution
                    details.useVersion(versionInfo.trim())  // ❌ Dynamically setting a version based on an HTTP response
                }
            }
        }
        ```
        尽可能地从源头解决依赖冲突，而不是强制指定版本号。
    * 复合构建替换依赖
        ```groovy
        includeBuild('../utils') { // 声明 included build 工程所在的目录，这里使用的是相对当前工程的位置
            dependencySubstitution {
                // 使用 utils 工程的 lib 模块替换其他构建 groupId=io.github.y4n9b0、artifactId=utils 的二进制依赖
                substitute module('io.github.y4n9b0:utils') with project(':lib')
            }
        }
        ```
        复合构建替换依赖基本只用在调试 lib 源码，其他情况大多都不应该使用。调试完必须移除，否则会反向拖慢整体构建。

6. 模块内部依赖使用 implementation 而不是 api，api 会将依赖暴露给当前模块的下游消费模块（依赖当前模块的模块），当 api 依赖有变动时也会引起下游消费模块重新编译，如果确定该依赖只是在当前模块使用，那么应该使用 implementation，implementation 依赖的变动不会引起下游消费模块重新编译。

## 优化 Android 工程

[Optimize your build performance (Android Developer Guide)](https://developer.android.com/studio/build/optimize-your-build.html){:target="_blank"}）

[Optimizing Gradle Builds for Android (Google I/O 2017 Talk)](https://www.youtube.com/watch?v=7ll-rkLCtyk){:target="_blank"}）

## gradle.properties 省流版

```
# 启用守护进程，避免每次启动 JVM
org.gradle.daemon=true
# 并行编译
org.gradle.parallel=true
# 配置缓存（gradle 7.+），⚠️ 有兼容风险，需验证
org.gradle.configuration-cache=true
# 并行配置缓存，依赖于 org.gradle.configuration-cache=true
org.gradle.configuration-cache.parallel=true
# 构件缓存（复用 task 输出）
org.gradle.caching=true
# 按需配置（减少解析）
org.gradle.configureondemand=true
```
