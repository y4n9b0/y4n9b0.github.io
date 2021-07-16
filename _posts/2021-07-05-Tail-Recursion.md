---
layout: post
title: 尾递归
date: 2021-07-05 20:20:00 +0800
categories: programming
tags: paradigm
published: true
---

* content
{:toc}

## 递归

计算斐波那契数列第n项

```kotlin
fun fib(n: Int): Int = if (n <= 1) 1 else fib(n - 1) + fib(n - 2)
```

## 尾递归

当递归调用是整个函数体中最后执行的语句且它的返回值不属于表达式的一部分时，这个递归调用就是尾递归。

```kotlin
// 尾递归
fun fib(n: Int, a: Int = 1, b: Int = 1): Int = if (n == 0) a else fib(n - 1, b, a + b)
```

如下递归调用虽然是在函数体的最后，但是它的返回值属于表达式的一部分，故不是尾递归：

```kotlin
// 线性递归
fun fib(n: Int, a: Int = 1, b: Int = 1): Int = if (n <= 1) 1 else a + fib(n - 1, b, a + b)
```

### 尾递归与普通递归相比有什么好处

非尾递归，下一个函数结束以后此函数还有后续，所以必须保存本身的环境以供处理返回值。<br>
尾递归，进入下一个函数不再需要上一个函数的环境了，得出结果以后直接返回。<br>
尾递归和一般的递归不同在对内存的占用，普通递归创建stack累积而后计算收缩，尾递归只会占用恒量的内存（和迭代一样）。<br>
很多编译器都可以把尾递归优化成迭代，栈内存空间由O(n)削减为O(1)。
而且迭代运算更灵活，容易保存临时结果，对于大型运算任务，可能耗时数日甚至数月，在如此长的时间跨度中，如果出现意外，比如停电，对于迭代算法来说，受到的影响可能很小，因为每步运算后只需保存不太多的结果即可保证下一步运算继续，因此只要定时把计算结果保存到硬盘中，就可以保住运算成果，需要时“存档读档”就可以继续运算；而对于递归算法，这种变故就可能是灾难性的: 递归时的“延迟函数链”通常需要存在内存中（备份到硬盘可是很耗资源的，不现实），一旦有一环丢失运算都难以继续了，只能从头再来。

## 尾递归优化-迭代

```kotlin
// 写法一，二逼青年专用
fun fib(n: Int): Int {
    var a = 1
    var b = 1
    for (i in 1..n) {
        val tempA = b
        val tempB = a + b
        a = tempA
        b = tempB
    }
    return a
}
```

```kotlin
// 写法二，优化掉写法一循环中的临时变量
fun fib(n: Int): Int {
    var a = 1
    var b = 1
    for (i in 1..n) {
        b = a + b
        a = b - a
    }
    return a
}
```

```kotlin
// 写法三，代码更精简，但是运行效率大概率不如写法二，文艺青年专用
fun fib(n: Int): Int {
    var a = 1
    var b = 1
    for (i in 1..n) {
        b += a.apply { a = b }
    }
    return a
}
```

```kotlin
// 写法四，巧用 sequence 生成方法，装*专用
fun fib(n: Int): Int {
    var a = 1
    return generateSequence(1) { (a + it).apply { a = it } }.elementAt(n - 1)
}

// 可以用 BigDecimal 解决整数越界问题。不过我们主要关注的是思路，是否越界不是这里的重点
fun fib(n: Int): BigDecimal {
    var a = BigDecimal.ONE
    return generateSequence(BigDecimal.ONE) { (a + it).apply { a = it } }.elementAt(n - 1)
}
```

### 尾递归为什么可以优化

在回答这个问题之前，我们先谈谈函数栈的作用。
栈是一种数据结构，后进先出。当一个函数被调用的时候，我们会把函数扔进一个叫做“函数栈“的地方，栈的意义其实非常简单——`保持入口环境`。结合一段简单代码：

```kotlin
fun main() {
    //...
    foo()
    //...
    bar()
    //...
    return
}
```

简单在大脑里面模拟一下这个 main 函数调用的整个过程，$ 字符用于表示栈地：

1. 首先我们建立一个函数栈。 $
2. main 函数调用，将 main 函数压进函数栈里面。$ main
3. 做完了一些操作以后，调用 foo 函数，foo 函数入栈。$ main foo
4. foo 函数返回并出栈。$ main
5. 做完一些操作以后，调用 bar 函数，bar 函数入栈。$ main bar
6. bar 函数返回并出栈。$ main
7. 做完余下的操作以后，main函数返回并出栈。$

上面这个过程说明了函数栈的作用是什么？<br>
就是第 4 和第 6 步的作用，让 foo 和 bar 函数执行完了以后能够在回到 main 函数调用 foo 和 bar 原来的地方。这就是栈，这种”后进先出“的数据结构的意义所在。

那么在什么情况下可以把这个入口环境给优化掉？答案不言而喻，当入口环境没意义时。尾递归进入下一个函数不再需要上一个函数的环境了，得出结果以后直接返回，恰好就是这种情况。

## 如何手动优化尾递归

有没有一种通用的方式优化尾递归？答案是肯定的，要不然那么多编译器是怎么做的？<br>
至于如何手动优化尾递归，挖个坑，以后再来埋。

`// todo`

## kotlin tailrec

比起迭代，尾递归的缺点不仅运行效率低下，而且还有 StackOverflow 的风险。kotlin 通过添加 tailrec 标记使尾递归在编译阶段自动优化成迭代，节省了手动优化。

可以对前文的尾递归代码进行测试，设置一个很大的整数入参，添加了 tailrec 标记很快就能计算出结果(忽略整数溢出导致的结果正确性，这里对结论没影响 不是我们关注的重点)，
而注释掉该标记则会抛出 StackOverflowError，即便成功运行也耗时相对严重

```kotlin
/*tailrec*/ fun fib(n: Int, a: Int = 1, b: Int = 1): Int = if (n == 0) a else fib(n - 1, b, a + b)

fun main() {
    val n = 100000
    println("fib($n)=${fib(n)}")
}
```

## kotlin非尾递归添加tailrec会怎么样

kotlin 非尾递归函数强行添加 tailrec 标记会引来 Android Studio 告警:“A function is marked as tail-recursive but no tail calls are found”。
当然，这只是告警，如果强行运行也是可以编译成功的。效果嘛 跟不加 tailrec 标记是一样的，因为编译器能够识别到这不是一个真的尾递归，不会也不可能优化成迭代。

BTW，如果一个真的尾递归函数不添加 tailrec 标记，编译器自然是不会优化成迭代的。从前文可知现在的 kotlin 编译器已经足够聪明，可以自动识别到一个递归是不是真正的尾递归，
不自动优化不是因为做不到，而是刻意为之，为的是避免困惑、亦或是特定需求冲突。编译器不需要绝对智能，需要的是在该智能的时候智能。在这里我们用 tailrec 标记告诉编译器 该你智能的时候啦！

## 所有的递归都可以转化为尾递归吗

参考 [Are there problems that cannot be written using tail recursion?](https://stackoverflow.com/questions/1888702/are-there-problems-that-cannot-be-written-using-tail-recursion){:target="_blank"}

答案是 yes or no，取决于是否引入额外参数。<br>
通过额外参数控制(比如 CPS:[Continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style){:target="_blank"})，
是可以将普通递归转化为尾递归。如果不引入额外参数，保持相同函数签名的情况下，是不可能将所有的递归都转化为尾递归的。

## 如何把非尾递归转化为尾递归

允许引入额外参数、改变函数签名的情况下如何把递归转化为尾递归？

个人观察到的一个现象是，如果递归函数表达式里有多少个递归调用 就需要多少个额外参数控制，简单的理解就是每一个递归调用都需要一个对应的中转参数来保存结果。<br>
未经过数学验证，瞎j8猜的。是否理论上存在一种通用方法将非递归转化为尾递归还有待商榷。<br>
继续挖坑，以后再来埋

`// todo`

## 非尾递归函数的自动优化

其实就尾递归来说，其代码远没有普通递归代码直观优雅、符合直觉，更何况转化成尾递归的过程也是相当考究编程能力。
希望未来的某天编译器更加聪明，能够有一种通用方法，在我们对普通递归添加约定的标记后，自动转化非尾递归为尾递归、并优化成迭代。

以后抠腚这行的门槛儿是不是越来越低了？还是说门槛儿直接就没了？:smile:

<!-- https://stackoverflow.com/questions/33923/what-is-tail-recursion -->
<!-- https://zhuanlan.zhihu.com/p/36587160 -->
<!-- https://blog.csdn.net/u011009574/article/details/84659215 -->
<!-- https://stackoverflow.com/questions/1888702/are-there-problems-that-cannot-be-written-using-tail-recursion -->