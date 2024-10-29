---
layout: post
title: AS Flutter 工程无法识别 adb 设备
date: 2024-10-29 10:30:00 +0800
categories: flutter
tags: flutter
published: true
---

* content
{:toc}

用 Android Studio 打开 Flutter 工程的时候 IDE 无法识别 adb 设备，一同打开的 Android native 工程却可以识别到 adb 设备。

在终端使用命令 `flutter devices` 发现手机设备已连接：

```bash
$ flutter devices

Found 3 connected devices:
  22101320C (mobile) • 9c232ca6 • android-arm64  • Android 13 (API 33)
  Linux (desktop)    • linux    • linux-x64      • Ubuntu 24.04.1 LTS
  6.8.0-47-generic
  Chrome (web)       • chrome   • web-javascript • Google Chrome 129.0.6668.89

Run "flutter emulators" to list and start any available device emulators.

If you expected another device to be detected, please run "flutter doctor" to
diagnose potential issues. You may also try increasing the time to wait for
connected devices with the "--device-timeout" flag. Visit
https://flutter.dev/setup/ for troubleshooting tips.
```

使用 `flutter run` 也可以把项目正常运行在连接的手机上。<br>
所以猜测应该是某项 flutter 配置问题，搜索发现是没有配置 flutter 的 android sdk：

```bash
$ flutter config --android-sdk <local android sdk path>
```

<!-- https://juejin.cn/post/6844904072198225933 -->