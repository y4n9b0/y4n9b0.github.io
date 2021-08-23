---
layout: post
title: Y Combinator
date: 2021-07-05 10:00:00 +0800
categories: programming
tags: paradigm
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

<!-- https://www.zhihu.com/question/20115649 -->
<!-- https://github.com/EasyKotlin/kotlin-demo/blob/master/src/main/kotlin/com/light/sword/ycombinator/Y.kt -->