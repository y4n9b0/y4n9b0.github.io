---
layout: post
title: mac 安装 windows 双系统
date: 2021-08-22 16:00:00 +0800
categories: windows
tags: windows
published: false
---

* content
{:toc}

## mac 安装 windows 的几种方式

1. 格盘安装单windows 系统
   这种方式我也没尝试过，大家可以自行搜索
2. 安装双系统
   这应该是 mac 最常见的安装方式了，为什么这么说呢？是因为这种方式比上面的更方便。
   macOS 自带的 boot camp 工具可以很方便的帮助我们完成安装，所以这也是大多数人都比较推荐的方式，也是本文所采用的方式。

## 准备工作

1. 插上电源
2. 连上网络
3. 设置节能模式 禁止休眠
4. 下载 windows ISO镜像文件

## Boot camp

macOS 有个系统工具叫 Boot Camp 助理，中文又名启动转换助理。

https://support.apple.com/zh-cn/HT208198



```kotlin
class FooActivity : AppCompatActivity() {

    override fun onDestroy() {
        super.onDestroy()
        handler.removeCallbacksAndMessages(null)
    }
}
```

## 无需擦屁股的 Handler

Handler 的内存泄漏虽然有了解决办法，但算不上一个完美的方案，一方面容易忘记，另一方面写起来略不爽。人们往往热衷于一项伟大工程脚手架的搭建，但没人喜欢在结束时清扫战场。
有点像上厕所，我们享受拉屎带来的快感，却讨厌擦屁股，但出于某些原因又不得不擦。那么问题来了，有没有一种办法保证无论怎么拉、拉完都无需自己擦？

回到 Handler 上来说，答案是肯定的，我们可以将 Handler 和 Jetpack Lifecycle 结合起来，利用 lifecycle 生命周期的回调自动清理 Handler。

```kotlin
fun LifecycleOwner.lifecycleHandler(looper: Looper = Looper.getMainLooper()): Handler = LifecycleHandler(this, looper)

private class LifecycleHandler(
    private val owner: LifecycleOwner,
    looper: Looper = Looper.getMainLooper()
) : Handler(looper), LifecycleObserver {
    init {
        owner.lifecycle.addObserver(this)
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    private fun onDestroy() {
        owner.lifecycle.removeObserver(this)
        removeCallbacksAndMessages(null)
    }
}
```

我们定义了一种 继承自 Handler 的 LifecycleHandler，监听 Lifecycle ON_DESTROY 事件，在回调里自动 removeCallbacksAndMessages。

由于 Lifecycle 是官方出品，日常的 Activity 和 Fragment 都已经自带 LifecycleOwner 属性，所以又对 LifecycleOwner 进行了 lifecycleHandler 扩展，使用起来相当方便。

## 函数类型的顶层属性

上面使用的 LifecycleOwner 的扩展函数，其本质上是一个函数。接下来我们看另一种扩展方案：

```kotlin
val lifecycleHandler: LifecycleOwner.(Looper) -> Handler = {
    LifecycleHandler(this, it)
}
```

这里定义的是一个函数类型的顶层属性，其调用方式比之前的扩展函数更加灵活，不仅可以这样 LifecycleOwner.lifecycleHandler(Looper)，还可以这样 lifecycleHandler(LifecycleOwner，Looper)。
作为对比，前文的扩展函数只能这样调用 LifecycleOwner.lifecycleHandler(Looper)。

当然顶层属性的方案也是有缺点的，由于其入参无法赋默认值，也就导致了无论何种调用方式 参数皆不可缺省。

## 多参的函数类型属性

熟悉 Handler 的盆友应该知道，Handler 的构造参数其实有三个，但对外开放到了两个，所以上面一个参数(Looper)的方案并不能覆盖所有的应用场景，我们应该把 Handler.Callback 加入进来。
对于函数类型的顶层属性，多个参数又该如何书写呢？

实际上 lambda 的 it 仅仅是在只有一个入参时的隐式代指，我们知道 lambda 的参数是可以显式命名的，所以上面的顶层属性又等同于如下写法：

```kotlin
val lifecycleHandler: LifecycleOwner.(Looper) -> Handler = { looper ->
    LifecycleHandler(this, looper)
}
```

到这里，我们就不难推导出多个参数的 lambda 写法（只需将多个入参按顺序显式化即可）：

```kotlin
val lifecycleHandler: LifecycleOwner.(Looper, Handler.Callback?) -> Handler = { looper, callback ->
    LifecycleHandler(this, looper, callback)
}
```

最后，不太推荐函数类型顶层属性的方案，可以作为学习了解之用。原因嘛主要是其入参不可缺省，参数一旦多起来，你懂的..

<!-- https://zhuanlan.zhihu.com/p/20246047 -->
<!-- https://zhuanlan.zhihu.com/p/34435136 -->
<!-- https://blog.csdn.net/weixin_35719402/article/details/111971491 -->
<!-- https://support.apple.com/zh-cn/HT208198 -->
