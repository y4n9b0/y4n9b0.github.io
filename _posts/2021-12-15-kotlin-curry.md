---
layout: post
title:  Kotlin 柯里化和偏函数
date:   2021-12-15 16:00:00 +0800
categories: functional-programming
tags: paradigm
published: true
---

* content
{:toc}

## 一、柯里化

柯里化：把一个多参函数转换成每次只接受一个参数的函数

## 二、柯里化推导

从大家都很容易理解的一个简单版本开始：
```kotlin
fun <P1, P2, R> ((P1, P2) -> R).curry(): (P1) -> (P2) -> R {
    return { p1: P1 -> { p2: P2 -> this(p1, p2) } }
}
```

函数 `((P1, P2) -> R)` 可以用 Function2 代替(Function2 是 SAM接口，所以这里其实是 SAM 逆转换)：
```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.curry(): (P1) -> (P2) -> R {
    return { p1: P1 -> { p2: P2 -> this(p1, p2) } }
}
```

由于扩展函数内就一条 return 语句，所以可以直接使用等号把 return 表达式赋值给扩展函数：
```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.curry(): (P1) -> (P2) -> R = 
    { p1: P1 -> { p2: P2 -> this(p1, p2) } }
```

把 lambda 表达式替换成匿名函数： 
```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.curry(): (P1) -> (P2) -> R =
    fun(p1: P1): (P2) -> R {
        return { p2: P2 -> this(p1, p2) }
    }
```

P1 匿名函数（为方便理解 这里我们简单把 P1 入参的匿名函数记作 P1 匿名函数，后续雷同）的函数体也是只有一个 return 语句，同样地 我们直接使用等号把 return 表达式赋值给 P1 匿名函数：
```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.curry(): (P1) -> (P2) -> R =
    fun(p1: P1): (P2) -> R = { p2: P2 -> this(p1, p2) }
```

再次把 lambda 表达式替换成匿名函数：
```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.curry(): (P1) -> (P2) -> R =
    fun(p1: P1): (P2) -> R = fun(p2: P2): R {
        return this(p1, p2)
    }
```

P2 匿名函数的函数体依然只有一个 return 语句，直接使用等号把 return 表达式赋值给 P2 匿名函数：
```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.curry(): (P1) -> (P2) -> R = 
    fun(p1: P1): (P2) -> R = fun(p2: P2): R = this(p1, p2)
```

最后，由于 kotlin 具有自动推导类型功能，我们可以把 P1 匿名函数、P2 匿名函数、curry 扩展函数的返回类型都给去掉：
```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.curry() = fun(p1: P1) = fun(p2: P2) = this(p1, p2)
```

最终的柯里化版本就出炉了，其实光看最终版也不难理解，只需要知道代码右结合性、类型自动推导、匿名函数就好了。<br>
以上仅仅是针对 Function2 的柯里化扩展，诸如 Function3 ~ Function22 都是依葫芦画瓢，以此类推。真正的难点在于 `FunctionN`。

另外，对于最终版还可以显式调用 Function2 的 invoke 方法（不建议这种装*写法，仅为了列出所有的柯里化写法）：
```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.curry() = fun(p1: P1) = fun(p2: P2) = invoke(p1, p2)
```

## 三、反柯里化推导

同样地，从大家都很容易理解的一个简单版本开始：
```kotlin
fun <P1, P2, R> ((P1) -> (P2) -> R).uncurry(): (P1, P2) -> R {
    return { p1: P1, p2: P2 -> this(p1)(p2) }
}
```
    
用 Function2 替换扩展函数返回值 `(P1, P2) -> R` (这里仅仅是为了演示两种写法是等价的)
```kotlin
fun <P1, P2, R> ((P1) -> (P2) -> R).uncurry(): Function2<P1, P2, R> {
    return { p1: P1, p2: P2 -> this(p1)(p2) }
}
```

由于 kotlin 可以自动推导类型，去掉扩展函数返回值(提前去掉便于后续少写点代码，也可以后面一并再去掉)：
```kotlin
fun <P1, P2, R> ((P1) -> (P2) -> R).uncurry() {
    return { p1: P1, p2: P2 -> this(p1)(p2) }
}
```

函数体只有一条 return 语句，直接等号赋值：
```kotlin
fun <P1, P2, R> ((P1) -> (P2) -> R).uncurry() = { p1: P1, p2: P2 -> this(p1)(p2) }
```

匿名函数替换掉 lambda 表达式：
```kotlin
fun <P1, P2, R> ((P1) -> (P2) -> R).uncurry() = fun(p1: P1, p2: P2): R { 
    return this(p1)(p2)
}
```

干掉匿名函数的 return，直接等号赋值：
```kotlin
fun <P1, P2, R> ((P1) -> (P2) -> R).uncurry() = fun(p1: P1, p2: P2): R = this(p1)(p2)
```

去掉匿名函数返回值：
```kotlin
fun <P1, P2, R> ((P1) -> (P2) -> R).uncurry() = fun(p1: P1, p2: P2) = this(p1)(p2)
```

反柯里化为 Function2 函数的最终版就完成了。同样，这里的 this 调用同样也可以替换为 invoke 调用。

## 四、测试

```kotlin
private fun foo(a: Int, b: Int, c: Int, d: Int): Int {
    return a + 10 * b + 100 * c + 1000 * d
}

@Test
fun testCurrying() {
    val curry = ::foo.curry()
    val uncurry = curry.uncurry()

    assertEquals(foo(9, 5, 2, 7), curry(9)(5)(2)(7))
    assertEquals(foo(9, 5, 2, 7), uncurry(9, 5, 2, 7))

    assertNotEquals(foo(9, 5, 2, 7), curry(1)(3)(1)(4))
    assertNotEquals(foo(9, 5, 2, 7), uncurry(1, 3, 1, 4))
}
```

## 五、偏函数

偏函数：固定一个多元函数的一些参数，然后产生另一个更小元的函数，那么这个小元函数就是原函数的一个偏函数。<br>
元是指函数参数的个数，比如一个带有两个参数的函数被称为二元函数。

```kotlin
fun <P1, P2, R> Function2<P1, P2, R>.partial1(p1: P1) = fun(p2: P2) = this(p1, p2)
fun <P1, P2, R> Function2<P1, P2, R>.partial2(p2: P2) = fun(p1: P1) = this(p1, p2)
```

<!-- https://github.com/MarioAriasC/funKTionale/blob/master/funktionale-currying/src/main/kotlin/org/funktionale/currying/namespace.kt -->
<!-- https://github.com/JetBrains/kotlin/blob/master/spec-docs/function-types.md -->
<!-- https://sorokod.github.io/post/2018-09-09-kotlin-functions-as-objects/ -->
