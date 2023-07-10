---
layout: post
title: Callback to Coroutine in Kotlin
date: 2023-07-10 09:30:00 +0800
categories: coroutine
tags: coroutine
published: true
---

* content
{:toc}

## Callback

很多三方库的耗时 api 在调用的时候，都需要传入一个 callback，library 内部自动开启子线程执行耗时操作，
待完成时再通过 callback 回调给调用方，避免阻塞调用所在的线程。热心一点的 library 会在 callback 的时候通过 Handler 切换到主线程，
当然更热心一点的 library 会切换到调用所在的线程。

```kotlin
// 这里的 Result 并不是 kotlin 的包裹类 `Result`，而是泛指 library 回调的结果
xxLibrary.ooApi(
    object : Callback() {
        override fun onSuccess(result: Result) {
            // do something success
        }

        override fun onFailure(throwable: Throwable) {
            // do something failure
        }
    }
)
```

## suspendCoroutine

现在我们想要给这个耗时 api 调用添加一个 timeout 限制，超过这个时间如果 library 没有回调返回就执行其他操作。
一个大家都比较容易想到的办法：在调用的时候开启一个倒计时，倒计时内 library 回调返回就取消倒计时，然后正常执行后续流程，
倒计时结束依然没有回调则执行 timeout 操作。

这个方法除了看起来不怎么优雅之外没有任何问题，事实上这也是唯一可行的方法，只不过如果我们自己手动去写这一套倒计时逻辑就会显得比较蠢。
kotlin 协程里边自带了一套 timeout suspend 方法，所以我们打算用协程来实现。

现在唯一的问题是我们需要把 library 的异步回调改成同步调用以便统计计时，而协程普通的 launch/async 方法无法完成这种 callback 异步转同步。
这时候就轮到 suspendCoroutine：

```kotlin
suspendCoroutine { continuation ->
    xxLibrary.ooApi(object : Callback {
        override fun onSuccess(result: Result) {
            continuation.resume(result)
        }

        override fun onFailure(throwable: Throwable) {
            continuation.resumeWithException(throwable)
        }

    })
}
```

再把 timeout 加上：

```kotlin
val exceptionHandler = CoroutineExceptionHandler { _, throwable ->
    // 这里捕获的不止 onFailure 异常，也包含代码 bug 引起的异常，be careful!
    Log.e(TAG, "Coroutine Exception: ${throwable.message}")
}
CoroutineScope(Dispatchers.Main).launch(exceptionHandler) {
    val ret = withTimeoutOrNull(timeout) {
        suspendCoroutine { continuation ->
            xxLibrary.ooApi(object : Callback {
                override fun onSuccess(result: Result) {
                    Log.d(TAG, "onSuccess: $result")
                    continuation.resume(result)
                }

                override fun onFailure(throwable: Throwable) {
                    Log.w(TAG, "onFailure: ${throwable.message}")
                    continuation.resumeWithException(throwable)
                }
            })
        }
    } ?: kotlin.run {
        val message = "Timed out waiting for $timeout ms"
        Log.w(TAG, "timeout: $message")
    }
}
```

看过协程源码的应该知道，协程 cps 转换，本质上是对 suspend 函数添加了一个 continuation 参数，中断函数并执行 block，根据结果成功异常与否调用 continuation.resume 或者 continuation.resumeWithException 恢复中断的函数。

launch/async 之类的操作对外屏蔽了 continuation，编译器根据关键字 suspend 在编译期间自动处理相关中断/恢复逻辑，而 suspendCoroutine 则把 continuation 抛了出来。不调用 continuation resume，当前函数便一直处于中断，使用 suspendCoroutine 需要开发者自行调用 resume 决定恢复中断的时机，大大提高了灵活性。有点类似于 View 与自定义 View。

## suspendCancellableCoroutine

suspendCoroutine 很好，但存在一个缺陷，以上面的代码为例，library 在超时后依然可能调用成功并执行 onSuccess callback，
因为只是 suspendCoroutine 协程超时了，我们并没有取消 library 调用，两者是独立的。

想要取消 library 调用，我们需要用到 suspendCancellableCoroutine，suspendCancellableCoroutine 相比 suspendCoroutine 多了一个 continuation.invokeOnCancellation 回调，可以在这里 cancel library 调用：

```kotlin
val exceptionHandler = CoroutineExceptionHandler { _, throwable ->
    // 这里捕获的不止 onFailure 异常，也包含代码 bug 引起的异常，be careful!
    Log.e(TAG, "Coroutine Exception: ${throwable.message}")
}
CoroutineScope(Dispatchers.Main).launch(exceptionHandler) {
    val ret = withTimeoutOrNull(timeout) {
        suspendCancellableCoroutine { continuation ->
            val taskId = xxLibrary.ooApi(object : Callback {
                override fun onSuccess(result: Result) {
                    Log.d(TAG, "onSuccess: $result")
                    continuation.resume(result)
                }

                override fun onFailure(throwable: Throwable) {
                    Log.w(TAG, "onFailure: ${throwable.message}")
                    continuation.resumeWithException(throwable)
                }
            })

            continuation.invokeOnCancellation {
                // 假设 xxLibrary.ooApi() 会返回一个 taskId 并有相应 cancel 方法
                xxLibrary.cancel(taskId)
            }
        }
    } ?: kotlin.run {
        val message = "Timed out waiting for $timeout ms"
        Log.w(TAG, "timeout: $message")
    }
}
```

当然，这一切的前提是 library 支持 cancel 方法，如果 library 本身并不支持 cancel，那么就只能小心 onSuccess 里的操作了：
onSuccess 除了 continuation.resume(result) 的其他操作都需要移到 suspendCancellableCoroutine 之外，因为 
suspendCancellableCoroutine 是带返回值的（超时之后 onSuccess 的 continuation.resume(result) 不会返回），所以照样可以在外面进行调用成功的后续操作，也避免了超时之后 onSuccess 继续执行带来的副作用。

## resume twice

目前为止，一切看起来都很合理正常，但上面 suspendCoroutine 和 suspendCancellableCoroutine 的代码都存在一个致命问题：
如果调用一次 library，成功返回了不止一个结果(即 onSuccess 回调了多次)，那么第二次 continuation.resume(result) 就会导致 crash `java.lang.IllegalStateException: Already resumed`。

我们可以简单地在 onSuccess 里重复 continuation.resume(result) 即可模拟出该 crash（我知道有人会说这应该是 library 的问题，但作为调用方来说我们也有义务使自己的代码更加健壮对吧）。

针对这个问题，只需要做一点点小小的改动，在 continuation resume 时增加一个判断即可：

```kotlin
val exceptionHandler = CoroutineExceptionHandler { _, throwable ->
    // 这里捕获的不止 onFailure 异常，也包含代码 bug 引起的异常，be careful!
    Log.e(TAG, "Coroutine Exception: ${throwable.message}")
}
CoroutineScope(Dispatchers.Main).launch(exceptionHandler) {
    val ret = withTimeoutOrNull(timeout) {
        suspendCancellableCoroutine { continuation ->
            val taskId = xxLibrary.ooApi(object : Callback {
                override fun onSuccess(result: Result) {
                    Log.d(TAG, "onSuccess: $result")
                    if (continuation.isActive) {
                        continuation.resume(result)
                    } else {
                        // continuation is completed or cancelled, ignore
                    }
                }

                override fun onFailure(throwable: Throwable) {
                    Log.w(TAG, "onFailure: ${throwable.message}")
                    if (continuation.isActive) {
                        continuation.resumeWithException(throwable)
                    }
                }
            })

            continuation.invokeOnCancellation {
                // 假设 xxLibrary.ooApi() 会返回一个 taskId 并有相应 cancel 方法
                xxLibrary.cancel(taskId)
            }
        }
    } ?: kotlin.run {
        val message = "Timed out waiting for $timeout ms"
        Log.w(TAG, "timeout: $message")
    }
}
```

## one more thing

BTW，都已经 2023 年了，写 library 的朋友可以考虑把 callback 的 onSuccess 和 onFailure 回调用 kotlin 的包裹类 `Result` 合并一下：

```kotlin
// 这里的 Result 指的就是 kotlin 的包裹类 `Result`
xxLibrary.ooApi(
    object : Callback() {
        override fun onComplete(result: Result) {
            result.onSuccess { 
                // do something success
            }.onFailure { 
                // do something failure
            }
        }
    }
)
```

合并之后的好处是，Callback 的回调方法只有一个 onComplete，那么可以使用 SAM 转换：

```kotlin
xxLibrary.ooApi { result ->
    result.onSuccess { 
        // do something success
    }.onFailure { 
        // do something failure
    }
}
```

很多现代语言都在借鉴使用 Result 包裹 success 和 failure 进行统一返回。

<!-- https://amitshekhar.me/blog/callback-to-coroutines-in-kotlin -->
<!-- https://stackoverflow.com/a/48562175 -->
<!-- https://stackoverflow.com/a/62536933 -->
