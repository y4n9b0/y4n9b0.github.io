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

```bash
gradle :app:dependencies
```

以树状结构列出 module(这里是app module) 所有的依赖。

## 二、平铺 dependencies

gradle 自身提供的 dependencies 命令提供的树状结构在某些场合下并不是简洁明了，很多时候我们只想单纯知道整个应用使用了哪些依赖，把树状结构平铺并去重后就会方便很多。如下是我自定义的一个 flatDeps task：

```groovy
project.afterEvaluate {
    project.plugins.withId('com.android.application') {
        project.android.applicationVariants.all { variant ->
            predicate(variant)
        }
    }
    project.plugins.withId('com.android.library') {
        project.android.libraryVariants.all { variant ->
            predicate(variant)
        }
    }
}

def predicate(variant) {
    tasks.create(
            group: "dependency",
            name: "flatDeps${variant.name.capitalize()}",
            description: "flat all dependencies"
    ) {
        doLast {
            def logDir = file("$buildDir${File.separator}outputs${File.separator}logs")
                    .with(true) {
                        it.mkdirs()
                    }

            file("${logDir}${File.separator}flatDeps${variant.name.capitalize()}.txt")
                    .with(true) {
                        delete(it)
                    }
                    .withOutputStream { outputStream ->
                        Configuration configuration
                        try {
                            configuration = project.configurations."${variant.name}CompileClasspath"
                        } catch (Exception ignored) {
                            configuration = project.configurations."_${variant.name}Compile"
                        }
                        configuration.resolvedConfiguration.lenientConfiguration.allModuleDependencies
                                .sort(true, new Comparator<ResolvedDependency>() {
                                    @Override
                                    int compare(ResolvedDependency l, ResolvedDependency r) {
                                        def idL = l.module.id
                                        def idR = r.module.id
                                        def groupResult = idL.group <=> idR.group
                                        def artifactResult = idL.name <=> idR.name
                                        def versionResult = idL.version <=> idR.version
                                        return groupResult != 0 ? groupResult : (artifactResult != 0 ? artifactResult : versionResult)
                                    }
                                })
                                .each {
                                    def identifier = it.module.id
                                    outputStream.write("${identifier.group}:${identifier.name}:${identifier.version}\n".getBytes())
                                }
                    }
        }
    }
}
```

新建 flatDeps.gradle 脚本，写入上面的自定义 task，然后在相应的 module apply 该脚本，sync 之后即可执行相应 flavor 的 task：flatDepsRelese/flatDepsDebug。输出日志在 ${相应module}/build/outputs/logs/flatDeps.txt，输出的依赖按字母升序。

## 三、configuration

[Gradle 理解：configuration、dependency](https://blog.csdn.net/Gdeer/article/details/104815986){:target="_blank"}

## 四、dependencyInsight

```bash
gradle :app:dependencyInsight --configuration debugCompileClasspath --dependency com.jingdong.wireless.cdyjy:utils
```

反向查看一个 lib 被哪些 module 依赖。
