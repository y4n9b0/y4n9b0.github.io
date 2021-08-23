---
layout: post
title: dagger hilt
date: 2021-07-29 22:00:00 +0800
categories: DI
tags: dagger
published: false
---

* content
{:toc}

在 Android 原生 ListView 中获取当前第一条/最后一条可见 item 的 index 是一件比较轻松的事情，然而在 flutter 中却没有相应的 API 可供直接使用。
比如官方这个 [issue-19941](https://github.com/flutter/flutter/issues/19941){:target="_blank"}，截至目前已经 open 三年了，有生之年估计是看不到了。
然而，业务还是要继续，只好撸起袖子另谋他路。

```dart
ListView.builder(
            itemCount: 40,
            itemBuilder: (context, index) {
              return _generateItem(index);
            };
```


<!-- https://blog.csdn.net/guolin_blog/article/details/109787732
<!-- https://dagger.dev/hilt/ --> -->
<!-- https://developer.android.google.cn/training/dependency-injection?hl=zh_cn -->
<!-- https://juejin.cn/post/6844904158324064269?utm_source=gold_browser_extension -->
