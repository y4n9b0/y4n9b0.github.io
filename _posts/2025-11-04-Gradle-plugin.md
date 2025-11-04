---
layout: post
title: Gradle plugin
date: 2025-11-04 16:00:00 +0800
categories: Gradle
tags: plugin
published: true
---

* content
{:toc}

之前写了个 gradle 脚本，输出 Android 模块最终的依赖，最近发现在多个项目使用时需要将脚本拷来拷去，有点费事儿，
于是打算将其发布为 gradle plugin，使用的时候直接依赖 plugin 即可。

## gradle plugin 发布配置

在插件模块的 build.gradle 里配置：
```gradle
plugins {
    id 'kotlin'
    id 'com.gradle.plugin-publish' version '2.0.0'
}

kotlin {
    jvmToolchain {
        languageVersion.set(JavaLanguageVersion.of(17))
    }
}

dependencies {
    implementation gradleApi()
    implementation "com.android.tools.build:gradle:8.13.0"
    implementation "org.jetbrains.kotlin:kotlin-stdlib:2.2.20"
    implementation "org.jetbrains.kotlin:kotlin-reflect:2.2.20"
}

group = 'io.github.y4n9b0'
version = '1.0.0'

gradlePlugin {
    website = 'https://github.com/y4n9b0/GradlePlugin'
    vcsUrl = 'https://github.com/y4n9b0/GradlePlugin'
    plugins {
        flatDeps {
            // 在 app 模块需要通过 id 引用这个插件
            id = 'io.github.y4n9b0.flatDeps'
            // 实现这个插件的类的路径
            implementationClass = 'y4n9b0.flatDeps.FlatDepsPlugin'
            displayName = 'Gradle flatDeps plugin'
            description = 'Gradle plugin to flat dependencies!'
            tags.set(['dependence', 'flat'])
        }
    }
}
```

## gradle plugin publish task & artifacts

配置好 plugin gradle 脚本并 sync 后，就可以在 gradle task 面板上看到 plugin 发布任务。<br>
这里以本地发布测试为例，发布 gradle plugin 任务：
```bash
./gradlew clean :flatDeps:publishToMavenLocal
```

生成的产物如下：
```bash
|____flatDeps
|    |____io.github.y4n9b0.flatDeps.gradle.plugin
|    |    |____1.0.0
|    |    |    |____io.github.y4n9b0.flatDeps.gradle.plugin-1.0.0.pom
|    |    |____maven-metadata-local.xml
|    |____1.0.0
|    |    |____flatDeps-1.0.0-javadoc.jar
|    |    |____flatDeps-1.0.0-sources.jar
|    |    |____flatDeps-1.0.0.jar
|    |    |____flatDeps-1.0.0.module
|    |    |____flatDeps-1.0.0.pom
|    |____maven-metadata-local.xml
```

gradle plugin 发布一般包含两个任务，一个是 main task，跟普通的 library 发布类似，是插件实现本体（真正的代码）；
另一个是 marker task，插件 marker artifact（标记文件）。<br>
上面的发布任务`./gradlew clean :flatDeps:publishToMavenLocal`汇总了这两个任务（严格地说，是汇总了所有的 mavenLocal task，只是这里碰巧只有这两个 mavenLocal task）

**gradle plugin main task & artifacts**
```bash
./gradlew clean :flatDeps:publishPluginMavenPublicationToMavenLocal
```

```bash
|____flatDeps
|    |____1.0.0
|    |    |____flatDeps-1.0.0-javadoc.jar
|    |    |____flatDeps-1.0.0-sources.jar
|    |    |____flatDeps-1.0.0.jar
|    |    |____flatDeps-1.0.0.module
|    |    |____flatDeps-1.0.0.pom
|    |____maven-metadata-local.xml
```

**gradle plugin marker task & artifacts**
```bash
./gradlew clean :flatDeps:publishFlatDepsPluginMarkerMavenPublicationToMavenLocal
```

```bash
|____flatDeps
|    |____io.github.y4n9b0.flatDeps.gradle.plugin
|    |    |____1.0.0
|    |    |    |____io.github.y4n9b0.flatDeps.gradle.plugin-1.0.0.pom
|    |    |____maven-metadata-local.xml
```

marker 任务名中的 `FlatDeps` 是插件 build.gradle 里配置的 plugin 变量：
```groovy
gradlePlugin {
    plugins {
        flatDeps {
        }
    }
}
```

marker 任务是用来标记插件本体的，实际真正生效的还是插件本体。<br>
是不是有种脱裤子放屁多此一举的感觉？<br>
这是因为我们引用插件的方式跟引用 library 的方式不一致：

```groovy
// 引用 plugin
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'io.github.y4n9b0.flatDeps' version '1.0.0'
}

// 引用 library
dependencies {
    implementation fileTree(dir: "libs", include: ['*.jar', '*.aar'])
    implementation 'androidx.core:core-ktx:1.17.0'
}
```

插件的引用和解析机制和 library 不一致，需要先寻找 marker，再根据 marker pom 文件解析找到插件本体进行使用。<br>
当然了，为什么要把插件引用设计成这屌逼样，就不得而知了😳。

## 发布到 jitpack

一开始打算把插件托管到 [jitpack](https://jitpack.io/){:target="_blank"}，使用 jitpack 有个好处是发布任务可以自动和 github 项目的 release tag 关联，
还有一个好处是之前也用过 jitpack 发布 library，相对比较熟悉，毕竟连水都是选择阻力最小的路径流淌。

配置好相关脚本后，发布到 jitpack 后发现确实可以生成两个任务的产物，但是路径不太对，导致最终引用的时候提示找不到插件。在网上也搜索了相关文章，比如
[Plugins published without Gradle Plugin Portal](https://docs.gradle.org/current/userguide/publishing_gradle_plugins.html#plugins_published_without_gradle_plugin_portal){:target="_blank"}，试过、不行、遂弃！

## 发布到 gradle plugin portal

[gradle plugin portal](https://plugins.gradle.org/){:target="_blank"} 是 gradle 官方的 plugin 托管中心，配置好任务后执行如下命令即可发布：
```
./gradlew clean :flatDeps:publishPlugins
```

如果只是验证（不上传）:
  ```bash
  ./gradlew clean :flatDeps:publishPlugins --validate-only
  ```

**一些注意事项**

* gradle plugin portal 需要注册，我使用的是 github 账号关联注册，这里有个坑，关联注册的时候要小心从 github 自动导入的邮箱，导入的邮箱并不是在 github 使用的真实邮箱，而是类似 xxx@users.noreply-github.com，该邮箱无法生成 API key，也无法发布插件，注册之前一定要先改为真实的邮箱。而我就比较悲催地没在意，最后只能找官方支持更改邮箱，BTW 官方支持邮箱😊：[plugin-portal-support@gradle.com](mailto:plugin-portal-support@gradle.com)

* gradle plugin portal 发布插件需要使用 API key，在个人 profile 页面生成 API key 后将其配置在 `$USER_HOME/.gradle/gradle.properties` 供登录使用：
    ```txt
    gradle.publish.key=${your gradle publish key}
    gradle.publish.secret=${your gradle publish secret}
    ```

* 发布插件有许多准则要求，详见 [https://plugins.gradle.org/docs/publish-plugin](https://plugins.gradle.org/docs/publish-plugin){:target="_blank"}，说几点主要的：
    * 一个新插件首次发布后需要官方批准通过，一般等待时间是1-2个工作日，通过的插件后续发布不再需要批准。
    * 插件必须能关联到作者，插件发布必须要配置对应的项目公开地址，大概率审批的时候会去阅读项目源码。
    * github 项目的插件使用 io.github 开头，不能使用 com.github（已废弃）。
    * 插件要有具体的功能，不能是 hello world 之类的，而且功能必须要普罗大众，个人特定使用的插件应该发布在私有仓库。

<!-- https://docs.gradle.org/current/userguide/publishing_gradle_plugins.html -->
<!-- https://plugins.gradle.org/docs/publish-plugin -->
<!-- https://docs.gradle.org/current/userguide/java_gradle_plugin.html#maven_publish_plugin -->
<!-- https://central.sonatype.org/changelog/#2021-04-01-comgithub-is-not-supported-anymore-as-a-valid-coordinate -->
