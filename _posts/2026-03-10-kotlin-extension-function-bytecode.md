---
layout: post
title: kotlin 扩展函数 bytecode
date: 2026-03-10 20:30:00 +0800
categories: extension-function
tags: kotlin、jvm
published: true
---

* content
{:toc}

谈到 kotlin 扩展函数，很多人有个误区，认为其本质上是 java 静态方法。

比如我们新建一个 kotlin 文件 StringUtil，写入如下扩展方法：
```kotlin
fun String.foo() {}
```

通过 Android Studio 自带的 Tools -> Kotlin -> Show Kotlin Bytecode，可以得到如下反编译的 java 代码：
```java
public final class StringUtilsKt {
   public static final void foo(@NotNull String $this$foo) {
      Intrinsics.checkNotNullParameter($this$foo, "<this>");
   }
}
```

可以看到，最终确实生成了一个静态方法。<br>
但是，这里生成静态方法是因为这是一个顶层扩展函数，如果把扩展函数放在某个类或者接口里，生成的会是某个类的实例方法：
```kotlin
class Bar {
    fun String.foo() {}
}
```

```java
public final class Bar {
   public final void foo(@NotNull String $this$foo) {
      Intrinsics.checkNotNullParameter($this$foo, "<this>");
   }
}
```

这样写有什么用？<br>
限制使用范围，该扩展方法仅可以用在 Bar 及其子类（如果是 open class 可被继承）的作用域内，可以防止滥用或者避免同名函数冲突。当然，绝大多数情况下都不应该这样使用。
