---
layout: post
title: 短路逻辑或/与
date: 2024-05-11 11:30:00 +0800
categories: bitwise
tags: infix
published: true
---

* content
{:toc}

今天写了如下一段 kotlin 代码：

```kotlin
fun foo(uid: Int, list: List<Int>): Boolean  = 
    list.isEmpty() or ((list.size == 1) and (list[0] == uid))
```

这段代码意图传入一个用户 id 列表和一个指定用户 id，如果列表为空、或者列表只包含指定的用户 id，返回 true，否则返回 false。

很简单的一段代码，部分场景崩了。原因是列表为空时，中缀操作符 or 后面的表达式会继续执行，导致数组越界异常。

想当然地以为 or 和 and 是短路逻辑，看了下源码才发现这两并不是短路的。反编译其对应的 java 字节码，底层实现使用的是非短路逻辑运算符 `| &`：

```java
public final boolean foo(int uid, @NotNull List list) {
    Intrinsics.checkNotNullParameter(list, "list");
    return list.isEmpty() | list.size() == 1 & ((Number)list.get(0)).intValue() == uid;
}
```

解决方式很简单，将中缀操作符 or and 换成短路逻辑运算符 `|| &&` 即可：

```kotlin
fun foo(uid: Int, list: List<Int>): Boolean =
    list.isEmpty() || (list.size == 1 && list[0] == uid)
```