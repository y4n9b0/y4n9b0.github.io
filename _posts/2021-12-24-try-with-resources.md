---
layout: post
title:  如何正确地关闭文件流
date:   2021-12-24 19:00:00 +0800
categories: kotlin
tags: try-with-resources
published: true
---

* content
{:toc}

## 一、JDK 7 try with resources

众所周知，对文件进行读写，结束之后如果不关闭文件流就会存在内存泄漏风险。早前的 Java 关闭文件流一般都是在 try catch 的 finally 代码块里进行。然而，由于文件流的 close 方法本身也可能抛异常，所以往往还得在 finally 里对每一个 close 进行 try catch，一个简简单单的文件流关闭搞得代码 pretty fxxking ugly！在 JDK7 引入 try-with-resources 之后，文件流关闭总算优雅多了。

**敲黑板**：<br>
最好是对 finally 里的每一个 close 都单独进行 try catch，否则 存在多个文件流的情况下，如果前面的文件流关闭抛异常，后面的文件流关闭就被跳过了。见过太多的代码都是多个文件流的 close 共用一个 try catch，这样是不对的。之所以没出问题，一方面是因为小概率事件不容易发生，另一方面即便发生了顶多也就是内存泄漏 还不至于崩给你看。但是，咱们作为一个称职的键盘侠，借鉴代码要严谨。

## 二、Kotlin 1.2 use

众所周知，kotlin 去掉了 Checked Exception，当然更不用说 try-with-resources 了。那么 kotlin 怎么优雅地关闭文件流呢？

自 kotlin 1.2 起，官方出品了一个 Closeable 的 use 扩展函数，这是一个高阶函数，其实现原理很简单 - 执行完入参代码块后再关闭 Closeable 自身。使用方式如下所示：

```kotlin
fun foo() {
    FileInputStream(File("path")).use { fis ->
        InputStreamReader(fis).use { isr ->
            BufferedReader(isr).use { br ->
                // do your work
            }
        }
    }  
}
```

出于性能/易用性的考虑，JDK 在设计 io 流的时候进行了多层封装，导致咱们在调用的时候需要做大量的嵌套构造，先通过 InputStream 去构造 StreamReader，再通过 StreamReader 去构造 BufferedReader。
执行完任务后对所有的 stream/reader 进行关闭。

作为一个有追求的键盘侠，直觉告诉咱们 上面的代码一定还有更优雅的方式。<br>
Indeed, it does! <br>
事实上，我们并不需要显式关闭被嵌套构造的 stream 和 reader，只需要关闭最外层构造的 reader 即可（这里 最外层的 reader 即是 BufferedReader）。虽然有这么多的 stream 和 reader 
实例对象，但实际真正意义上的文件流只有一个（毕竟 咱们只打开了一个文件 且仅打开了一次），它通过嵌套构造的形式贯穿 stream 和 reader。如果仔细查看源码，就可以知道关闭最外层的 reader 会递归调用嵌套 stream/reader 的 close 方法。当然，这种设计不仅仅局限于 reader，而是整个 io 流，包括 socket 之类。

所以，上面的代码可以简化成如下方式：

```kotlin
fun foo() {
    BufferedReader(InputStreamReader(FileInputStream(File("path")))).use {
        // do your work
    }
}
```

可能会有人问：既然嵌套调用 close 关闭文件流，那么能否把关闭最外层 reader 改为关闭最内层 stream 呢（或者中间的某层 stream/reader）？<br>
理论上是可以的，反正最终都可以关闭打开的文件（这才是源头）。但实际上，各层 stream/reader 的 close 方法除了调用内层嵌套 stream/reader 的 close，一般都还会有些当前类的其他资源释放操作。如果改为关闭内层 stream/reader，外层 reader 的这些操作就无法被执行。<br>
所以，don't do that!

## 三、Being good enough is not good enough

So far，文件流关闭的问题算是彻底解决了。但是，作为一个有追求的键盘侠，看到上面文件流嵌套构造的书写方式，内心还是万马奔腾(Rule No.1：不要做括号爱好者，无论是小括号还是大括号(λ同学回避一下))。接下来让咱们自定义一个简单的中缀操作符摆脱它：

```kotlin
infix fun <T : Closeable?, R> T.to(block: (T) -> R): R = block(this)

fun foo() {
    (FileInputStream(File("path")) to ::InputStreamReader to ::BufferedReader).use {
        // do your work
    }
}
```

怎么样？这下是不是顺眼多了？<br>
但是，作为一个有追求的键盘侠，咱们依然还是不能够忍受 Stream 对 File 的嵌套构造，one more little optimization:

```kotlin
infix fun <T, R> T.to(block: (T) -> R): R = block(this)

fun foo() {
    ("path" to ::File to ::FileInputStream to ::InputStreamReader to ::BufferedReader).use {
        // do your work
    }
}
```

只需要去掉泛型 T 的类型限定，中缀操作符 to 就可以用于 File 和 String(以及任何其他类型)。Wow~是不是很 amazing 很 awesome？其实，这就是某些人嘴上常嚷嚷的控制反转。<br>
怎么样？你学废了吗:smile:

<!-- https://www.geeksforgeeks.org/correct-ways-to-close-inputstream-and-outputstream-in-java-with-examples/ -->
<!-- http://www.javapractices.com/topic/TopicAction.do?Id=8 -->
<!-- https://stackoverflow.com/questions/65949050/kotlin-nested-use-function-for-java-try-with-resource -->
<!-- http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/sun/nio/cs/StreamDecoder.java -->