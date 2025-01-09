---
layout: post
title: kotlin overload resolution ambiguity
date: 2025-01-09 17:30:00 +0800
categories: kotlin
tags: kotlin
published: true
---

* content
{:toc}

## 模凌两可的重载

```kotlin
fun main() {
    val list = listOf(-2, -1, 0, 1, 2)
    val negativeCount = list.sumOf { if (it < 0) 1 else 0 }
    println(negativeCount)
}
```

如上代码统计列表里负数的个数，使用了 built-in 函数 `sumOf`，运行时报错：

```txt
Type inference fails to infer type for sumOf call with integer literal: "Overload resolution ambiguity TypeVariable(T) -> Int / Long"
```

`sumOf` 扩展函数有多个重载，在推断表达式 `if (it < 0) 1 else 0` 类型时编译器不知道使用 Int 还是 Long，于是报错了。
而作为开发者来说，一眼 Int，很简单嘛，于是反人类直觉的 bug 就出现了。

## 整数字面量的类型

问题根源涉及到一个概念 kotlin specific 8.1.3 [The types for integer literals](https://kotlinlang.org/spec/expressions.html#the-types-for-integer-literals){:target="_blank"}

kotlin 任何十进制、十六进制、二进制字面量都可以添加 L 后缀（即 long literal mark），一个整数字面量添加了 L 后缀就拥有 kotlin.Long 类型。
没有添加 L 后缀的整数字面量有一个特殊的整数字面量类型（ILT - integer literal type），该类型取决于字面量本身有如下几种情况：

1. 如果整数字面量 value 大于 kotlin.Long 的最大值，那么它是一个非法的整数字面量，编译器检测到会报错。
2. 如果整数字面量 value 大于 kolint.Int 的最大值且小于等于 kotlin.Long 的最大值，那么它是 kotlin.Long 类型。
3. 否则，它的整数字面量类型包含了所有能够表示其 value 的 built-in 整数类型。

第 3 点可能有点绕，简单地说，kotlin 内建的整数类型有 kotlin.Byte, kotlin.Short, kotlin.Int, kotlin.Long，这些内建整数类型都有自己的取值范围（能够表示的整数最大范围），如果一个整数字面量在某个整数类型的表示范围内，那么它就拥有该类型。<br>
比如：0x01 同时拥有 ILT（kotlin.Byte,kotlin.Short,kotlin.Int,kotlin.Long）；而 70000 则只拥有 ILT（kotlin.Int,kotlin.Long），因为 70000 超出了 kotlin.Byte,kotlin.Short 的取值范围，此时使用这两个类型已经无法表示 70000。

其实以上三点都可以归为一点：一个整数字面量拥有的 ILT 类型为 kotlin 内建的整数类型(kotlin.Byte, kotlin.Short, kotlin.Int, kotlin.Long)中所有能够表示该字面量值的类型，如果一个类型都没有（超出了所有整数类型的表示范围），那么该整数字面量非法。

基于这个特性，一个没有添加 L 后缀的整数字面量是可能同时拥有多个内建整数类型的，所以重载 `sumOf` 的时候就出现了同时符合 kotlin.Int 和 kotlin.Long 类型的重载函数。

## 解决方案

这个问题在官方 issue [kotlin issue 46360](https://youtrack.jetbrains.com/issue/KT-46360){:target="_blank"} 里已经 open 三年多了，就连 Roman Elizarov 大佬也踩雷现身评论区。

鉴于底层 ILT 的影响范围很广，这种根基类的玩意儿不太可能动。一个比较可行的方案就是在多个重载函数都符合的时候设定一个优先级，但貌似 kotlin 官方在重载函数选择上也有自己的设定 [11.4.2 Algorithm of MSC selection](https://kotlinlang.org/spec/overload-resolution.html#algorithm-of-msc-selection){:target="_blank"} (MSC - most specific candidate)

即然官方不解决，咱们就另辟蹊径：

```kotlin
fun main() {
    val list = listOf(-2, -1, 0, 1, 2)
    val negativeCount = list.sumOf { (if (it < 0) 1 else 0) as Int }
    println(negativeCount)
}
```

注意，Android Studio 之类的 IDE 大概率会智能提示 as Int 为 useless cast 建议 remove，勿听！

如果担心不小心快捷键自动优化 remove，可以写成如下方式：

```kotlin
fun main() {
    val list = listOf(-2, -1, 0, 1, 2)
    val negativeCount = list.sumOf { (if (it < 0) 1 else 0).toInt() }
    println(negativeCount)
}
```

或者：

```kotlin
fun main() {
    val list = listOf(-2, -1, 0, 1, 2)
    val negativeCount = list.sumOf {
        val cnt: Int = if (it < 0) 1 else 0
        cnt
    }
    println(negativeCount)
}
```

又或者干脆不用 `sumOf`：

```kotlin
fun main() {
    val list = listOf(-2, -1, 0, 1, 2)
    val negativeCount = list.count { it < 0 }
    println(negativeCount)
}
```

## ILT

最后，聊一聊为什么 kotlin 要设计 ILT，个人猜测是方便使用，你可以把一个整数字面量赋值给任意一个它拥有的 ILT 整数类型变量。
kotlin 充斥着大量的语法糖，这门语言设计的很大一个原则就是方便使用，使得很多 javaer 初上手时感受到其便利并快速切换。<br>
但是，方便和自由是有代价的，设计 ILT 和写 `sumOf` 这类扩展函数的人大概不是同一波。

<!-- https://kotlinlang.org/spec/introduction.html -->
<!-- https://kotlinlang.org/spec/pdf/kotlin-spec.pdf -->
<!-- https://stackoverflow.com/questions/73062447/why-doesnt-kotlin-treat-numbers-as-int-by-default-inside-sumof-function-lambd -->
