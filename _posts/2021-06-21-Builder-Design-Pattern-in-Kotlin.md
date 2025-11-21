---
layout: post
title: Builder Design Pattern in Kotlin
date: 2021-06-21 20:00:00 +0800
categories: Kotlin
tags: design-pattern
published: true
---

* content
{:toc}

## 优雅高效的Kotlin构造者模式

```kotlin
class Foo private constructor(
    val bar: Int,
    val baz: String,
    val qux: Boolean,
) {
    companion object {
        @JvmStatic
        fun build(block: Builder.() -> Unit = {}) = Builder().apply(block).build()
    }

    class Builder internal constructor() {
        var bar: Int = 0
        var baz: String = ""
        var qux: Boolean = false

        fun build() = Foo(bar = bar, baz = baz, qux = qux)
    }
}

fun main() {
    val foo = Foo.build {
        bar = 9527
        baz = "haha"
        qux = true
    }
    println("foo.bar=${foo.bar} foo.baz=${foo.baz} foo.qux=${foo.qux}")
}
```

## 更极端的方式

```kotlin
class Foo private constructor(
    val bar: Int,
    val baz: String,
    val qux: Boolean,
) {
    companion object {
        @JvmStatic
        operator fun invoke(block: Builder.() -> Unit = {}) = Builder().apply(block).build()
    }

    class Builder internal constructor() {
        var bar: Int = 0
        var baz: String = ""
        var qux: Boolean = false

        fun build() = Foo(bar = bar, baz = baz, qux = qux)
    }
}

fun main() {
    val foo = Foo {
        bar = 9527
        baz = "haha"
        qux = true
    }
    println("foo.bar=${foo.bar} foo.baz=${foo.baz} foo.qux=${foo.qux}")
}
```

该方式通过重载 invoke 运算符，使得调用代码更加精简，但是少了 build 字段导致可读性降低（特别是 java 下调用）。
有利有弊吧，个人项目我更喜欢后一种，团体项目推荐使用前者。

Java的调用方式，以前一种为例(后一种只需要将 `Foo.build` 替换为 `Foo.invoke`)：

```java
    Foo foo = Foo.build(new Function1<Foo.Builder, Unit>() {
        @Override
        public Unit invoke(Foo.Builder builder) {
            builder.setBar(9527);
            builder.setBaz("haha");
            builder.setQux(true);
            return Unit.INSTANCE;   // 也可以直接 return null
        }
    });
```

如果是Java 8及以上，还可以使用 lambda 的方式调用：

```java
    Foo foo = Foo.build(builder -> {
        builder.setBar(9527);
        builder.setBaz("haha");
        builder.setQux(true);
        return null;
    });
```

## 大道至简

primary constructor + val property + default value：

```kotlin
class Foo @JvmOverloads constructor(
    val bar: Int = 0,
    val baz: String = "",
    val qux: Boolean = false,
)

fun main() {
    val foo = Foo(bar = 9527, baz = "haha", qux = true)
    println("foo.bar=${foo.bar} foo.baz=${foo.baz} foo.qux=${foo.qux}")
}
```

kotlin 根本不需要构造者模式^_^

## kotlin 什么情况下需要构造者模式

* 对象有复杂验证逻辑（虽然 Foo class 在 init 方法里也能做校验，但复杂校验还是放在 builder 的 build 方法里更合适）
* 对象初始化步骤多，DSL 场景需要更语义化
* 需要可变配置对象 + 不可变最终对象
* 需要在 Java 调用时避免 telescoping constructor disaster
* 多层嵌套对象的 DSL

## Type-safe builder

Type-safe builder 在 Kotlin DSL 里是一个专门的概念，和传统的 Builder 模式相比，它额外提供了编译时的类型安全检查，以防止 DSL 使用错误。

在如下嵌套 Builder 的示例中，Config 里注释掉的 `// bar = 1024` 是错写了嵌套作用域的属性，如果 DSL 没有限制，Kotlin 会默认把外层 Foo.Builder.bar 暴露在嵌套 Config {} 里，容易出错。

Kotlin 提供 @DslMarker 来限制 DSL 作用域，防止嵌套错误访问。我们只需要自定义一个 DslMarker 注解 FooDsl，将其同时应用到 Foo.Builder 和 Config.Builder 上即可，这样 Config {} 里就无法调用外层作用域的 bar 属性。

```kotlin
@DslMarker
annotation class FooDsl

class Foo private constructor(
    val bar: Int,
    val baz: String,
    val qux: Boolean,
    val config: Config,
) {
    fun copy(block: Builder.() -> Unit = {}): Foo {
        val builder = Builder().apply {
            bar = this@Foo.bar
            baz = this@Foo.baz
            qux = this@Foo.qux
            config = this@Foo.config
        }
        return builder.apply(block).build()
    }

    companion object {
        @JvmStatic
        operator fun invoke(block: Builder.() -> Unit = {}) = Builder().apply(block).build()
    }

    @FooDsl
    class Builder internal constructor() {
        var bar: Int = 0
        var baz: String = ""
        var qux: Boolean = false
        var config: Config? = null

        fun build(): Foo {
            require(bar >= 0) { "bar must be >= 0" }
            return Foo(bar = bar, baz = baz, qux = qux, config = requireNotNull(config) { "config required" })
        }
    }
}

class Config private constructor(
    val haha: String,
    val hoho: Boolean,
) {

    override fun toString(): String {
        return "Config(haha=$haha, hoho=$hoho)"
    }

    companion object {
        @JvmStatic
        operator fun invoke(block: Builder.() -> Unit = {}) = Builder().apply(block).build()
    }

    @FooDsl
    class Builder internal constructor() {
        var haha: String = ""
        var hoho: Boolean = false

        fun build(): Config {
            require(haha.isNotEmpty()) { "haha must not be empty" }
            return Config(haha = haha, hoho = hoho)
        }
    }
}

fun main() {
    val foo = Foo {
        bar = 9527
        baz = "haha"
        qux = true
        config = Config {
            // 如果 Config.Builder 没有添加 FooDsl marker，可以在这里设置外层 Foo.Builder 的属性
            // bar = 1024
            haha = "xixi"
            hoho = true
        }
    }
    println("foo.bar=${foo.bar} foo.baz=${foo.baz} foo.qux=${foo.qux} foo.config=${foo.config}")

    val foo2 = foo.copy {
        bar = 1024
    }
    println("foo2.bar=${foo2.bar} foo2.baz=${foo2.baz} foo2.qux=${foo2.qux} foo2.config=${foo2.config}")
}
```

* [try.kotl.in](https://try.kotl.in/){:target="_blank"}
* [play.kotlinlang.org](https://play.kotlinlang.org/){:target="_blank"}

<!-- https://www.jianshu.com/p/f5f0d38e3e44 -->