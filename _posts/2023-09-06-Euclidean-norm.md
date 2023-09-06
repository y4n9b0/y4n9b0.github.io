---
layout: post
title: 欧几里得范数
date: 2023-09-06 19:30:00 +0800
categories: math
tags: math
published: true
---

* content
{:toc}

## 问题

毕达哥拉斯三元组由三个自然数 a < b < c 组成，并满足
$$a^2 + b^2 = c ^2$$，
例如 $$3^2 + 4^2 = 5^2$$

有且只有一个毕达哥拉斯三元组满足 a + b + c = 1000。求这个三元组的乘积 $$a * b * c$$

## 暴力枚举

为了使题目更具一般性，假设 a + b + c = n，由 a < b < c，可以得知 $$a < \frac{n}{3}$$，$$max(a, \frac{n}{4}) < b < \frac{n}{2}$$

```kotlin
fun main() {
    println("${pythagoreanTriplet(12)}")    // 3 * 4 * 5 = 60
    println("${pythagoreanTriplet(1000)}")  // 200 * 375 * 425 = 31875000
}

fun pythagoreanTriplet(n: Int): Triple<Int, Int, Int> {
    if (n.and(1) == 1) {
        // n 为奇数，不存在毕达哥拉斯三元组（尾数奇偶性可反证）
        return Triple(0, 0, 0)
    }
    val upper = n.div(3)
    (0 until upper).forEach { a ->
        val lower = a.coerceAtLeast(n.div(4)) + 1
        (lower until n.shr(1)).forEach { b ->
            if (a * a + b * b == (n - a - b) * (n - a - b)) {
                return Triple(a, b, n - a - b)
            }
        }
    }
    return Triple(0, 0, 0)
}
```

## 奇技淫巧

对于 0 < p < q，以下三个数可以构成毕达哥拉斯三元组：
$$ q^2 - p^2,\, 2 * q * p,\, q^2 + p^2 $$

令这三个数为 a、b、c(倘若 $$ 2 * q * p > q^2 - p^2 $$，调换 a、b 即可，不影响结论)，则有：
$$ a + b + c = q^2 - p^2 + 2 * q * p + q^2 + p^2 = 2 * q * (q + p) = n $$，即：

$$ q * (q + p) = \frac{n}{2} $$

可以推出 $$ q = \sqrt{\frac{n}{2} - q * p} < \sqrt{\frac{n}{2}} $$； <br>
同时可推出，$$ p = \frac{\frac{n}{2}}{q} - q $$，又因 p < q， 故 $$ \frac{\frac{n}{2}}{q} - q < q $$，
即 $$ q > \frac{\sqrt n}{2} $$。所以：

$$ \frac{\sqrt n}{2} < q < \sqrt{\frac{n}{2}}，\, 且\, q\, 是\, \frac{n}{2} \,的因子 $$

n = 1000 时满足区间 $$ [\frac{\sqrt n}{2}, \sqrt{\frac{n}{2}}] $$ 即 $$ [\sqrt{250}, \sqrt{500}] $$ 的整数只有 16、17、18、19、20、21、22，遍历区间，显然只有 20 是 $$ \frac{n}{2} = 500 $$ 的因子，所以 q = 20，p = 5，故有：
$$ q^2 - p^2 = 20^2 - 5^2 = 375,\, 2 * q * p = 2 * 20 * 5 = 200,\, q^2 + p^2 = 20^2 + 5^2 = 425 $$

按大小顺序分别赋值 $$ a = 200,\, b = 375,\, c = 425 $$，可验证：
$$ a^2 + b^2 = 200^2 + 375^2 = 425^2 = c^2 $$

抠腚如下：
```kotlin
fun main() {
    println("${pythagoreanTriplet(12)}")    // 3 * 4 * 5 = 60
    println("${pythagoreanTriplet(1000)}")  // 200 * 375 * 425 = 31875000
}

fun pythagoreanTriplet(n: Int): Triple<Int, Int, Int> {
    if (n.and(1) == 1) {
        // n 为奇数，不存在毕达哥拉斯三元组（尾数奇偶性可反证）
        return Triple(0, 0, 0)
    }
    val lower = floor(sqrt(n.toDouble()).div(2)).toInt()
    val upper = ceil(sqrt(n.toDouble().div(2))).toInt()
    (lower..upper).forEach { q ->
        if (n.shr(1).mod(q) == 0) {
            val p = n.shr(1).div(q) - q
            if (p < q) {
                return Triple(q * q - p * p, 2 * p * q, q * q + p * p)
            }
        }
    }
    return Triple(0, 0, 0)
}
```

## 欧几里得范数

to be continue...

<!-- https://pe-cn.github.io/9/ -->
<!-- https://pe.metaquant.org/pe009.html -->
<!-- https://jingyan.baidu.com/article/597a064332b227702a52436a.html -->
<!-- https://ww2.mathworks.cn/help/matlab/ref/norm.html -->
