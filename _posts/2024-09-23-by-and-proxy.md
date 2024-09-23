---
layout: post
title: 委托与代理
date: 2024-09-23 23:30:00 +0800
categories: kotlin
tags: kotlin
published: true
---

* content
{:toc}

## kotlin 委托

艺术源于生活：

```kotlin
interface Employee {
    fun doWork(){}
}

class Me : Employee {
    override fun doWork() {
        println("Let's do it!")
    }
}

class Leader(private val me: Me) : Employee {
    override fun doWork() {
        me.doWork()
    }
}
```

这是一个最简单的委托模式，相信各位职场牛马都能轻松看懂。<br>
上面的代码虽然通俗易懂，但写法有一点点繁琐，特别是接口有多个方法都需要委托的时候。我们可以使用 kotlin 关键字 by 稍做优化，减少不必要的 boilerplate code：

```kotlin
class Leader(private val me: Me) : Employee by me
```

更进一步，by 后面直接 new 出委托对象：

```kotlin
class Leader : Employee by Me()
```

至此，你应该已经学会 kotlin 的类委托啦！

<!-- **属性委托** -->
<!-- Delegates.notNull() -->

## Java 动态代理

好困🥱

## 学会了就要搞事情

作为资深 UI 开发工程师，各位平日里应该没少绘动画：

```kotlin
fun foo(anim: Animation) {
    anim.setAnimationListener(object : Animation.AnimationListener {
        override fun onAnimationEnd(animation: Animation?) {
            // continue...
        }
            
        override fun onAnimationStart(animation: Animation?) {
            // I don't care!
        }
            
        override fun onAnimationRepeat(animation: Animation?) {
            // I don't give a shit!
        }
    })
}
```

很多时候我们只关心动画结束，以便接着做其他事情。但是 Animation 库的设计者需要照顾所有使用者，得把可能用到的回调都给暴露出来。这样一来就会导致我们需要实现很多不关心的回调方法。

一种解决方案是先定义一个空实现类，后续用到的地方各自去重写自己关心的回调：

```kotlin
open class EmptyAnimationListener : Animation.AnimationListener {
    override fun onAnimationStart(animation: Animation?) {}

    override fun onAnimationEnd(animation: Animation?) {}

    override fun onAnimationRepeat(animation: Animation?) {}
}

fun foo(anim: Animation) {
    anim.setAnimationListener(object : EmptyAnimationListener() {
        override fun onAnimationEnd(animation: Animation?) {
            // continue...
        }
    })
}
```

看起来只是稍好一点，毕竟 `Animation.AnimationListener` 仅有三个方法需要实现，要是换成 `Application.ActivityLifecycleCallbacks` 呢？

优雅的终极方案：

```kotlin
internal inline fun <reified T : Any> noOpDelegate(): T {
    val javaClass = T::class.java
    return Proxy.newProxyInstance(javaClass.classLoader, arrayOf(javaClass)) { _, _, _ ->
        // no op
    } as T
}

fun foo(anim: Animation) {
    anim.setAnimationListener(object : Animation.AnimationListener by noOpDelegate() {
        override fun onAnimationEnd(animation: Animation) {
            // continue...
        }
    })
}
```

最后，都 4202 年了，写库的机灵鬼们，能不能把接口方法默认实现给加上，两大括号的事儿！

<!-- https://juejin.cn/post/7243627371994038309 -->
<!-- https://mp.weixin.qq.com/s/ZwmPXifzdO79YnOX9WLwHA -->
<!-- https://paradisehell.org/2021/02/20/understand-java-dynamic-proxy-deeply/ -->
<!-- https://juejin.cn/post/6974018412158664734 -->