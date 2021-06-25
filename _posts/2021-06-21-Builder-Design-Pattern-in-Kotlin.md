---
layout: post
title: Builder Design Pattern in Kotlin
date:   2021-06-21 20:00:00 +0800
categories: Kotlin
tags: design-pattern
published: true
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
    val bar: Int

    init {
        bar = builder.bar
    }

    companion object {
        fun build(block: Builder.() -> Unit) = Builder(block)()
    }

    class Builder internal constructor(block: Builder.() -> Unit) {
        var bar: Int = 0

        init {
            block()
        }

        internal operator fun invoke(): Foo = Foo(this)
    }
}
```

* [try.kotl.in](https://try.kotl.in/){:target="_blank"}
* [play.kotlinlang.org](https://play.kotlinlang.org/){:target="_blank"}

<!-- https://www.jianshu.com/p/f5f0d38e3e44 -->