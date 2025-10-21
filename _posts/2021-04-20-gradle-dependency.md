---
layout: post
title:  gradle dependency
date:   2021-04-20 11:00:00 +0800
categories: gradle
tags: gradle
published: true
---

* content
{:toc}

## 前言

工程组件化带来比较显著的依赖冲突，如何快速高效地解决冲突显得尤为重要。

## 一、树状 dependencies

以树状结构列出 module(这里是app module) 所有的依赖：

```bash
gradle :app:dependencies
```

## 二、平铺 dependencies

gradle 自身提供的 dependencies 命令提供的树状结构在某些场合下并不是简洁明了，很多时候我们只想单纯知道整个应用使用了哪些依赖，把树状结构平铺并去重后就会方便很多。如下是我自定义的一个 flatDeps task：

```groovy
project.afterEvaluate {
    def allVariantsFlatDepsTasks = []

    project.plugins.withId('com.android.application') {
        project.android.applicationVariants.all { variant ->
            predicate(variant)
            allVariantsFlatDepsTasks << "flatDeps${variant.name.capitalize()}"
        }
    }

    project.plugins.withId('com.android.library') {
        project.android.libraryVariants.all { variant ->
            predicate(variant)
            allVariantsFlatDepsTasks << "flatDeps${variant.name.capitalize()}"
        }
    }

    tasks.register("flatDeps") {
        group = "dependency"
        description = "flat all dependencies for all variants"
        dependsOn allVariantsFlatDepsTasks
    }
}

def predicate(variant) {
    def reset = "\033[0m"      // 重置所有样式
    def bold = "\033[1m"       // 加粗
    def red = "\033[31m"       // 红色，用于错误
    def green = "\033[32m"     // 绿色，用于成功
    def yellow = "\033[33m"    // 黄色，用于告警

    Configuration configuration = project.configurations.findByName("${variant.name}CompileClasspath")
            ?: project.configurations.findByName("${variant.name}Compile")
    tasks.register("flatDeps${variant.name.capitalize()}", DefaultTask) { task ->
        task.group = "dependency"
        task.description = "flat all dependencies"
        task.doLast {
            def logDir = layout.buildDirectory.dir("outputs/logs").get().asFile
            if (!logDir.exists()) logDir.mkdirs()

            def outFile = new File(logDir, "flatDeps${variant.name.capitalize()}.txt")
            if (outFile.exists()) outFile.delete()

            outFile.withWriter('UTF-8') { writer ->
                if (configuration == null) {
                    throw new GradleException("${red}No configuration found for ${bold}:${project.name}:${variant.name}${reset}")
                }
                configuration.resolvedConfiguration.lenientConfiguration.allModuleDependencies
                        .sort { l, r ->
                            l.module.id.group <=> r.module.id.group ?:
                                    l.module.id.name <=> r.module.id.name ?:
                                            l.module.id.version <=> r.module.id.version
                        }
                        .each { dep ->
                            def mid = dep.module.id
                            writer.writeLine("${mid.group}:${mid.name}:${mid.version}")
                        }
            }
        }
    }
}
```

新建 flatDeps.gradle 脚本，写入上面的自定义 task，然后在相应的 module apply 该脚本，sync 之后即可执行相应 flavor 的 task：flatDepsRelese/flatDepsDebug。输出日志在 ${相应module}/build/outputs/logs/flatDeps.txt，输出的依赖按字母升序。<br>
也可以执行总任务 flatDeps，同时输出 debug 和 release 的依赖日志。

## 三、configuration

[Gradle 理解：configuration、dependency](https://blog.csdn.net/Gdeer/article/details/104815986){:target="_blank"}

## 四、dependencyInsight

反向查看一个 lib 被哪些 module 依赖：

```bash
gradle :app:dependencyInsight --configuration debugCompileClasspath --dependency com.jingdong.wireless.cdyjy:utils
```

如果配置有 flavor，则需要加上 flavor 名字（参考 [Why did gradlew :app:dependencyInsight fail](https://stackoverflow.com/a/61530873/7368406){:target="_blank"}）：

```bash
gradle :app:dependencyInsight --configuration ${flavor}DebugCompileClasspath --dependency com.jingdong.wireless.cdyjy:utils
```

* `app` 替换成需要查看的 module name
* `debugCompileClasspath` 替换成对应的 configuration。
   如何查看所有的 configuration：`gradle projects --info`，找到对应 module 的 Configure project section。
   * 参考 [How can I get a list of all configurations for a Gradle project?](https://stackoverflow.com/questions/41173616/how-can-i-get-a-list-of-all-configurations-for-a-gradle-project){:target="_blank"}
* `com.jingdong.wireless.cdyjy:utils` 替换成需要查看的 library(group:artifact)
