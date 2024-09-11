---
layout: post
title: Gradle maven 发布 library 后自动创建 git tag
date: 2024-09-11 11:00:00 +0800
categories: buildscript
tags: gradle
published: true
---

* content
{:toc}


## git-tag.gradle

```groovy
def reset = "\033[0m"      // 重置所有样式
def bold = "\033[1m"       // 加粗
def red = "\033[31m"       // 红色，用于错误
def green = "\033[32m"     // 绿色，用于成功
def yellow = "\033[33m"    // 黄色，用于告警

tasks.register('createGitTag') {
    def projectName = project.name
    def projectDir = project.projectDir
    def tagName = "v${POM_VERSION_NAME}"
    def snapshot = POM_VERSION_NAME.endsWithIgnoreCase("SNAPSHOT")

    configure {
        onlyIf {
            def publishSuccess = tasks.withType(PublishToMavenRepository).every { publishTask ->
                publishTask.state.failure == null
            }
            if (!publishSuccess) {
                println "${yellow}Skipping $bold$projectName$reset$yellow git tag creation because the publish task failed.$reset"
            } else if (snapshot) {
                println "Skipping $bold$projectName$reset git tag creation because the publish is SNAPSHOT."
            }
            // 快照版 或者 publish 任务失败跳过 createGitTag
            !snapshot && publishSuccess
        }
    }

    doLast {
        println "Creating $bold$projectName$reset git tag $tagName."
        def command = "git --git-dir=$projectDir/.git --work-tree=$projectDir tag $tagName && git --git-dir=$projectDir/.git --work-tree=$projectDir push origin $tagName"
        def process = ["sh", "-c", command].execute()
        process.waitFor()
        def code = process.exitValue()
        if (code == 0) {
            println "$green$bold$projectName$reset$green git tag $tagName created successfully.$reset"
        } else {
            println "${red}Failed to create $bold$projectName$reset$red git tag $tagName, errCode: $code!$reset"
        }
    }
}

// 在发布任务结束后执行 createGitTag 任务
tasks.withType(PublishToMavenRepository).configureEach {
    finalizedBy createGitTag
}
```

## mvn-publish-lib.gradle

```groovy
/**
 * https://docs.gradle.org/current/userguide/publishing_maven.html
 * https://developer.android.google.cn/studio/build/maven-publish-plugin
 */
apply plugin: 'maven-publish'
publishing {
    publications {
        release(MavenPublication) {
            afterEvaluate { from components.release }
            groupId = 'com.y4n9b0'
            artifactId = POM_ARTIFACT_ID
            version = POM_VERSION_NAME
        }
    }
    repositories {
        maven {
            allowInsecureProtocol = true
            url = POM_VERSION_NAME.endsWithIgnoreCase("SNAPSHOT")
                    ? "http://maven.xxx.com/artifactory/libs-snapshot"
                    : "http://maven.xxx.com/artifactory/libs-release"
            credentials {
                // ~/.gradle/gradle.properties
                username "${MAVEN_USERNAME}"
                password "${MAVEN_PASSWORD}"
            }
        }
    }
}

apply from: "${rootDir}/git-tag.gradle"
```

## module build.gradle

git-tag.gradle 和 mvn-publish-lib.gradle 都放在根目录<br>
在对应需要发布的 module build.gradle 文件里依赖发布脚本并配置发布相关变量即可：

```groovy

ext {
    POM_ARTIFACT_ID = "util"
    POM_VERSION_NAME = "1.0.0"
}

if (hasProperty('MAVEN_USERNAME') && hasProperty('MAVEN_PASSWORD')) {
    apply from: "${rootDir}/mvn-publish-lib.gradle"
}
```

## 自定义 task 一次发布多个 module

以 ExoPlayer 为例，在根目录 root build.gradle 配置：

```groovy
subprojects {
    ['MavenRepository', 'MavenLocal'].each { repository ->
        tasks.register("publishExoPlayerTo$repository") { task ->
            if (task.project.parent.name.equalsIgnoreCase("ExoPlayer")
                    && task.project.hasProperty("publishReleasePublicationTo$repository")) {
                // project 父目录是 ExoPlayer 并且 apply 了 maven-publish 插件
                task.dependsOn("publishReleasePublicationTo$repository")
            }
        }
    }
}
```