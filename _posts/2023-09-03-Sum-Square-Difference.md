---
layout: post
title: 平方和与和平方之差
date: 2023-09-03 21:00:00 +0800
categories: math
tags: math
published: true
---

* content
{:toc}

## 问题

前十个自然数的平方和是：

$$1^2 + 2^2 + 3^2 + 4^2 + 5^2 + 6^2 + 7^2 + 8^2 + 9^2 + 10^2 = 385$$

前十个自然数的和平方是：

$$(1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10)^2 = 3025$$

因此，前十个自然数的平方和与和平方之差是 2640。<br>
求前一百个自然数的平方和与和平方之差。

## 递归

```kotlin
fun main() {
    assert(diff(10) == 2640L)
    assert(diff(20) == 25164150L)
}

fun diff(n: Int): Long {
    return if (n == 1) 0 else n.times(n).times(n - 1) + diff(n - 1)
}
```

## 遍历（数组缓存）

递归书写简单，也很容易理解，缺点就是容易 stackoverflow。

```kotlin
fun main() {
    assert(diff(10) == 2640L)
    assert(diff(20) == 25164150L)
}

fun diff(n: Int): Long {
    val dp = LongArray(n) { 0 }
    repeat(n) { i ->
        if (i != 0) {   // i = 0 时，dp[i] = 0，直接复用 dp 数组初始化的值
            dp[i] = dp[i - 1] + i.times(i + 1).times(i + 1)
        }
    }
    return dp[n - 1]
}
```

## 遍历（变量缓存）

使用数组缓存中间值一个缺点就是当 n 较大时，数组会占用较多连续内存空间。<br>
使用变量缓存可节省缓存空间，一般地，状态方程里当前状态与前面多少个状态有关就需要多少个变量。<br>
严格来说，需要缓存状态方程里当前状态与最远状态之间的所有状态，变量个数等于当前状态与最远状态之间的距离。

```kotlin
fun main() {
    assert(diff(10) == 2640L)
    assert(diff(20) == 25164150L)
}

fun diff(n: Int): Long {
    var pre = 0L
    var current = 0L
    repeat(n) { i ->
        if (i != 0) {
            current = pre + i.times(i + 1).times(i + 1)
            pre = current
        }
    }
    return current
}
```

## 遍历（无缓存）

如果当前状态仅与之前状态有关，即只需一个变量缓存时，可以借用 kotlin 集合相关的扩展函数来节省掉缓存变量，比如 reduce、fold

```kotlin
fun main() {
    assert(diff(10) == 2640L)
    assert(diff(20) == 25164150L)
}

fun diff(n: Int): Long {
    return (0L until n).reduce { total, i ->
        total + i.times(i + 1).times(i + 1)
    }
}
```

需要注意的是，reduce 默认把第一个元素作为 acc，实际遍历是从第二个元素开始的。而 fold 需要传入一个初始值作为 acc 的值，遍历是从第一个元素开始(理解不了就去看对应扩展函数的源码吧，一看就懂)：

```kotlin
fun main() {
    assert(diff(10) == 2640)
    assert(diff(20) == 25164150)
}

fun diff(n: Int): Long {
    return (1L until n).fold(0) { total, i ->
        total + i.times(i + 1).times(i + 1)
    }
}
```

<!-- https://pe-cn.github.io/6/ -->
