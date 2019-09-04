---
layout: post
title:  Gradle 启动应用
date:   2019-09-04 12:30:00 +0800
categories: Gradle
tags: Gradle
published: true
---

* content
{:toc}

Gradle 执行 install 后无法自动启动应用，两年前写了个 gradle 脚本解决该问题。今天在编译安装的时候出现了告警：

```log
WARNING: API 'variantOutput.getInstall()' is obsolete and has been replaced with 'variantOutput.getInstallProvider()'.
It will be removed at the end of 2019.
For more information, see https://d.android.com/r/tools/task-configuration-avoidance.
To determine what is calling variantOutput.getInstall(), use -Pandroid.debug.obsoleteApi=true on the command line to display more information.
```

告警大致意思是 `variantOutput.getInstall()` 已过期，将会在 2019 年底移除，需要使用 `variantOutput.getInstallProvider()` 代替。

该问题应该是在 Android Studio v3.3.0（or later）才发生的，具体参考 [Android studio 源码](
https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/api/InstallableVariant.java){:target="_blank"}：

```java
    /**
     * Returns the install task for the variant.
     *
     * @deprecated Use {@link #getInstallProvider()}
     */
    @Nullable
    @Deprecated
    DefaultTask getInstall();
    /**
     * Returns the {@link TaskProvider} for the install task for the variant.
     *
     * <p>Prefer this to {@link #getInstall()} as it triggers eager configuration of the task.
     */
    @Nullable
    TaskProvider<Task> getInstallProvider();
```

<br>

引起告警的启动脚本：

```groovy
android.applicationVariants.all { variant ->
    if (variant.install != null) {
        variant.install.doLast {

        ···

        }
    }
}
```

通过遍历 applicationVariants 找到 InstallableVariant，调用其 getInstall() 方法，该方法返回 DefaultTask，再调用 DefaultTask 的 doLast() 方法，在 doLast() 方法里启动应用。

现在必须将 getInstall() 方法替换为 getInstallProvider()，该方法返回的是一个 [TaskProvider](https://github.com/gradle/gradle/blob/b189979845c591d8c4a0032527383df0f6d679b2/subprojects/core-api/src/main/java/org/gradle/api/tasks/TaskProvider.java){:target="_blank"}：

```java
/**
 * Providers a task of the given type.
 *
 * @param <T> Task type
 * @since 4.8
 */
public interface TaskProvider<T extends Task> extends NamedDomainObjectProvider<T> {
    /**
     * Configures the task with the given action. Actions are run in the order added.
     *
     * @param action A {@link Action} that can configure the task when required.
     * @since 4.8
     */
    @Override
    void configure(Action<? super T> action);

    /**
     * The task name referenced by this provider.
     * <p>
     * Must be constant for the life of the object.
     *
     * @return The task name. Never null.
     * @since 4.9
     */
    @Override
    String getName();
}
```

从源码可以得知 TaskProvider 是 gradle v4.8 新加入，适合调用的只有 configure() 方法，通过该方法传入 Action。  
修复后的启动脚本：

```groovy
android.applicationVariants.all { variant ->
    if (variant.installProvider != null) {
        variant.installProvider.configure {
            doLast {

            ···

            }
        }
    }
}
```
