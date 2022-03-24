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

## 优雅高效的Kotlin构造者模式

```kotlin
fun main() {
    val foo = Foo.build {
        bar = 1024
    }
    println("foo.bar=${foo.bar}")
}

class Foo private constructor(builder: Builder) {
    val bar: Int = builder.bar

    companion object {
        fun build(block: Builder.() -> Unit = {}) = Foo(Builder(block))
    }

    class Builder internal constructor(block: Builder.() -> Unit) {
        var bar: Int = 0

        init {
            block()
        }
    }
}
```

## 更极端的方式

```kotlin
fun main() {
    val foo = Foo {
        bar = 1024
    }
    println("foo.bar=${foo.bar}")
}

class Foo private constructor(builder: Builder) {
    val bar: Int = builder.bar

    companion object {
        operator fun invoke(block: Builder.() -> Unit = {}) = Foo(Builder(block))
    }

    class Builder internal constructor(block: Builder.() -> Unit) {
        var bar: Int = 0

        init {
            block()
        }
    }
}
```

该方式通过重载invoke运算符，使得调用代码更加精简，但是少了build字段导致可读性降低（很难一眼看出这是构造者模式）。
有利有弊吧，个人项目我更喜欢后一种，团体项目推荐使用前者。

Java的调用方式，以前一种为例(后一种只需要将build替换为invoke)：

```java
    Foo foo = Foo.Companion.build(new Function1<Foo.Builder, Unit>() {
        @Override
        public Unit invoke(Foo.Builder builder) {
            builder.setBar(1024);
            return Unit.INSTANCE;   // 也可以直接 return null
        }
    });
```

如果是Java 8及以上，还可以使用lambda的方式调用：

```java
    Foo foo = Foo.Companion.build(builder -> {
        builder.setBar(1024);
        return null;
    });
```

## The hack way

data class + val property + default value：

```kotlin
fun main() {
    val foo = Foo(hehe = "heihei")
    print("foo=$foo")
}

data class Foo(
    val haha: String = "haha",
    val hoho: String = "hoho",
    val hehe: String = "hehe"
)
```

一直觉得 data class 是 kotlin 挺废物的一个设计，直到最近突然悟到可以用来完美替代构造者模式。
该方式唯一的缺点就是在 java 里调用，由于无法缺省默认值导致设置属性非常麻烦。

* [try.kotl.in](https://try.kotl.in/){:target="_blank"}
* [play.kotlinlang.org](https://play.kotlinlang.org/){:target="_blank"}

<!-- https://www.jianshu.com/p/f5f0d38e3e44 -->