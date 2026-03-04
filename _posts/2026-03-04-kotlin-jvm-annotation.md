---
layout: post
title: kotlin jvm 注解
date: 2026-03-04 14:30:00 +0800
categories: annotation
tags: kotlin、jvm
published: true
---

* content
{:toc}

## @JvmOverloads

生成默认参数重载

比较典型的应用场景是 Android 自定义 View：
```kotlin
class CustomLayout @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : ConstraintLayout(context, attrs, defStyleAttr) {}
```

构造函数因为添加了 JvmOverloads 注解会生成如下三个重载构造函数：
* CustomLayout(Context context)
* CustomLayout(Context context, AttributeSet attrs)
* CustomLayout(Context context, AttributeSet attrs, int defStyleAttr)

如果不加 @JvmOverloads 注解，在 xml 中使用 CustomLayout 会导致运行时 crash。这是因为 xml inflate 该控件时会使用构造函数 `CustomLayout(Context context, AttributeSet attrs)`，没有 JvmOverloads 注解就会找不到对应的重载构造函数。

## @JvmStatic

生成真正 static，仅可以用在 named objects 和 companion objects 里。

```kotlin
class Foo {

    companion object {
        @JvmStatic
        fun bar() {}
    }
}
```

上面的 kotlin 代码如果没有 JvmStatic 注解,在 java 中只能这样调用 `Foo.Companion.bar()`，有了 JvmStatic 注解就可以省略 Companion 直接调用 `Foo.bar()`。当然，即使有 JvmStatic 注解你依然可以使用 Companion 调用 `Foo.Companion.bar()`。

JvmStatic 注解不仅可以用在方法上，还可以用在属性上：
```kotlin
class Foo {

    companion object {
        @JvmStatic
        var bar: String = ""
    }
}
```

java 调用可以省略 Companion：
```java
Foo.setBar(1);
Foo.getBar();
```

上面的代码是以 companion objects 为例，JvmStatic 注解可以省去 java 调用中的 Companion。如果是 named objects，java 调用省去的是 INSTANCE。

## @JvmField

直接暴露字段

kotlin 类公开的属性默认会生成 setter/getter，添加了 JvmField 注解后直接暴露当前字段，不再生成 setter/getter。

```kotlin
class Foo {
    @JvmField
    var bar: Int = 0
}
```

java 中使用：
```java
Foo foo = new Foo();
foo.bar = 1;
println(foo.bar);
```

没有 JvmField 注解在 java 中调用只能使用 setter/getter：
```java
Foo foo = new Foo();
foo.setBar(1);
println(foo.getBar());
```

## @JvmName

改 JVM 方法名，可以用于：
* 成员函数
* 扩展函数
* 顶层函数

主要避免签名冲突，比如：
```kotlin
fun List<Int>.foo() {}
fun List<String>.foo() {}
```
上面的两个扩展函数因为类型擦除在编译阶段就会报错 `have the same JVM signature (foo(Ljava/util/List;)V)`，使用 JvmName 将其改为不同方法可以解决：

```kotlin
@JvmName("fooInt")
fun List<Int>.foo() {}

@JvmName("fooString")
fun List<String>.foo() {}
```

## @file:JvmName

改文件类名

Kotlin 顶层函数/顶层属性默认会被放进一个`文件名 + Kt` 的类，比如文件名叫 Foo.kt，默认会生成 FooKt.kt，供 java 调用。如果我们不喜欢 FooKt 这个默认名字，可以通过 @file:JvmName 注解来修改。需要注意的是，@file:JvmName 必须放在文件的最开始（package/import 等之前，注释除外）。

## @JvmSynthetic

对 Java 隐藏，仅 kotlin 可见，主要用于 SDK API 设计。

需要注意的是，该注解仅仅是给 target 打上 ACC_SYNTHETIC 标志，java 源码不可见，但是依然可以反射使用。

## @Throws

把 Kotlin 函数声明的异常写进 JVM 方法签名，让 Java 必须处理它。

```kotlin
class Foo {
    fun bar() {
        throw IOException()
    }
}
```

Kotlin 没有 checked exception 概念，在 Kotlin 中不需要 try/catch 或者声明 throws。java 调用不知道该方法会抛异常，加上 @Throws 注解可以让 Java 调用方强制处理异常。

```kotlin
class Foo {
    @Throws(IOException::class)
    fun bar() {
        throw IOException()
    }
```

另外，即使没有 java 调用，也建议加上该注解列出所有可能抛出的异常，增加可读性。

## @JvmSuppressWildcards、@JvmWildcard

控制 Kotlin 泛型在生成 JVM 字节码时是否生成 Java 通配符，解决 Kotlin 和 Java 泛型协变规则不同导致的互操作问题。
JvmSuppressWildcards 去掉通配符，JvmWildcard 强制生成通配符。

```kotlin
class Foo {
    fun bar(list: List<String>) {}
}
```
在 kotlin 里，List 是 List<out E>，在 JVM 层灰变成 `void bar(List<? extends String> list)`，这可能导致：
* Java 调用端类型推断复杂
* 重载冲突
* 无法直接匹配某些 API
* 框架反射匹配失败（例如某些 JSON / RPC 框架）

使用 @JvmSuppressWildcards 可以抑制通配符，该注解可以用在：

* 类型参数位置（仅对该参数生效）
    ```kotlin
    class Foo {
        fun bar(list: List<@JvmSuppressWildcards String>) {}
    }
    ```
* 函数（对整个函数生效）
    ```kotlin
    class Foo {
        @JvmSuppressWildcards
        fun bar(list: List<String>) {}
    }
    ```
* 类（对整个类生效）
    ```kotlin
    @JvmSuppressWildcards
    class Foo {
        fun bar(list: List<String>) {}
    }
    ```

以上只是列举了一些常见的 kotlin jvm 注解，更多注解参考 [kotlin-stdlib/kotlin.jvm](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.jvm/){:target="_blank"}。
