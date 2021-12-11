---
layout: post
title: Android Studio Plugin
date: 2021-12-11 23:30:00 +0800
categories: IDE
tags: plugin
published: true
---

* content
{:toc}

## 字节码插件

最近在看字节码相关，为了方便调试，需要用到相关的 IDE 插件，在 Android Studio -> Preferences -> Plugins -> Marketplace 里搜索 bytecode 得到大致如下几类插件：

* ASM Bytecode Viewer<br>
    推荐，但是对于使用 Maven 和 Gradle 的工程，容易报错 couldn't generate bytecode view, no .class file found，[解决办法](https://plugins.jetbrains.com/plugin/10302-asm-bytecode-viewer/reviews#review=41075-41076){:target="_blank"}就是不要在 .java 文件上进行 bytecode view，而是对 build 后生成的 .class 文件 bytecode view。

    Android Studio 生成的 .class 所在目录为 `build -> intermediates -> javac -> ${flavor} -> classes`。

* jclasslib Bytecode Viewer<br>
    不推荐，[无法运行](https://plugins.jetbrains.com/plugin/9248-jclasslib-bytecode-viewer/reviews#review=44055){:target="_blank"}在 Android Studio 4.x 及以上。虽说 Android Studio 是基于 IntelliJ IDEA，理论上应该是兼容 IntelliJ IDEA 的插件，但实际却是无法运行，Android Studio 里 根本没有 View -> Show Bytecode With jclasslib 选项。

* ASM Bytecode Outline<br>
    强烈不推荐，很多年没更新，应该是不再维护。安装后不成功，Android Studio 启动后无法正常加载该插件，会一直 log 告警，而且无法直接卸载(plugin 列表不显示该插件)。

* ASM Bytecode Viewer Support Kotlin<br>
    不推荐，Android Studio 本身就支持查看 Kotlin 的字节码：`Tools -> Kotlin -> Show Kotlin Bytecode`。<br>
    另外，该插件和 ASM Bytecode Viewer 同时安装会导致 Android Studio 无法正常启动。

## 卸载 Android Studio 插件

正常情况下，我们可以通过 `Android Studio -> Preferences -> Plugins -> Installed 找到插件列表，右键单击对应的插件，选择 Uninstall 即可。但如果 Android Studio 无法启动，或者启动后无法加载也无法卸载，这个时候就需要我们手动去删除对应的 plugin 安装文件了：

* /Applications/Android Studio.app/Contents/plugins<br>
    Android Studio 自身集成 plugin 所在的文件位置，不要动。

* ${HOME}/Library/Application Support/Google/AndroidStudio2020.3/plugins<br>
    用户为 Android Studio 后期安装的 plugin 所在文件位置，需要注意的是这里的 AndroidStudio2020.3 文件夹对应的是 Android Studio Arctic Fox| 2020.3.1 版本，如果你用的是其他版本则寻找对应的文件夹就好了。<br>
    如果删除整个 plugins 文件夹，则所有安装的 plugin 都会消失不见。
    
    `${HOME}/Library/Application Support` 就是 [Application's data folder in Mac](https://stackoverflow.com/questions/9495503/applications-data-folder-in-mac){:target="_blank"}。
