---
layout: post
title: 如何获取 Flutter ListView first/lastVisibleItemIndex
date: 2021-07-20 22:10:00 +0800
categories: Flutter
tags: ListView
published: true
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

假设 Listview 构造方式如上，我们可以通过自定义 SliverChildBuilderDelegate 来获取其 first/lastVisibleItemIndex：

* 首先，新建文件 custom_silver_child_builder_delegate.dart，重写方法 `didFinishLayout(firstIndex, lastIndex)`，这里的 firstIndex/lastIndex 便是我们需要的第一条/最后一条 item index。

    ```dart
    import 'package:flutter/material.dart';

    int _kDefaultSemanticIndexCallback(Widget _, int localIndex) => localIndex;

    class CustomSliverChildBuilderDelegate extends SliverChildBuilderDelegate {
      const CustomSliverChildBuilderDelegate(
        builder, {
        findChildIndexCallback,
        childCount,
        addAutomaticKeepAlives = true,
        addRepaintBoundaries = true,
        addSemanticIndexes = true,
        semanticIndexCallback = _kDefaultSemanticIndexCallback,
        semanticIndexOffset = 0,
      }) : super(
              builder,
              findChildIndexCallback: findChildIndexCallback,
              childCount: childCount,
              addAutomaticKeepAlives: addAutomaticKeepAlives,
              addRepaintBoundaries: addRepaintBoundaries,
              addSemanticIndexes: addSemanticIndexes,
              semanticIndexCallback: semanticIndexCallback,
              semanticIndexOffset: semanticIndexOffset,
            );

      @override
      void didFinishLayout(int firstIndex, int lastIndex) {
        print("didFinishLayout: firstIndex=$firstIndex lastIndex=$lastIndex");
        super.didFinishLayout(firstIndex, lastIndex);
      }
    }
    ```

* 然后，替换原来 ListView 的构造方式。这里有两点需要注意：一是原来的 itemCount 变为了 SliverChildBuilderDelegate 的 childCount；二是 cacheExtent 需要设置为 0（默认不为0），cacheExtent 决定了 ListView 是否预先构造 item 布局，可以参考 [Flutter ListView 用法详解](https://zhuanlan.zhihu.com/p/62324067){:target="_blank"}，只有在不预先构造 item 的时候 firstIndex/lastIndex 才等于第一个/最后一个可见的 itemIndex。禁了预构造布局，对 ListView 的加载流畅性会有一定影响。

    ```dart
    ListView.custom(
          cacheExtent: 0,
          childrenDelegate: CustomSliverChildBuilderDelegate(
            (context, index) {
              return _generateItem(index);
            },
            childCount: 40,
          );
    ```

每次新的 item 构造布局结束时便会回调 `CustomSliverChildBuilderDelegate` 的 `didFinishLayout(firstIndex, lastIndex)`，我们可以在该方法里将 firstIndex/lastIndex 传出来使用。

最后，该[文章](https://blog.csdn.net/u014803467/article/details/103750018){:target="_blank"}提到：如果保存某一 item 状态会导致 `didFinishLayout(firstIndex, lastIndex)` 方法的 firstIndex/lastIndex 不符合预期（想想也是可以理解的，这两本身并不是 firstVisibleItemIndex/lastVisibleItemIndex，我们只是试图让其等于 firstVisibleItemIndex/lastVisibleItemIndex），可以使用 `estimateMaxScrollOffset(firstIndex, lastIndex, leadingScrollOffset, trailingScrollOffset)` 方法代替。
