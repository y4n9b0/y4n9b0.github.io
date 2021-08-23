---
layout: post
title: Kotlin Hidden Costs
date: 2021-07-02 10:00:00 +0800
categories: Kotlin
tags: Kotlin
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

* [try.kotl.in](https://try.kotl.in/){:target="_blank"}
* [play.kotlinlang.org](https://play.kotlinlang.org/){:target="_blank"}

<!-- https://sites.google.com/a/athaydes.com/renato-athaydes/posts/kotlinshiddencosts-benchmarks -->