---
layout: post
title: 最小公倍数
date: 2023-09-03 15:00:00 +0800
categories: math
tags: math
published: true
---

* content
{:toc}

## 问题

```txt
2520 是最小的能够被 1 到 10 整除的整数。
最小的能够被 1 到 20 整除的整数是多少？
```

这个问题等同于依次求解最小公倍数(least common multiple)，先求 1 和 2 的最小公倍数，再求该公倍数与 3 的最小公倍数，直到循环结束。

## 较大数扩大找倍数法

假设 m = 6, n = 15<br>
取较大数 15 依次乘以 1,2,3,4···<br>
直到某个乘积（15 * 2 = 30）为较小数 6 的倍数时，该乘积即最小公倍数

```kotlin
fun main() {
    assert(solution(10) == 2520L)
    assert(solution(20) == 232792560L)
}

fun solution(n: Int): Long {
    return List(n) { it + 1L }.reduce { sum, num ->
        lcm(sum, num)
    }
}

fun lcm(m: Long, n: Long): Long {
    val min = min(m, n)
    val max = max(m, n)
    var i = 1
    while (max.times(i).mod(min) != 0L) {
        i++
    }
    return max * i
}
```

## 欧几里得算法求最大公约数

对于任意整数 m 和 n，假设其最大公约数和最小公倍数分别为 p 和 q，则有 m * n = p * q<br>
所以求最小公倍数可以转换为求最大公约数(greatest common divisor)。

一个整数 n 能被整数 d 整除当且仅当存在整数 k 使得 n = k * d，记做 `d|n`<br>
如果 d 又同时能整除另一个数 m，则我们称 d 为 m 和 n 的公约数。由于 m 和 n 只能有有限个约数，
所以如果把同时满足 `d|m` 和 `d|n` 的所有 d 聚成集合 S，则 S 必为有限集。
而这个集合的最大值便被称为 m、n 的最大公约数。

假设整数 d 满足 `d|m` 和 `d|n`（即 d 为 m、n 的公约数），由于整除的传递性，我们可以发现对于任何的整数 k 均有 `d|(m - k * n)`，所以有：

```
gcd(m, n) = gcd(m - k * n, n)
```

所以我们可以利用取模运算在不改变函数值的情况下降低自变量的大小：

```
gcd(m, n) = gcd(m % n, n)
```

由于 0 ≤ (m % n) < n，所以接下来我们可以再对 n 取 (m % n) 的模。如此往返迭代最终一定会有一个值转化成 0，即：

```
gcd(m, n) = gcd(r, 0) = r
```

这种计算方法被称为**欧几里得算法**（Euclidean algorithm），也称为辗转相除法：<br>

```kotlin
fun main() {
    assert(solution(10) == 2520L)
    assert(solution(20) == 232792560L)
}

fun solution(n: Int): Long {
    return (1L .. n).reduce { sum, num ->
        lcm(sum, num)
    }
}

fun lcm(m: Long, n: Long): Long {
    return m.times(n).div(gcd(m, n))
}

fun gcd(m: Long, n: Long): Long {
    return if (n == 0L) m else gcd(n, m.mod(n))
}
```

## 欧几里得算法复杂度

现在假设由于递归的终止条件是第二个参数为 0，所以我们只需要分析 n 在每次迭代后下降的速度就可以得到算法的时间复杂度。
当 m ≥ n 时，我们有 m = k * n + r，其中 k ≥ 1 且 0 ≤ r < n，因此有 m > (q + 1) * r ≥ 2 * r，即 (m % n) < (m / 2)。
因此每做两次递归，gcd 的两个输入变量至少也会减半。而只要 m、n 的其中一者先下降到零就会终止。综上所述欧几里得算法的时间复杂度为 O{log[min(m,n)]}

## 更相减损术

秦九韶在《九章算术》中的更相减损术，原理也是类似，不过他是相互做减取差，迭代速度会比欧几里得取模慢。
但是，如果改进将其配合上位移操作，迭代速度会比欧几里得算法更快：

```kotlin
fun main() {
    assert(solution(10) == 2520L)
    assert(solution(20) == 232792560L)
}

fun solution(n: Int): Long {
    return (1L .. n).reduce { sum, num ->
        lcm(sum, num)
    }
}

fun lcm(m: Long, n: Long): Long {
    return m.times(n).div(gcd(m, n))
}

fun gcd(m: Long, n: Long): Long {
    if (m == n) return m
    return if ((m.and(n).and(1L)) == 0L) {
        // 此条件包含三种情况，巧用位运算书写成统一的代码
        // m 和 n 均为偶数，则 gcd(m, n) = 2 * gcd(m/2, n/2)
        // m 为偶数，n 为奇数，则 gcd(m, n) = gcd(m/2, n)
        // m 为奇数，n 为偶数，则 gcd(m, n) = gcd(m, n/2)
        val shiftRightM = (m + 1).and(1).toInt()
        val shiftRightN = (n + 1).and(1).toInt()
        gcd(m.shr(shiftRightM), n.shr(shiftRightN)).shl(shiftRightM.and(shiftRightN))
    } else {
        // m 和 n 均为奇数，相减将其变为一个偶数和一个奇数的情况
        gcd(abs(m - n), min(m, n))
    }
}
```

<!-- https://pe-cn.github.io/5/ -->
<!-- https://zhuanlan.zhihu.com/p/374026729 -->
<!-- https://blog.csdn.net/weixin_38426554/article/details/95787416 -->
<!-- https://leetcode.com/problems/find-greatest-common-divisor-of-array/ -->