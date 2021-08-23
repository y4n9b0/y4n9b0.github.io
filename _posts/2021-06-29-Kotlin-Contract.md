---
layout: post
title: Kotlin Contract
date: 2021-06-29 10:00:00 +0800
categories: Kotlin
tags: contract
published: false
---

* content
{:toc}

优雅高效的Kotlin构造者模式：

```kotlin
fun main() {
    val foo = Foo.build {
        bar = 1024
    }
    println("foo.bar=${foo.bar}")
}

class Foo private constructor(builder: Builder) {
    internal var bar: Int

    init {
        bar = builder.bar
    }

    companion object {
        fun build(block: Builder.() -> Unit) = Builder(block)()
    }

    class Builder private constructor() {
        var bar: Int = 0

        constructor(block: Builder.() -> Unit) : this() {
            block()
        }

        operator fun invoke(): Foo = Foo(this)
    }
}
```

* [play.kotlinlang.org](https://play.kotlinlang.org/){:target="_blank"}

<!-- https://pspdfkit.com/blog/2018/kotlin-contracts/#the-main-problem -->
<!-- https://juejin.cn/post/6844903730219859982 -->
<!-- https://www.jianshu.com/p/a35f99adf365 -->