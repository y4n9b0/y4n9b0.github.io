---
layout: post
title: 如何将一个 flutter 模块接入鸿蒙项目
date: 2026-05-27 10:00:00 +0800
categories: Harmony
tags: flutter
published: false
---

* content
{:toc}

## Implement Rand10() Using Rand7()

1. 鸿蒙 Flutter 是基于 Google Flutter 魔改适配的一个版本

[鸿蒙 Flutter 仓库](https://gitcode.com/openharmony-tpc/flutter_flutter){:target="_blank"}

2. fvm 安装 鸿蒙 flutter 

[FVM 官网](https://fvm.app/){:target="_blank"}

[Custom Flutter Version (Forks)](https://fvm.app/documentation/advanced/custom-version){:target="_blank"}


```bash
fvm fork add openharmony https://gitcode.com/openharmony-tpc/flutter_flutter.git --verbose
fvm install openharmony/3.27.5-ohos-1.0.5
fvm use openharmony/3.27.5-ohos-1.0.5
```

3. 改造 flutter 工程适配鸿蒙

切换新分支，将工程根目录 .fvm 配置的 flutter 版本改为 openharmony/3.27.5-ohos-1.0.5，然后确认

```bash
fvm flutter --version
fvm flutter doctor -v
```

使用鸿蒙版的 Flutter 为项目创建 ohos 文件夹：
```bash
fvm flutter create --platforms ohos .
```

```bash
fvm flutter pub get
```

如果这时候出现了如下错误：
```txt
[!] No Hmos SDK found. Try setting the HOS_SDK_HOME environment variable.
```

在 ~/.zshrc 文件中添加(替换为你的SDK路径)
```txt
export HOS_SDK_HOME=~/Library/OpenHarmony/Sdk
```

若已配置环境变量但仍报错，说明Flutter可能缓存了错误的SDK路径，需手动重置
```bash
flutter config --ohos-sdk=""
```

4. 对项目进行打包运行

这时候就需要使用鸿蒙的开发 IDE 了。使用 DevEco-studio 打开 Flutter 项目刚刚创建的 ohos 文件夹

5. ohos 单跑
```
> hvigor ERROR: Could not resolve "../plugins/GeneratedPluginRegistrant" from "entry/src/main/ets/entryability/EntryAbility.ets"
1 ERROR: 10505001 ArkTS Compiler Error
```
https://developer.huawei.com/consumer/cn/forum/topic/0203352007280380568?fid=23

https://mp.weixin.qq.com/s/ItFvPDFs_7eD8OE8Pa2N1g

[Flutter 鸿蒙化迎来"大搬家"](https://mp.weixin.qq.com/s/dXof9mjd2ZK9YVgA46HQDg)