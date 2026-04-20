---
layout: post
title: kotlin 取余
date: 2026-04-20 14:30:00 +0800
categories: kotlin
tags: mod
published: true
---

* content
{:toc}

众所周知，kotlin 里取余有两种方式 `%` 和 `mod`。一直以来我以为两者是等价的，甚至想当然认为 `mod` 是 `%` 的操作符重载函数，直到最近刷 LeetCode 碰上一个 bug，深入了解之下才发现并不一样。<br>
不借助任何搜索看看你能否答对如下测试：

```kotlin
fun main() {
    println("5 % 4 = ${5 % 4}")
    println("5.mod(4) = ${5.mod(4)}")

    println("-5 % 4 = ${-5 % 4}")
    println("-5.mod(4) = ${(-5).mod(4)}")

    println("5 % -4 = ${5 % -4}")
    println("5.mod(-4) = ${5.mod(-4)}")

    println("-5 % -4 = ${-5 % -4}")
    println("-5.mod(-4) = ${(-5).mod(-4)}")
}
```

结果如下：

```txt
5 % 4 = 1
5.mod(4) = 1

-5 % 4 = -1
-5.mod(4) = 3

5 % -4 = 1
5.mod(-4) = -3

-5 % -4 = -1
-5.mod(-4) = -1
```

当被除数和除数符号位一致时，`%` 和 `mod` 结果相同；然而符号位不一致时，结果大相庭径。

## %

* % 使用的是向 0 截断除法（truncate toward zero），即 `a % b = a - (a / b) * b`
* % 取余的结果符号位跟 dividend（被除数） 相同

BTW，`%` 对应的操作符重载函数是 `rem`：
```kotlin
class Foo(private val value: Int) {
    operator fun rem(other: Foo): Foo {
        return Foo(value % other.value)
    }
}
```

## mod

源码在 [FloorDivMod.kt](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/FloorDivMod.kt){:target="_blank"} 文件，以整数为例：

```kotlin
/**
 * Calculates the remainder of flooring division of this value (dividend) by the other value (divisor).
 *
 * The result is either zero or has the same sign as the _divisor_ and has the absolute value less than the absolute value of the divisor.
 */
@SinceKotlin("1.5")
@kotlin.internal.InlineOnly
@kotlin.internal.IntrinsicConstEvaluation
public inline fun Int.mod(other: Int): Int {
    val r = this % other
    return r + (other and (((r xor other) and (r or -r)) shr 31))
}
```

`return r + (other and (((r xor other) and (r or -r)) shr 31))` 意思是如果 r 和 other 符号不同，就加上 other。等价于：
```kotlin
public inline fun Int.mod(other: Int): Int {
    val r = this % other
    return if ((r >= 0 && other > 0) || (r <= 0 && other < 0)) r else r + other
}
```

* 由文件名不难看出 mod 是向下取整（floor division），也即 `a.mod(b) = a - floor(a.toDouble() / b).toInt() * b`
* mod 取余的结果符号位跟 divisor（除数） 相同（余数 0 除外）

## 总结

`%` 和 `mod` 区别省流版：
1. `%` 的实现使用的是向 0 截断除法；`mod` 使用的是向下取整除法
2. `%` 取余结果的符号位与被除数相同；`mod` 结果符号位与除数相同（余数 0 除外）
