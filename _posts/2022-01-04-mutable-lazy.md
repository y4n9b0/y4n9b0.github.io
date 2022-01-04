---
layout: post
title:  Mutable Lazy
date:   2022-01-04 23:30:00 +0800
categories: Kotlin
tags: lazy-evaluation
published: true
---

* content
{:toc}

## 一、前言

接触过 kotlin 一段时间的朋友都应该知道，kotlin 有一个 lazy 惰性求值（lazy evaluation 是函数式编程比较基础的东西）。对于很多新手朋友来说，常常会跳入 lazy 缓存的这个坑，
简单地说就是 Lazy 内部实现维持着一个 cache 变量，一旦惰性求值 evaluate 过后，便将值保存在 cache 里，以后再来取的时候直接从 cache 里拿。

如果惰性求值 evaluation 里有依赖外部的其他变量，那么 lazy 得到的值永远不会跟随变量改变。
很多时候我们希望延迟计算的同时 值也能动态跟随变量改变（典型的既要又要，没办法 我们就是这么贪得无厌:smile: ），那么这个时候就只能选择 getter 了(getter 的本质也是方法)。
但是 getter 的一个缺陷在于每次都会重新计算，如果变量改变得不是很频繁，那么会白白浪费很多无用的 evaluation。如果 evaluation 比较简单，那也罢了；
如果 evaluation 很耗资源，可不容小觑。

至此，我们已经知道 lazy 和 getter 其实对应着两个极端，要么不重新求值 要么就一直重新求值。有没有一种方式在实现惰性且动态求值的同时，又不用每次都重新计算呢？
这就是我们今天要聊的 MutableLazy，只有在我们需要它重新 evaluate 的时候才重新计算（当然，我们得主动告诉 MutableLazy 该重新计算了）。

走起，放码！

## 二、v1

```kotlin
class MutableLazy<T>(private val initializer: () -> T) : Lazy<T> {

    private var cached: T? = null

    override val value: T
        get() {
            if (cached == null) cached = initializer()
            @Suppress("UNCHECKED_CAST")
            return cached as T
        }

    fun eval(block: (() -> T)? = null) {
        cached = if (block != null) block() else initializer()
    }

    override fun isInitialized(): Boolean = cached != null

    override fun toString(): String =
        if (isInitialized()) value.toString() else "MutableLazy value not initialized yet."
}

fun <T> mutableLazy(initializer: () -> T): MutableLazy<T> = MutableLazy(initializer)
```

使用方式如下：

```kotlin
fun main() {
    var foo = 1
    val bar = mutableLazy {
        2 * foo
    }

    println("bar=$bar")         // bar=MutableLazy value not initialized yet.
    println("bar=${bar.value}") // bar=2
    println("bar=$bar")         // bar=2
    foo++
    bar.eval()
    println("bar=$bar")         // bar=4
    bar.eval()
    println("bar=$bar")         // bar=4
    bar.eval { 9 }
    println("bar=$bar")         // bar=9
    println("bar=$bar")         // bar=9
    bar.eval()
    println("bar=$bar")         // bar=4
    println("bar=$bar")         // bar=4
}
```

这个版本大概是所有实现里边最精简的一个了。

## 三、v2

v1 版本的实现有一个小缺陷，就是可以通过直接构造 MutableLazy() 而绕过我们定义的扩展方法 mutableLazy()，为了保持对外 API 的简洁性，也为了与 Lazy 保持一致，我们决定使用接口的方式来改善这一缺陷（你没有办法直接 new 一个接口对吧？）：

```kotlin
interface MutableLazy<T> {

    val value: T

    fun isInitialized(): Boolean

    fun eval(block: (() -> T)? = null)
}

internal class MutableLazyImpl<T>(private val initializer: () -> T) : MutableLazy<T> {

    private var cached: T? = null

    override val value: T
        get() {
            if (cached == null) cached = initializer()
            @Suppress("UNCHECKED_CAST")
            return cached as T
        }

    override fun eval(block: (() -> T)?) {
        cached = if (block != null) block() else initializer()
    }

    override fun isInitialized(): Boolean = cached != null

    override fun toString(): String =
        if (isInitialized()) value.toString() else "MutableLazy value not initialized yet."
}

operator fun <T> MutableLazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value

fun <T> mutableLazy(initializer: () -> T): MutableLazy<T> = MutableLazyImpl(initializer)
```

通过公开对外接口、隐藏具体实现类的方式（参考系统的 Lazy 设计），使用上跟 v1 的示例保持一致(结果也一样)。

## 四、v3

v3 这一版本严格来说算不上优化，更应该是一个选择问题。

对于 v1、v2 版本，在 eval 方法中，如果传入的 Function0 block 为 null，我们会选择最开始的 initializer 去求值。
在这一版本中，如果传入的 Function0 block 为 null，我们选择最近一次设置的非空 block 去求值（当然，如果一直没有设置过非空 block，自然是使用 initializer）。

```kotlin
interface MutableLazy<T> {

    val value: T

    fun isInitialized(): Boolean

    fun eval(block: (() -> T)? = null)
}

internal class MutableLazyImpl<T>(private val initializer: () -> T) : MutableLazy<T> {

    private var cached: T? = null
    private var evaluation: () -> T = initializer

    override val value: T
        get() {
            if (cached == null) cached = evaluation()
            @Suppress("UNCHECKED_CAST")
            return cached as T
        }

    override fun eval(block: (() -> T)?) {
        if (block != null) evaluation = block
        cached = evaluation()
    }

    override fun isInitialized(): Boolean = cached != null

    override fun toString(): String =
        if (isInitialized()) value.toString() else "MutableLazy value not initialized yet."
}

operator fun <T> MutableLazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value

fun <T> mutableLazy(initializer: () -> T): MutableLazy<T> = MutableLazyImpl(initializer)
```

```kotlin
fun main() {
    var foo = 1
    val bar = mutableLazy {
        2 * foo
    }

    println("bar=$bar")         // bar=MutableLazy value not initialized yet.
    println("bar=${bar.value}") // bar=2
    println("bar=$bar")         // bar=2
    foo++
    bar.eval()
    println("bar=$bar")         // bar=4
    bar.eval()
    println("bar=$bar")         // bar=4
    bar.eval { 9 }
    println("bar=$bar")         // bar=9
    println("bar=$bar")         // bar=9
    bar.eval()
    // 与 v2 不同，这里使用的是上一次有效的 evaluation 求值
    println("bar=$bar")         // bar=9
    println("bar=$bar")         // bar=9
}
```

相较而言，个人更喜欢 v2 版本，不容易产生误解。v3 的应用场景，v2 通过显式传递 Function0 入参也可以做到。