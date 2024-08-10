---
layout: post
title: O(log(n!)) ≟ O(nlogn)
date: 2024-08-10 09:30:00 +0800
categories: algorithm
tags: complexity
published: true
---

* content
{:toc}

思考一个问题：时间复杂度 $$O(\log(n!))$$ 与 $$O(n\log{n})$$ 相比谁更低一些？<br>
时间到，咱们先看两个函数：

```kotlin
fun foo(nums: IntArray) {
    for (i in nums.indices) {
        nums.binarySearch(element = 9527, fromIndex = 0, toIndex = nums.size)
        ···
    }
}

fun bar(nums: IntArray) {
    for (i in nums.indices) {
        nums.binarySearch(element = 9527, fromIndex = 0, toIndex = i)
        ···
    }
}
```

如上两个函数 foo 和 bar，假设整型数组 nums 长度为 n，函数 foo 对数组 nums 遍历 n 次，每次都作长度为 n 的二分查找，可以得知其复杂度为 $$O(n\log{n})$$。

函数 bar 同样对数组 nums 遍历 n 次，唯一不同的是每次二分查找的范围长度是递增的：

$$
\begin{aligned}
& \quad\, \log{1}+\log{2}+\log{3}+···+\log{n}\\
& = \log(1*2*3*···*n)\\
& = \log(n!)
\end{aligned}
$$

同样遍历 n 次，由于函数 bar 每次二分查找范围小于等于函数 foo 的二分查找范围，显然函数 bar 会比函数 foo 更快。
所以可以得出结论 $$O(\log(n!))$$ 比 $$O(n\log{n})$$ 复杂度更低?

Unfortunately，这个结论是错误的，事实上 $$O(\log(n!)) = O(n\log{n})$$。下面我们用夹逼定理证明：

$$
\begin{aligned}
& \: n! = n(n-1)(n-2)⋯(2)(1)\\
& \quad \geq n(n-1)(n-2)⋯(\frac{n}{2})\\
& \quad \geq (\frac{n}{2})(\frac{n}{2})(\frac{n}{2})⋯(\frac{n}{2})\\
& \quad = (\frac{n}{2})^{\frac{n}{2}}
\end{aligned}
$$

两边同时取对数可得

$$ \log(n!) \geq {\frac{n}{2}}\log{\frac{n}{2}} = {\frac{n}{2}}(\log{n} - 1) $$

$$ ⇒ O(\log(n!)) \geq O({\frac{n}{2}}(\log{n} - 1)) = O(n\log{n}) $$

又

$$ \log(n!) \leq \log(n^n) = n\log{n} $$

$$ ⇒ O(\log(n!)) \leq O(n\log{n}) $$

故

$$ O(\log(n!)) = O(n\log{n}) $$

之所以结论有悖直觉，跟时间复杂度的定义有关，时间复杂度比较的是 n→∞ 复杂度函数的数量级是否相同。

<!-- https://www.quora.com/Why-does-O-nlogn-O-log-n -->