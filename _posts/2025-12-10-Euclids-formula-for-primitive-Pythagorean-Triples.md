---
layout: post
title: Euclid's formula for primitive Pythagorean Triples
date: 2025-12-10 11:00:00 +0800
categories: algorithm
tags: math
published: true
---

* content
{:toc}

## Count Square Sum Triples

[LeetCode 1925](https://leetcode.com/problems/count-square-sum-triples/description/){:target="_blank"}

A square triple $$(a,b,c)$$ is a triple where a, b, and c are integers and $$a^2 + b^2 = c^2$$.

Given an integer n, return the number of square triples such that $$1 <= a, b, c <= n$$.

**Example 1:**

```
Input: n = 5
Output: 2
Explanation: The square triples are (3,4,5) and (4,3,5).
```

**Example 2:**

```
Input: n = 10
Output: 4
Explanation: The square triples are (3,4,5), (4,3,5), (6,8,10), and (8,6,10).
```

**Constraints:**

* 1 <= n <= 250

**Approach 1: brute force**

```kotlin
class Solution {
    fun countTriples(n: Int): Int {
        var cnt = 0
        for (a in 1..n) {
            for (b in 1..n) {
                val c = sqrt(a * a + b * b + 1f).toInt()
                if (c <= n && c * c == a * a + b * b) cnt++
            }
        }
        return cnt
    }
}
```

**Approach 2: Euclid's formula for primitive Pythagorean Triples**

```kotlin
class Solution {
    fun countTriples(n: Int): Int {
        var cnt = 0
        val nSqrt = sqrt(n.toDouble()).toInt()
        for (u in 2..nSqrt) {
            for (v in 1..<u) {
                if ((u - v).and(1) == 0 || gcd(u, v) != 1) continue
                val c = u * u + v * v
                if (c > n) break
                cnt += (n / c) * 2
            }
        }
        return cnt
    }

    private fun gcd(x: Int, y: Int): Int = if (y == 0) x else gcd(y, x % y)
}
```

## Primitive Pythagorean Triple

一个毕达哥拉斯三元组是满足 a² + b² = c² 的整数解。<br>
如果 a,b,c 两两互质 gcd(a,b,c)=1，则称为 本原毕达哥拉斯三元组（Primitive Pythagorean Triple）。

## Euclid's Formula

如果 (a,b,c) 是本原毕达哥拉斯三元组，则存在整数 u>v>0，且：

* u - v 为奇数
* gcd(u,v)=1 (u,v 互质)

使得：

$$
\begin{aligned}
a &= u^2 - v^2 \\
b &= 2uv \\
c &= u^2 + v^2
\end{aligned}
$$

并且这种表示是 唯一的。

最早可以追溯到 欧几里得在《几何原本》中提出的生成勾股数方法（虽然没有以今天的形式写成 u,v 参数化），但已经包含了公式的雏形。

完整、现代形式的证明不是欧几里得给出的。
欧几里得只给了生成方法，没有严格证明 “所有本原毕达哥拉斯三元组都来自此公式”。

严格意义上的完整现代证明：
*	在 丢番图（Diophantus）的研究中出现
*	在 17–19 世纪由多位数学家（尤其是费马、欧拉、Legendre 等）逐步完善
*	最终成为现代数论中的标准定理

### 为什么这个公式一定满足毕达哥拉斯定理？

代入验证即可：

$$
\begin{aligned}
& \quad\, a^2 + b^2 \\
& = (u^2 - v^2)^2 + (2uv)^2 \\
& = u^4 - 2u^2v^2 + v^4 + 4u^2v^2 \\
& = u^4 + 2u^2v^2 + v^4 \\
& = (u^2 + v^2)^2 \\
& = c^2
\end{aligned}
$$

### 为什么必须满足 gcd(u,v)=1 ？

如果 gcd(u,v)=d>1，那么 a, b, c 都会被 d 的倍数整除，生成的勾股三元组不是本原的

### 为什么必须满足 u–v 为奇数？

枚举反证：

(1) 假设 u,v 同奇<br>
→ u²,v² 同奇<br>
→ u²–v² 和 u²+v² 为偶，但 2uv 也是偶<br>
→ a,b,c 都为偶数 → 不是本原<br>

(2) 假设 u,v 同偶<br>
→ gcd(u,v) 不可能为 1 → 不符合要求

所以为了生成 primitive triple，u,v 必为一奇一偶，等价于其差值为奇数。<br>
由于 u-v 为奇数，所以在 u 确定的情况下 v 的步进一定为 2，**Approach 2** 也可以写成如下方式：

```kotlin
class Solution {
    fun countTriples(n: Int): Int {
        var cnt = 0
        val nSqrt = sqrt(n.toDouble()).toInt()
        for (u in 2..nSqrt) {
            for (v in 1 + u.and(1)..<u step 2) {
                if (gcd(u, v) != 1) continue
                val c = u * u + v * v
                if (c > n) break
                cnt += (n / c) * 2
            }
        }
        return cnt
    }

    private fun gcd(x: Int, y: Int): Int = if (y == 0) x else gcd(y, x % y)
}
```
