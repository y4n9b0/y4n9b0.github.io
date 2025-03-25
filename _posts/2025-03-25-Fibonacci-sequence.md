---
layout: post
title: 斐波那契数列
date: 2025-03-25 10:00:00 +0800
categories: algorithm
tags: math
published: true
---

* content
{:toc}

在数学中，斐波那契数列是其中每个元素都是其前面的两个元素之和的序列。斐波那契数列中的数字称为斐波那契数，通常表示为 Fₙ。许多作者从 0 和 1 开始序列，也有些作者从 1 和 1 开始，还有些作者从 1 和 2 开始。这里我们也从 0 和 1 开始。

## 递归

```kotlin
fun fib(n: Int): Int = if (n < 2) n else fib(n - 1) + fib(n - 2)
```

递归的代码简洁明了，但是存在大量重复计算，复杂度指数级别，运行效率极其低下。

## 尾递归

```kotlin
fun fib(n: Int, a: Int = 0, b: Int = 1): Int = if (n == 0) a else fib(n - 1, b, a + b)
```

尾递归一般比较难想到，主要是平时能见到的应用不多，更多的是为后续转化为循环迭代做准备，比如 kotlin 里使用 tailrec 标记尾递归可以被编译器自动优化成循环。

## 迭代-数组

```kotlin
fun fib(n: Int): Int {
    val dp = IntArray(n + 1) { it }
    for (i in 2..n) dp[i] = dp[i - 1] + dp[i - 2]
    return dp.last()
}
```

这大概是大多数人能想到的优化版本。

## 迭代-临时变量

```kotlin
fun fib(n: Int): Int {
    var a = 0
    var b = 1
    repeat(n) { b += a.apply { a = b } }
    return a
}
```

使用临时变量替代数组，降低空间复杂度。

## 通项公式

<!-- $$ F_n = \tfrac{1}{\sqrt{5}}[(\tfrac{1 + \sqrt{5}}{2})^n - (\tfrac{1 - \sqrt{5}}{2})^n] $$ -->

<!-- $$ F_n = \tfrac{1}{\sqrt{5}}\big [(\tfrac{1 + \sqrt{5}}{2})^n - (\tfrac{1 - \sqrt{5}}{2})^n\big ] $$ -->

<!-- $$ F_n = \tfrac{1}{\sqrt{5}}\bigg [(\tfrac{1 + \sqrt{5}}{2})^n - (\tfrac{1 - \sqrt{5}}{2})^n\bigg ] $$ -->

$$ F_n = \tfrac{1}{\sqrt{5}}\Big [(\tfrac{1 + \sqrt{5}}{2})^n - (\tfrac{1 - \sqrt{5}}{2})^n\Big ] $$

<!-- $$ F_n = \tfrac{1}{\sqrt{5}}\Bigg [(\tfrac{1 + \sqrt{5}}{2})^n - (\tfrac{1 - \sqrt{5}}{2})^n\Bigg ] $$ -->

数学理论上通项公式最快，复杂度 O(1)。<br>
但在实际计算机工程应用中，浮点数计算的复杂度为 O(logn)，且浮点数存在运算精度问题。

## 矩阵快速幂

```kotlin
import kotlin.system.measureTimeMillis

class Matrix(vararg elements: IntArray) {
    private val arr: Array<out IntArray> = elements.map { it.copyOf() }.toTypedArray()

    val rows = arr.size
    val cols = if (arr.isNotEmpty()) arr[0].size else 0

    fun set(row: Int, col: Int, value: Int) {
        require(row in 0..<rows && col in 0..<cols) { "Index out of bounds: ($row, $col)" }
        arr[row][col] = value
    }

    fun get(row: Int, col: Int): Int {
        require(row in 0..<rows && col in 0..<cols) { "Index out of bounds: ($row, $col)" }
        return arr[row][col]
    }

    operator fun times(m: Matrix): Matrix {
        require(cols == m.rows) { "Matrix dimensions do not match for multiplication" }
        val product = Matrix(*Array(rows) { IntArray(m.cols) })
        repeat(rows) { r ->
            repeat(m.cols) { c ->
                product.set(r, c, (0..<cols).fold(0) { acc, i -> acc + this.get(r, i) * m.get(i, c) })
            }
        }
        return product
    }
}

fun power(matrix: Matrix, exp: Int): Matrix {
    if (exp == 1) return matrix
    val half = power(matrix, exp / 2)
    return if (exp % 2 == 0) half * half else half * half * matrix
}

fun fib(n: Int): Int {
    if (n < 2) return n
    val init = Matrix(intArrayOf(1), intArrayOf(0))
    val eigen = Matrix(intArrayOf(1, 1), intArrayOf(1, 0))
    // return ((2..<n).fold(eigen) { matrix, _ -> matrix * eigen } * init).get(0, 0)
    return (power(eigen, n - 1) * init).get(0, 0)
}

fun main() {
    val n = 1_000_000
    val time = measureTimeMillis {
        val result = fib(n)
        println("Fibonacci($n) = $result")
    }
    println("Computation time: $time ms")
}
```

欢迎来到线性代数的世界，显然：

$$
  \left\{
    \begin{array}{l}
      F_2 = F_1 + F_0 \\
      F_1 = F_1
    \end{array}
  \right.
$$

使用矩阵表示上述等式关系：

$$
  \bigg [
    \begin{matrix}
      F_2 \\
      F_1
    \end{matrix}
  \bigg ]
  =
  \bigg [
    \begin{matrix}
      1 & 1 \\
      1 & 0
    \end{matrix}
  \bigg ]
  \times
  \bigg [
    \begin{matrix}
      F_1 \\
      F_0
    \end{matrix}
  \bigg ]
$$

那么：

$$
  \bigg [
    \begin{matrix}
      F_3 \\
      F_2
    \end{matrix}
  \bigg ]
  =
  \bigg [
    \begin{matrix}
      1 & 1 \\
      1 & 0
    \end{matrix}
  \bigg ]
  \times
  \bigg [
    \begin{matrix}
      F_2 \\
      F_1
    \end{matrix}
  \bigg ]
  =
  \bigg [
    \begin{matrix}
      1 & 1 \\
      1 & 0
    \end{matrix}
  \bigg ]^2
  \times
  \bigg [
    \begin{matrix}
      F_1 \\
      F_0
    \end{matrix}
  \bigg ]
$$

一般地：

$$
  \bigg [
    \begin{matrix}
      F_n \\
      F_{n-1}
    \end{matrix}
  \bigg ]
  =
  \bigg [
    \begin{matrix}
      1 & 1 \\
      1 & 0
    \end{matrix}
  \bigg ]
  \times
  \bigg [
    \begin{matrix}
      F_{n-1} \\
      F_{n-2}
    \end{matrix}
  \bigg ]
  =
  \bigg [
    \begin{matrix}
      1 & 1 \\
      1 & 0
    \end{matrix}
  \bigg ]^{n-1}
  \times
  \bigg [
    \begin{matrix}
      F_1 \\
      F_0
    \end{matrix}
  \bigg ]
  =
  \bigg [
    \begin{matrix}
      1 & 1 \\
      1 & 0
    \end{matrix}
  \bigg ]^{n-1}
  \times
  \bigg [
    \begin{matrix}
      1 \\
      0
    \end{matrix}
  \bigg ]
$$

斐波那契数列增长很快，上面的 Int 版本很容易整数溢出，即便是 Long 类型也在 n=562 时就溢出了，有需要的可以使用 BigInteger 来实现。

## 卢卡斯数列

占坑

## 齐肯多夫定理

坑 2

## 斐波那契编码

坑 3

<!-- https://www.zhihu.com/question/463190146/answer/3443209466 -->
<!-- https://oi-wiki.org/math/combinatorics/fibonacci/ -->
<!-- https://zh.wikipedia.org/wiki/%E9%BD%8A%E8%82%AF%E5%A4%9A%E5%A4%AB%E5%AE%9A%E7%90%86 -->
<!-- https://y4n9b0.github.io/2021/07/05/Tail-Recursion/ -->
<!-- https://blog.csdn.net/TOMOCAT/article/details/100602408 -->