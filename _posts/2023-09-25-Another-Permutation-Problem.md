---
layout: post
title: Another Permutation Problem
date: 2023-09-25 11:00:00 +0800
categories: math
tags: math
published: true
---

* content
{:toc}

## 问题

[CodeForces 1859 C](https://codeforces.com/problemset/problem/1859/C){:target="_blank"}

Andrey is just starting to come up with problems, and it's difficult for him. That's why he came up with a strange problem about permutations and asks you to solve it. Can you do it?

Let's call the cost of a permutation 𝑝 of length 𝑛 the value of the expression: 

$$(\sum_{i=1}^{n}{p_i \cdot i})-(max_{j=1}^n{p_j \cdot j})$$

Find the maximum cost among all permutations of length 𝑛.

A permutation of length 𝑛 is an array consisting of 𝑛 distinct integers from 1 to 𝑛 in arbitrary order. For example, [2,3,1,5,4] is a permutation, but [1,2,2] is not a permutation (2 appears twice in the array), and [1,3,4] is also not a permutation (𝑛=3 but there is 4 in the array).

**Input**<br>
Each test consists of multiple test cases. The first line contains a single integer 𝑡 (1≤𝑡≤30) — the number of test cases. The description of the test cases follows.

The only line of each test case contains a single integer 𝑛 (2≤𝑛≤250) — the length of the permutation.

It is guaranteed that the sum of 𝑛 over all test cases does not exceed 500.

**Output**<br>
For each test case, output a single integer — the maximum cost among all permutations of length 𝑛.

**Example**

|   input   |   output  |
|   :-      |   :-      |
|   5       |           |
|   2       |   2       |
|   4       |   17      |
|   3       |   7       |
|   10      |   303     |
|   20      |   2529    |

<!-- 
|   input   |
|   :-      |
|   5       |
|   2       |
|   4       |
|   3       |
|   10      |
|   20      |

|   output  |
|   :-      |
|   2       |
|   17      |
|   7       |
|   303     |
|   2529    | -->

**Note**<br>
In the first test case, the permutation with the maximum cost is [2,1]. The cost is equal to 2⋅1+1⋅2−max(2⋅1,1⋅2)=2+2−2=2.

In the second test case, the permutation with the maximum cost is [1,2,4,3]. The cost is equal to 1⋅1+2⋅2+4⋅3+3⋅4−4⋅3=17.

**中文版**<br>
给定 n (2≤𝑛≤250)，定义排列的成本为：

$$(\sum_{i=1}^{n}{p_i \cdot i})-(max_{j=1}^n{p_j \cdot j})$$

求长度为 n 的排列的成本的最大值。

## 思路

先打个表，观察现象、研究性质、发现规律（英文说法 [observation](https://codeforces.com/blog/entry/66715?#comment-507869){:target="_blank"}）：

|   n   |   最大成本 |   排列                     |
|   :-  |   :-      |   :-                      |  
|   2   |   2       |   2 1                     |  
|   3   |   7       |   1 3 2                   |
|   4   |   17      |   1 2 4 3                 |
|   5   |   35      |   1 2 5 4 3               |
|   6   |   62      |   1 2 3 6 5 4             |
|   7   |   100     |   1 2 3 4 7 6 5           |
|   8   |   152     |   1 2 3 4 8 7 6 5         |
|   9   |   219     |   1 2 3 4 5 9 8 7 6       |  
|   10  |   303     |   1 2 3 4 5 6 10 9 8 7    |

从前几项我们可以看出，最大成本的排列呈现一个规律：反转升序排列的最后几项即可，且反转的长度为 $$\sqrt{2n}$$，实际上反转长度 $$\sqrt{2n+1}$$ 也是正确的，也就是说对于某些特定的 n (满足 $$\sqrt{2n} \not= \sqrt{2n+1}$$)有两种最大成本的排列方式。

找到规律后就简单多了，把排列分为两部分：升序排列的前半部分、反转后降序的后半部分，令 k = $$\sqrt{2n}$$ 为反转长度，可列出最大成本公式：

$$\sum_{i=1}^{n-k}{i^2} + \sum_{i=n-k+1}^{n}{i \cdot (2n-k+1-i)} - max_{j=1}^n{p_j \cdot j}\tag{1} $$

先求出公式前两项的和：

$$
\begin{aligned}
& \quad\,\sum_{i=1}^{n-k}{i^2} + \sum_{i=n-k+1}^{n}{i \cdot (2n-k+1-i)}\\
&=\sum_{i=1}^{n-k}{i^2} + \sum_{i=n-k+1}^{n}{[(2n-k+1) \cdot i-i^2]}\\
&=\sum_{i=1}^{n-k}{i^2} + (2n-k+1)\sum_{i=n-k+1}^{n}{i} - \sum_{i=n-k+1}^{n}{i^2}\\
&=\sum_{i=1}^{n-k}{i^2} + (2n-k+1)(\sum_{i=1}^{n}{i}-\sum_{i=1}^{n-k}{i}) - (\sum_{i=1}^{n}{i^2}-\sum_{i=1}^{n-k}{i^2})\\
&=2\sum_{i=1}^{n-k}{i^2} - \sum_{i=1}^{n}{i^2}  + (2n-k+1)(\sum_{i=1}^{n}{i}-\sum_{i=1}^{n-k}{i})\\
&=2\frac{(n-k)[(n-k)+1][2(n-k)+1]}{6} - \frac{n(n+1)(2n+1)}{6} + (2n-k+1)(\frac{n(n+1)}{2}-\frac{(n-k)(n-k+1)}{2})\\
&=\frac{2n^3+3n^2+n-k^3+k}{6}
\end{aligned}
$$

因此，最大成本公式(1)可写为：

$$\frac{2n^3+3n^2+n-k^3+k}{6} - max_{j=1}^n{p_j \cdot j}\tag{2}$$

接下来处理 $$max_{j=1}^n{p_j \cdot j}$$。
显然，当反转长度 k 为偶数时，$$max_{j=1}^n{p_j \cdot j}$$ 等于反转部分最中间两项的乘积，反转长度 k 为奇数时则等于反转部分最中间一项的平方，即：

$$
max_{j=1}^n{p_j \cdot j}=
\begin{cases}
[(n-k+1)+\frac{k}{2}] \cdot (n-\frac{k}{2}) = \dfrac{(2n-k+1)^2-1}{4}, &if\ k\ is\ even\\[1.2ex]
[(n-k+1)+\frac{k-1}{2}] \cdot (n-\frac{k-1}{2}) = \dfrac{(2n-k+1)^2}{4}, &if\ k\ is\ odd
\end{cases}
$$

将其代入到最大成本公式(2)中：

$$
\begin{cases}
\dfrac{2n^3+3n^2+n-k^3+k}{6} - \dfrac{(2n-k+1)^2-1}{4}, &if\ k\ is\ even\\[1.2ex]
\dfrac{2n^3+3n^2+n-k^3+k}{6} - \dfrac{(2n-k+1)^2}{4}, &if\ k\ is\ odd
\end{cases}\tag{3}
$$

虽然已经可以得出精确的结果，但公式(3)需要区分 k 的奇偶性。让我们试试如何把公式统一起来，鉴于数学层面已经如上精确定义，我们无法再改变了，这里需要用到计算机里整数相除再取整这种 hack 技巧。

统一公式(3)有两种方法，先看第一种：k 为 偶数时，由于 $$max_{j=1}^n{p_j \cdot j}$$ 的分子 $$(2n-k+1)^2-1$$ 能被分母 4 整除，所以把它加上 1 除以 4 后再取整在计算机里依然等于原结果：

$$
\begin{aligned}
&\quad\, \dfrac{(2n-k+1)^2-1}{4}\\
&=\dfrac{(2n-k+1)^2-1+1}{4} - \dfrac{1}{4}\\
&=\dfrac{(2n-k+1)^2}{4} - 0, &//\ 计算机整数相除再取整\\
&=\dfrac{(2n-k+1)^2}{4}
\end{aligned}
$$

因此，最大成本公式(3)可以统一为：

$$\frac{2n^3+3n^2+n-k^3+k}{6} - \frac{(2n-k+1)^2}{4}\tag{4}$$

需要注意的是，由于利用了计算机整数相除后取整(向下)的规则，公式(4)不能直接进行数学上的通分。直接通分后必须向上取整才行（因为第 2 项是相减，需要反过来向上取整）。

接下来实现第二种统一公式，首先把最大成本公式(3)进行通分：

$$
\begin{cases}
    \dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn}{12}, &if\ k\ is\ even\\[1.2ex]
    \dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn-3}{12}, &if\ k\ is\ odd
\end{cases}\tag{5}
$$

我们知道公式(5)中无论 k 是奇数还是偶数，其分子都能被分母 12 整除（因为通分前两个分数的分子都能分别被其分母整除），所以给 k 为奇数的分子 $$4n^3-6n^2-10n-2k^3-3k^2+8k+12kn-3$$ 加上 3 除以 12 后再取整在计算机里依然等于原结果。于是通分后的最大成本公式(5)可以统一为其偶数形式：

$$\dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn}{12}\tag{6}$$

更一般地，由于公式(5)分子能被分母整除，我们可以给其分子分别加上一个常数 $$c∈[0,11]$$ 使其除以分母 12 取整后结果保持不变：

$$
\begin{aligned}
&\quad\, \begin{cases}
    \dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn}{12}, &if\ k\ is\ even\\[1.2ex]
    \dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn-3}{12}, &if\ k\ is\ odd
\end{cases}\\
&=\begin{cases}
    \dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn+c}{12}, \qquad c∈[0,11] &if\ k\ is\ even\\[1.2ex]
    \dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn+c-3}{12},\ \ c∈[0,11] &if\ k\ is\ odd
\end{cases}\\
&=\begin{cases}
    \dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn+c}{12}, \qquad c∈[0,11] &if\ k\ is\ even\\[1.2ex]
    \dfrac{4n^3-6n^2-10n-2k^3-3k^2+8k+12kn+c}{12}, \qquad c∈[-3,8] &if\ k\ is\ odd
\end{cases}\\
&=\frac{4n^3 - 6n^2 - 10n - 2k^3 - 3k^2 + 8k + 12kn + c}{12},\ c∈[0,8]
\end{aligned}\tag{7}
$$

当然，公式(7)取 c=0 干掉常数项最简单，此时即为公式(6)。

版本一：
```kotlin
import kotlin.math.sqrt

fun main() {
    val count = readLine()!!.toInt()
    repeat(count) {
        val n = readLine()!!.toInt()
        println(maxCost(n))
    }
}

fun maxCost(n: Int): Int {
    val k = sqrt((2 * n).toDouble()).toInt()
    val sum = (2 * n * n * n + 3 * n * n + n - k * k * k + k) / 6
    val temp = 2 * n - k + 1
    val mid1 = temp / 2
    val mid2 = temp - mid1
    val max = mid1 * mid2
    return sum - max
}
```

版本二：
```kotlin
import kotlin.math.sqrt

fun main() {
    val count = readLine()!!.toInt()
    repeat(count) {
        val n = readLine()!!.toInt()
        println(maxCost(n))
    }
}

fun maxCost(n: Int): Int {
    val k = sqrt((2 * n).toDouble()).toInt()
    val sum = (2 * n * n * n + 3 * n * n + n - k * k * k + k) / 6
    val max = (2 * n - k + 1) * (2 * n - k + 1) / 4
    return sum - max
}
```

版本三：
```kotlin
import kotlin.math.ceil
import kotlin.math.sqrt

fun main() {
    val count = readLine()!!.toInt()
    repeat(count) {
        val n = readLine()!!.toInt()
        println(maxCost(n))
    }
}

fun maxCost(n: Int): Int {
    val k = sqrt((2 * n).toDouble()).toInt()
    return ceil((4 * n * n * n - 6 * n * n - 10 * n - 2 * k * k * k - 3 * k * k + 8 * k + 12 * k * n - 3) / 12.toDouble()).toInt()
}
```

版本四：
```kotlin
import kotlin.math.sqrt

fun main() {
    val count = readLine()!!.toInt()
    repeat(count) {
        val n = readLine()!!.toInt()
        println(maxCost(n))
    }
}

fun maxCost(n: Int): Int {
    val k = sqrt((2 * n).toDouble()).toInt()
    val kIsEven = k.and(1) == 0
    return (4 * n * n * n - 6 * n * n - 10 * n - 2 * k * k * k - 3 * k * k + 8 * k + 12 * k * n - if (kIsEven) 0 else 3) / 12
}
```

版本五：
```kotlin
import kotlin.math.sqrt

fun main() {
    val count = readLine()!!.toInt()
    repeat(count) {
        val n = readLine()!!.toInt()
        println(maxCost(n))
    }
}

fun maxCost(n: Int): Int {
    val k = sqrt((2 * n).toDouble()).toInt()
    return (4 * n * n * n - 6 * n * n - 10 * n - 2 * k * k * k - 3 * k * k + 8 * k + 12 * k * n) / 12
}
```

**Note**
无论是哪个版本的答案，k 取 $$\sqrt{2n}$$ 或者 $$\sqrt{2n+1}$$ 都是正确的（前面说过哦）。

这道题作者 [induk_v_tsiane](https://codeforces.com/blog/entry/119287){:target="_blank"} 的本意是希望使用贪心算法之类来解决，却没想到一个公式推导就轻松搞定。数学之奇妙，直接惊掉了我的下巴。

**一些辅助公式**<br>

* 自然数和

  $$1+2+ \cdots +n=\frac{n(n+1)}{2}$$

* 自然数平方和

  $$1^2+2^2+ \cdots +n^2=\frac{n(n+1)(2n+1)}{6}$$

* $$n \cdot 1+(n-1) \cdot 2+ \cdots +2\cdot(n-1)+1 \cdot n=\frac{n(n+1)(n+2)}{6}$$

<!-- https://codeforces.com/problemset/problem/1859/C -->
<!-- https://codeforces.com/blog/entry/119287 -->
<!-- https://zhuanlan.zhihu.com/p/649683190 -->
<!-- http://gohom.win/2015/11/06/Kramdown-note/ -->
<!-- https://codeforces.com/blog/entry/16599 -->
<!-- https://codeforces.com/blog/entry/66909 -->
<!-- https://codeforces.com/blog/entry/66715?#comment-507869 -->
<!-- https://blog.csdn.net/weixin_46233323/article/details/104538187 -->
<!-- https://www.zhihu.com/question/36029266/answer/1687590679 -->
<!-- https://www.zhihu.com/question/547684104/answer/2636953324 -->
<!-- Markdown数学公式语法 -->
<!-- https://www.jianshu.com/p/383e8149136c -->
<!-- LaTeX中的\cfrac和\dfrac有什么区别？ -->
<!-- https://www.zhihu.com/question/457761901/answer/2017930888 -->
