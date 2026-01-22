---
layout: post
title: 一行代码没改 解了一个bug
date: 2026-01-22 10:00:00 +0800
categories: debug
tags: Android
published: true
---

* content
{:toc}

网吧创新部门的语音聊天项目终止后接着搞了游戏租号，为了降本增笑，使用 Flutter 开发，项目初期恰逢小家伙出生，参与得不多，搭了搭首页框架、画了画 UI，托兄弟萌的努力，回来之后初版已经完成得差不多了。再然后，进一步降本增笑，老杨被分流去了一个坑逼项目，租号项目的客户端就只剩了 Android、iOS 各一名。

随着降本增笑的逐级加深，上周租号项目仅剩的一名 Android 也被优化了，好巧不巧，刚好该项目升级启动游戏的第三方 SDK，该 SDK 双开启动游戏，使用了大量的 hack 技术，这次升级主要是适配 Android 15，在这之前 Android 15 及以上无法使用。

iOS 兄弟替换 SDK 后发现 Android 15 上游戏启动不起来，日志给到 SDK 方看不出任何问题。被拉去协助，最后的结果有点儿意思。

## 分析日志

SDK 方给了一个 demo，安装发现同一台 Android 15 机子是可以成功启动的，抓取两个 app 的日志进行比对。

游戏租号 app 异常启动游戏日志：
![app log]({{ '/styles/images/bug/log-app.jpeg' | prepend: site.baseurl  }}){:width="690" height="387"}  

SDK demo 正常启动游戏日志：
![demo log]({{ '/styles/images/bug/log-demo.jpeg' | prepend: site.baseurl  }}){:width="690" height="387"}  

日志过滤了部分无关信息，可以看到有用的信息并不多，启动游戏后两个 app 都 fork 了进程，SDK 成功 startup，demo app 很快就输出了一条 SDK 版本号的 test 日志（1.0.1.0），与 app 里配置的版本号一致。而游戏租号 app crash 之前并没有输出版本号，崩溃在 [interpreter_common.cc](https://android.googlesource.com/platform/art/+/master/runtime/interpreter/interpreter_common.cc#201){:target="_blank"}：

```c++
void UnexpectedOpcode(const Instruction* inst, const ShadowFrame& shadow_frame) {
  LOG(FATAL) << "Unexpected instruction: "
             << inst->DumpString(shadow_frame.GetMethod()->GetDexFile());
  UNREACHABLE();
}
```

Android Runtime(ART) 解释执行代码时遇上了非法指令 `unused-f8`。至此，感觉像是 SDK 双开 hack 技术修改 dex 文件出了问题，但是无法解释为什么 demo 能够成功运行，而 SDK 方也正是借着这一由头让我们提供更多详细日志。wtf！

## 查看代码

对比了双方对 SDK 的初始化和调用，区别不大，主要的区别是 app 里 flutter 代码使用 channel 调用 Android 原生代码启动游戏，但日志显示已经成功执行了启动游戏的代码调用，线程也正确。

## 查看 Android 15 官方变动

[行为变更](https://developer.android.com/about/versions/15/behavior-changes-all?hl=zh-cn){:target="_blank"}

[适配](https://developer.android.com/about/versions/15/migration?hl=zh-cn){:target="_blank"}

## 搜索游戏双开原理

排查疑难杂症需要尽可能多的了解相关信息，寻找蛛丝马迹，SDK 代码经过混淆，几乎没有任何可读性，只能搜索相关背景知识，过程略过不表。

## minimal working example (MWE)

游戏租号 app 代码已经有 20 多万行，为了尽可能减少无关代码影响，决定仿照 app 做一个无法启动游戏的 minimal working example 测试 app，使用 Android 原生代码实现。

## MWE 与 demo 对齐

测试用的 MWE app 如预期一样无法成功启动，接下来就只剩一个苦办法了，把 MWE 上的配置、依赖、代码一项一项与 demo app 靠齐，找出到底是哪出了问题。
这是个体力活儿，比二分查找难多了，二分查找是你知道有一个特定结果（或者没有结果），当前的情况可能有一项或者多项同时影响。

1. gradle 和 AGP 版本对齐
2. build.gradle 相关配置对齐
3. lib 依赖对齐
4. 代码调用方式对齐

幸运的是，在执行到上面第二步时，MWE 就成功启动了游戏，经过仔细比对，发现影响结果的配置是如下这一行：

```groovy
    buildTypes {
        debug {
            debuggable false
            ···
        }
        release {
            ···
        }
    }
```

SDK demo app 在 debug buildType 里将 debuggable 设置为了 false。

debuggable 会影响： 
* Zygote fork 参数
* ART 是否开启 debug 模式
* 是否允许 ptrace

刚好对应上了 ART 解释执行遇上了非法指令 `unused-f8`。回头去看上面的对比日志，其实早已给出了指示，游戏租号 app 的日志里有一条 `adbd jdwp connection from 31221`，
而 demo app 日志里 `Not starting debugger since process cannot load the jdwp agent`。
盲猜是 debug 模式下 ART 对 method 插桩和 SDK 双开 hack 修改 dex 文件冲突，限于时间、精力和能力，无法继续深入。

所以，最终的结果是什么都不需要修改，如果要测试 Android 15 上的游戏启动功能，编译 release 包即可。<br>
对了，为什么不将 flutter 工程 Android build.gradle 脚本里的 debug buildType 设置 debuggable 为 false 呢？
因为 flutter 工程支持 hot reload，要求 debuggable 必须为 true，设置 false 之后无法运行，也无法成功编译出 apk。

## flutter 编译、安装

flutter 编译 apk 命令：

```kotlin
flutter build apk --flavor ${flavor} --release
```

flutter 安装 apk 命令：

```kotlin
flutter install apk --flavor ${flavor} --release
```

两点注意事项：
* flutter 安装命令并不会执行编译命令，不像 Android 原生工程执行 `gradle install` 会自动 dependsOn `gradle assemble`。需要先执行 flutter 编译 apk 命令，再执行安装命令。
* 安装命令的 flavor 和 buildType 选项必须和编译命令保持一致，否则会找不到对应选项的 apk。实际使用中老杨习惯将命令组合到一起：
    ```kotlin
    flutter clean && \
    flutter pub get && \
    flutter build apk --flavor ${flavor} --release && \
    flutter install apk --flavor ${flavor} --release
    ```