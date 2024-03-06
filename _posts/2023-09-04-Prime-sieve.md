---
layout: post
title: 素数筛
date: 2023-09-04 12:00:00 +0800
categories: math
tags: math
published: true
---

* content
{:toc}

## 问题

```
前 6 个素数分别是 2、3、5、7、11、13，第 6 个素数为 13。
第 10001 个素数是多少？
```

## 素数

1. 素数定义

    素数又被称为质数，是指一个大于1的自然数，除了1和它自身外，不能被其它自然数整除。

2. 算术基本定理

    任何一个大于 1 的自然数 N，如果 N 不为素数，那么 N 可以唯一分解成有限个素数的乘积。<br>
    数学公式记作：

    $$
        n={p_1}^{c_1}·{p_2}^{c_2}·{p_3}^{c_3}···{p_k}^{c_k} \\
        其中 c_i ∈ N^*, p_1、p_2、p_3、···、p_k 均为素数且 p_1 < p_2 < p_3 < ··· < p_k
    $$

    <!-- [算术基本定理的证明](https://zhuanlan.zhihu.com/p/345488859){:target="_blank"} -->

3. 素数无穷多

    欧几里得在《几何原本》中使用经典的反证法：<br>
    假设素数只有有限的 n 个，从小到大依次排成一行，记作 $$p_1,p_2,p_3,···,p_n$$，
    我们构造 $$N=p_1·p_2·p_3···p_n$$
    * 如果 N+1 为素数，则 N+1 必定位于 $$p_1,p_2,p_3,···,p_n$$ 中，但是 N+1 比  $$p_1,p_2,p_3,···,p_n$$ 都大，它不在那些假设的素数集合中，矛盾。
    * 如果 N+1 为合数，由算术基本定理可知，该合数可以分解为几个素数的积，而 N 和 N+1 的最大公约数是 1，所以 N+1 不可能被 $$p_1,p_2,p_3,···,p_n$$ 整除，所以该合数分解得到的素因数肯定不在假设的素数集合中。

    也就是说，假设并不成立，也就证明了素数的个数是无限的。

    证明方式很多，除欧几里得证明外还有 互素证明、欧拉证明、素分解证明，参考
    [素数分布的初等分析方法](https://zhuanlan.zhihu.com/p/640798561){:target="_blank"}

4. 素数定理（PNT：Prime Number Theorem）
    
    定义 π(n) 为 1、2、···、n 中所有素数的个数，则素数定理可以记作：
    
    $$\lim^{}_{n \to \infty}{\frac{π(n)}{\int^{n}_{2}{\frac{1}{ln\,x}dx}}}=1$$

    由于 $$\lim^{}_{n \to \infty}{\frac{\int^{n}_{2}{\frac{1}{ln\,x}dx}}{\frac{n}{ln\,n}}}=1$$ （洛必达法则可证），所以素数定理还有另外一种写法：

    $$\lim^{}_{n \to \infty}{\frac{π(n)ln\,n}{n}}=1$$

    可渐近表示为 $$π(n) ≈ \int^{n}_{2}{\frac{1}{ln\,x}dx} ≈ \frac{n}{ln\,n}$$

    根据这个定理可以推出第 n 个素数的近似估计：$$ p_n ≈ n·ln\,n $$

    <!-- [素数定理的介绍+非常简单的推导](https://zhuanlan.zhihu.com/p/114662915){:target="_blank"} -->

5. [素数计数函数](https://en.wikipedia.org/wiki/Prime-counting_function){:target="_blank"}

    $$
        ln(n·ln(n))−1 < \frac{p_n}{n} < ln(n·ln(n))
    $$

    确定第 n 个素数的上下界对于素数筛估算数组大小非常有用，实际上我们只用到了上界，保证在囊括第 n 个素数的情况下分配足够小的数组以节省内存空间。

## 素数筛

回到开篇问题本身，一种解决思路是我们把自然数从小到大依次遍历，判断是否是素数并对素数计数，直到获取到指定的第 n 个素数。

判断一个数是否是素数：

```kotlin
fun isPrime(n: Long): Boolean {
    val upper = ceil(sqrt(n.toDouble())).toLong()
    (2..upper).forEach {
        if (n.mod(it) == 0L) return false
    }
    return true
}
```
[目前为止判断一个数是否为素数的最快方法是什么](https://www.zhihu.com/question/512360756/answer/2318498459){:target="_blank"}

这种思路有个缺陷，每判断一个自然数都会从头计算（2 开始，$$\sqrt n$$ 结束），计算量很大，特别是 n 较大的时候。而素数筛的思想是先分配一个足够大的数组（保证该数组包含第 n 个素数），每个元素代表其对应顺序的自然数，然后一次性筛出所有素数，再按顺序查找第 n 个素数即可。<br>
有点类似斐波拉切数列，如果使用递归实现每个 fib(n) 会被计算两次，但如果分配一个大小为 n 的数组，再自底向顶计算斐波拉切数列则会避免重复计算。

素数筛又有埃氏筛和欧拉筛（线性筛）

### 埃氏筛

埃氏筛是公元前三世纪的希腊天文学家、数学家和地理学家**埃拉托斯色尼**（Eratosthenes）所提出的一种简单检定素数的算法。要得到自然数 n 以内的全部素数，必须把不大于 $$\sqrt n$$ 的所有素数的倍数剔除，剩下的就是素数。

```kotlin
fun main() {
    println(eratosthenesSieve(6))     // 13
    println(eratosthenesSieve(10001)) // 104743
}

fun eratosthenesSieve(n: Int): Int {
    val upper = primeCountingUpper(n)
    // true - 素数
    val flags = BooleanArray(upper + 1) {
        it != 0 && it != 1
    }
    val floor = floor(sqrt(upper.toDouble())).toInt()
    (2..floor).forEach { i ->
        if (!flags[i]) return@forEach  // 优化运行效率，忽略已经标记过
        (i * 2 until upper step i).forEach {
            flags[it] = false
        }
    }
    var count = 0
    flags.forEachIndexed { index, b ->
        if (b) count++
        if (count == n) return index
    }
    throw IllegalStateException("WTF")
}

fun primeCountingUpper(n: Int): Int {
    return n.times(ln(n.times(ln(n.toDouble())))).toInt()
}
```

### 欧拉筛

埃氏筛在剔除素数的倍数时有一部分合数被不同的素数重复标记，比如 6 就分别被 2 和 3 标记。线性筛可以避免重复操作。

```kotlin
fun main() {
    println(eulerSieve(6))        // 13
    println(eulerSieve(10001))    // 104743
}

fun eulerSieve(n: Int): Int {
    val upper = primeCountingUpper(n)
    // true - 素数
    val flags = BooleanArray(upper + 1) { true }
    val primes = IntArray(upper + 1) { 0 }
    var count = 0
    for (i in 2..upper) {
        if (flags[i]) {
            primes[count++] = i
        }
        var j = 0
        while (i * primes[j] <= upper) {
            flags[i * primes[j]] = false
            if (i.mod(primes[j]) == 0) break
            j++
        }
    }
    return primes[n - 1]
}

fun primeCountingUpper(n: Int): Int {
    return n.times(ln(n.times(ln(n.toDouble())))).toInt()
}
```

<!-- https://pe-cn.github.io/7/ -->
<!-- https://zhuanlan.zhihu.com/p/542171819 -->
<!-- https://zhuanlan.zhihu.com/p/345488859 -->
<!-- https://zhuanlan.zhihu.com/p/509771255 -->
<!-- https://blog.csdn.net/lcx0128/article/details/128408977 -->
<!-- https://baike.baidu.com/item/%E5%AD%AA%E7%94%9F%E7%B4%A0%E6%95%B0%E7%8C%9C%E6%83%B3/4937896 -->
<!-- https://www.zhihu.com/question/512360756/answer/2318498459 -->
<!-- markdown 数学公式符号 https://zhuanlan.zhihu.com/p/357093758 -->
