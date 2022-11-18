---
layout: post
title:  variable argument ViewModel
date:   2022-11-18 19:30:00 +0800
categories: publish
tags: ViewModel
published: true
---

* content
{:toc}

## 前言

```gradle
implementation androidx.fragment:fragment-ktx:1.5.4
implementation androidx.activity:activity-ktx:1.5.1
```

Android 官方 Activity/Fragment 扩展库里对 ViewModel 的扩展构造方法无法携带参数，使用不太方便，轮一个支持可变长参数的 ViewModel 扩展方法。

本轮子绝大多数时候都可替代官方的扩展方法（不传构造参数即可），唯一例外的是 SaveState 重建场景，官方默认使用的 SavedStateViewModelFactory 支持 SaveState 重建，由于这类场景过于小众且支持起来比较复杂，故本轮子暂未考虑。

## vararg

`ViewModelExt.kt`

```kotlin
import android.os.Build
import androidx.activity.ComponentActivity
import androidx.activity.viewModels
import androidx.annotation.MainThread
import androidx.fragment.app.Fragment
import androidx.fragment.app.activityViewModels
import androidx.fragment.app.viewModels
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

fun argViewModelFactory(vararg args: Any): ViewModelProvider.Factory =
    object : ViewModelProvider.NewInstanceFactory() {
        @Suppress("UNCHECKED_CAST")
        override fun <VM : ViewModel> create(modelClass: Class<VM>): VM {
            modelClass.constructors.filter {
                args.size == if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                    it.parameterCount
                } else {
                    it.parameterTypes.size
                }
            }.forEach {
                try {
                    return it.newInstance(*args) as VM
                } catch (ignore: Exception) {
                }
            }
            throw InstantiationException("Cannot create an instance of $modelClass")
        }
    }

@MainThread
inline fun <reified VM : ViewModel> Fragment.argViewModels(vararg args: Any) =
    viewModels<VM> { argViewModelFactory(*args) }

@MainThread
inline fun <reified VM : ViewModel> Fragment.activityArgViewModels(vararg args: Any) =
    activityViewModels<VM> { argViewModelFactory(*args) }

@MainThread
inline fun <reified VM : ViewModel> ComponentActivity.argViewModels(vararg args: Any) =
    viewModels<VM> { argViewModelFactory(*args) }
```

如果扩展函数显式指定了返回类型，则 viewModels 可以省去类型<VM>指定（由编译器智能推导）:

```kotlin
@MainThread
inline fun <reified VM : ViewModel> Fragment.argViewModels(vararg args: Any): Lazy<VM> =
    viewModels { argViewModelFactory(*args) }
```

使用方式：

```kotlin
// 1
private val viewModel: FooViewModel by argViewModels(2, true, "haha")

// 2
private val viewModel by argViewModels<FooViewModel>(2, true, "haha")


// 3
private val args: Array<Any> = arrayOf(2, true, "haha")
private val viewModel: FooViewModel by argViewModels(*args)

// 4
private val args: Array<Any> = arrayOf(2, true, "haha")
private val viewModel by argViewModels<FooViewModel>(*args)


class FooViewModel(a: Int, b: Boolean, c: String) : ViewModel()
```

## Array

使用 vararg 可变参数的函数不能用作高阶函数的参数，将 vararg 替换成 Array 才可以用于高阶函数，当然本文这里不存在高阶函数问题，仅记录该用法：

```kotlin
import android.os.Build
import androidx.activity.ComponentActivity
import androidx.activity.viewModels
import androidx.annotation.MainThread
import androidx.fragment.app.Fragment
import androidx.fragment.app.activityViewModels
import androidx.fragment.app.viewModels
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

fun argViewModelFactory(args: Array<out Any>): ViewModelProvider.Factory =
    object : ViewModelProvider.NewInstanceFactory() {
        @Suppress("UNCHECKED_CAST")
        override fun <VM : ViewModel> create(modelClass: Class<VM>): VM {
            modelClass.constructors.filter {
                args.size == if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                    it.parameterCount
                } else {
                    it.parameterTypes.size
                }
            }.forEach {
                try {
                    return it.newInstance(*args) as VM
                } catch (ignore: Exception) {
                }
            }
            throw InstantiationException("Cannot create an instance of $modelClass")
        }
    }

@MainThread
inline fun <reified VM : ViewModel> Fragment.argViewModels(vararg args: Any) =
    viewModels<VM> { argViewModelFactory(args) }

@MainThread
inline fun <reified VM : ViewModel> Fragment.activityArgViewModels(vararg args: Any) =
    activityViewModels<VM> { argViewModelFactory(args) }

@MainThread
inline fun <reified VM : ViewModel> ComponentActivity.argViewModels(vararg args: Any) =
    viewModels<VM> { argViewModelFactory(args) }
```

## one more thing

由于传入的参数都是 Any 类型，故无法通过参数确定对应的 constructor，只能匹配 constructor 参数个数，然后挨个尝试，
这样效率不够高，而且感觉不是很优雅。一种解决方式便是在传入参数的时候同时传入参数对应的 class type，通过参数的 class type 可以直接获取对应的 constructor，缺点就是调用方会相对麻烦一些。

```kotlin
import androidx.activity.ComponentActivity
import androidx.activity.viewModels
import androidx.annotation.MainThread
import androidx.fragment.app.Fragment
import androidx.fragment.app.activityViewModels
import androidx.fragment.app.viewModels
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import java.lang.reflect.InvocationTargetException

@Throws(
    NoSuchMethodException::class,
    SecurityException::class,
    IllegalAccessException::class,
    IllegalArgumentException::class,
    InstantiationException::class,
    InvocationTargetException::class,
)
fun argViewModelFactory(vararg args: Pair<Any, Class<*>>): ViewModelProvider.Factory =
    object : ViewModelProvider.NewInstanceFactory() {
        override fun <VM : ViewModel> create(modelClass: Class<VM>): VM {
            val params: Array<Any> = args.map { it.first }.toTypedArray()
            val signature: Array<Class<*>> = args.map { it.second }.toTypedArray()
            val constructor = modelClass.getConstructor(*signature)
            return constructor.newInstance(*params)
        }
    }

@MainThread
inline fun <reified VM : ViewModel> Fragment.argViewModels(
    vararg args: Pair<Any, Class<*>>
) = viewModels<VM> { argViewModelFactory(*args) }

@MainThread
inline fun <reified VM : ViewModel> Fragment.activityArgViewModels(
    vararg args: Pair<Any, Class<*>>
) = activityViewModels<VM> { argViewModelFactory(*args) }

@MainThread
inline fun <reified VM : ViewModel> ComponentActivity.argViewModels(
    vararg args: Pair<Any, Class<*>>
) = viewModels<VM> { argViewModelFactory(*args) }
```

相应的使用方式：

```kotlin
// 1
private val viewModel: FooViewModel by argViewModels(
    2 to Int::class.java,
    true to Boolean::class.java,
    "haha" to String::class.java
)

// 2
private val viewModel by argViewModels<FooViewModel>(
    2 to Int::class.java,
    true to Boolean::class.java,
    "haha" to String::class.java
)


// 3
private val pairs: Array<Pair<Any, Class<*>>> = arrayOf(
    2 to Int::class.java,
    true to Boolean::class.java,
    "haha" to String::class.java
)
private val viewModel: FooViewModel by argViewModels(*pairs)

// 4
private val pairs: Array<Pair<Any, Class<*>>> = arrayOf(
    2 to Int::class.java,
    true to Boolean::class.java,
    "haha" to String::class.java
)
private val viewModel by argViewModels<FooViewModel>(*pairs)


class FooViewModel(a: Int, b: Boolean, c: String) : ViewModel()
```

## another one more thing

传入参数对应的 class type 是为了优雅高效地寻找到对应的 constructor，那么问题来了：我们为何不直接把 constructor 传进去呢？

here we go:

```kotlin
import androidx.activity.ComponentActivity
import androidx.activity.viewModels
import androidx.annotation.MainThread
import androidx.fragment.app.Fragment
import androidx.fragment.app.activityViewModels
import androidx.fragment.app.viewModels
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

fun <VM : ViewModel> argViewModelFactory(
    constructor: (Array<out Any>) -> VM
): (Array<out Any>) -> ViewModelProvider.Factory = {
    object : ViewModelProvider.NewInstanceFactory() {
        @Suppress("UNCHECKED_CAST")
        override fun <VM : ViewModel> create(modelClass: Class<VM>): VM {
            return constructor(it) as VM
        }
    }
}

@MainThread
inline fun <reified VM : ViewModel> Fragment.argViewModels(
    noinline constructor: (Array<out Any>) -> VM,
    vararg args: Any
): Lazy<VM> = viewModels { argViewModelFactory(constructor)(args) }

@MainThread
inline fun <reified VM : ViewModel> Fragment.activityArgViewModels(
    noinline constructor: (Array<out Any>) -> VM,
    vararg args: Any
): Lazy<VM> = activityViewModels { argViewModelFactory(constructor)(args) }

@MainThread
inline fun <reified VM : ViewModel> ComponentActivity.argViewModels(
    noinline constructor: (Array<out Any>) -> VM,
    vararg args: Any
): Lazy<VM> = viewModels { argViewModelFactory(constructor)(args) }
```

相应的使用方式：

```kotlin
private val viewModel by argViewModels(
    { FooViewModel(it[0] as Int, it[1] as Boolean, it[2] as String) },
    2, true, "haha"
)

class FooViewModel(a: Int, b: Boolean, c: String) : ViewModel()
```

可以看到，调用方其实并没有省事儿，反而还更麻烦了。不过，也不失为一种思路。