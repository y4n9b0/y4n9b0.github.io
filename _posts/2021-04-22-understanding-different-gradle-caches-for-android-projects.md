---
layout: post
title:  深入理解 Android 工程的各种 Gradle 缓存
date:   2021-04-22 11:00:00 +0800
categories: android
tags: gradle
published: false
---

* content
{:toc}

## 一、树状 dependencies

```bash
# clear incremental build cache
$ ./gradlew clean

# clear Gradle build cache
$ rm -rf $GRADLE_HOME/caches/build-cache-*

# clear AGP cache (<user-home>/.android/build-cache/)
$ ./gradlew cleanBuildCache

# clear 3rd party dependency cache
$ rm -rf $GRADLE_HOME/caches/modules-2
$ rm -rf $GRADLE_HOME/caches/transform-2

# stop gradle daemon
$ ./gradlew --stop

# run a build
$ ./gradlew --scan app:assembleDebug

BUILD SUCCESSFUL in 11m 49s
279 actionable tasks: 192 executed, 82 from cache, 5 up-to-date

# re-run a build without changing anything
$ ./gradlew --scan app:assembleDebug

BUILD SUCCESSFUL in 9s
279 actionable tasks: 279 up-to-date
```

<!-- https://jasonatwood.io/archives/1966 -->
<!-- https://jasonatwood.io/archives/1995 -->
