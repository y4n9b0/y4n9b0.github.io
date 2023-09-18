---
layout: post
title: 完美数
date: 2023-09-18 22:00:00 +0800
categories: math
tags: math
published: true
---

* content
{:toc}

## 问题

```txt
完美数（Perfect number），是指其因数（不包含本身）的和恰好等于它本身的自然数。
第一个完美数是 6（6=1+2+3），求 1000 以内的所有完美数。
```

## 暴力枚举

```kotlin
import kotlin.math.floor
import kotlin.math.sqrt

fun main() {
    val num = 1000
    val list = mutableListOf<Int>()
    (1 until num).forEach {
        if (isPerfectNumber(it)) {
            list += it
        }
    }
    list.forEach(::println) // 6、28、496
}

fun isPerfectNumber(n: Int): Boolean {
    val upper: Int = floor(sqrt(n.toDouble())).toInt()
    var sum = 0
    var i = 1
    while (i <= upper) {
        val divisor = n.div(i)
        if (i * divisor == n) {
            if (i != n) {
                sum += i
            }
            if (divisor != n && divisor != i) {
                sum += divisor
            }
        }
        i++
    }
    return sum == n
}
```

下面是上述代码里函数 `isPerfectNumber` 的一种优化实现，但是实测貌似两者耗时几乎没差别，胜在代码短吧^_^

```kotlin
fun isPerfectNumber(n: Int): Boolean {
    if (n == 1) return false
    val upper: Int = floor(sqrt(n.toDouble())).toInt()
    var sum = 1
    var i = 2
    while (i < upper) {
        val divisor = n.div(i)
        if (i * divisor == n) {
            sum += i
            sum += divisor
        }
        i++
    }
    if (i * i == n) sum += i
    return sum == n
}
```

## 另辟蹊径

判断一个数是否是完美数，关键在于求其因子和。<br>
在之前的文章[素数筛](https://y4n9b0.github.io/2023/09/04/Prime-sieve/){:target="_blank"}里提到过算术基本定理，任何一个大于 1 的自然数 N，如果 N 不为素数，那么 N 可以唯一分解成有限个素数的乘积。

$$
    n={p_1}^{c_1}·{p_2}^{c_2}·{p_3}^{c_3}···{p_k}^{c_k} \\
    其中 c_i ∈ N^*, p_1、p_2、p_3、···、p_k 均为素数且 p_1 < p_2 < p_3 < ··· < p_k
$$

考虑到素数可表示为其自身的 1 次幂，对于素数 N 上述公式亦满足。

可以得出结论，对于任何一个大于 1 的自然数 N，<br>
其因子(含自身)个数为：

$$
    countOfDivisor=({c_1}+1)·({c_2}+1)···({c_k}+1)
$$

因子(含自身)和为：

$$
    sumOfDivisor={\frac{p_1^{c_1+1}-1}{p_1-1}}·{\frac{p_2^{c_2+1}-1}{p_2-1}}···{\frac{p_k^{c_k+1}-1}{p_k-1}}
$$

证明算不上复杂，简单地说就是素因子组合再加上等比数列求和公式，有空了再来详细补充。 // todo

至此，判断一个数是否是完美数关键在于如何将其分解为算术基本定理表达式，而这将会再次用到素数筛。

```kotlin
fun main() {
    val num = 1000
    val list = mutableListOf<Int>()
    val primes = eulerSieve(num)
    (1 until num).forEach {
        if (isPerfectNumber(it, primes)) {
            list += it
        }
    }
    list.forEach(::println) // 6、28、496
}

fun isPerfectNumber(n: Int, primes: List<Int>): Boolean {
    val map = expressionOfFTA(n, primes)
    val sumOfDivisor = map.toList().fold(1) { acc, (p, c) ->
        acc.times((p.pow(c + 1) - 1).div(p - 1))
    }
    return sumOfDivisor == n.shl(1)
}

// 算术基本定理(Fundamental Theorem of Arithmetic)分解自然数
// 使用 map 保存表达式的素因子及其指数，key - 素因子，value - 指数
fun expressionOfFTA(n: Int, primes: List<Int>): Map<Int, Int> {
    val map = mutableMapOf<Int, Int>()
    var num = n
    var i = 0
    while (i < primes.size && primes[i] <= num) {
        while (num.mod(primes[i]) == 0) {
            map[primes[i]] = (map[primes[i]] ?: 0) + 1
            num = num.div(primes[i])
        }
        i++
    }
    return map
}

// 欧拉筛
fun eulerSieve(n: Int): List<Int> {
    // true - 素数
    val flags = BooleanArray(n) { true }
    val primes = IntArray(n) { 0 }
    var count = 0
    for (i in 2 until n) {
        if (flags[i]) {
            primes[count++] = i
        }
        var j = 0
        while (i * primes[j] < n) {
            flags[i * primes[j]] = false
            if (i.mod(primes[j]) == 0) break
            j++
        }
    }
    return primes.filter { it != 0 }
}

// 垃圾 kotlin，指数运算都没得
fun Int.pow(n: Int): Int = (0 until n).fold(1) { acc, _ ->
    times(acc)
}
```

实测这种方法比暴力枚举慢上很多倍，应该是慢在算术基本定理分解自然数这里（大概率是其中的模运算），后续看看是否有更好的实现方式。不过思路却是极好的。

<!-- https://www.zhihu.com/zvideo/1412093812366368768 -->
<!-- https://brilliant.org/wiki/fundamental-theorem-of-arithmetic/ -->
<!-- https://blog.csdn.net/a675115471/article/details/107553091 -->
