---
layout: post
title: 最长公共子串
date: 2024-06-22 16:00:00 +0800
categories: algorithm
tags: algorithm
published: true
---

* content
{:toc}

## LCS

[LeetCode 1143](https://leetcode.com/problems/longest-common-subsequence/){:target="_blank"}

Given two strings text1 and text2, return the length of their longest common subsequence. If there is no common subsequence, return 0.

A subsequence of a string is a new string generated from the original string with some characters (can be none) deleted without changing the relative order of the remaining characters.

* For example, "ace" is a subsequence of "abcde".

A common subsequence of two strings is a subsequence that is common to both strings.

**动态规划解法[^1]**
```kotlin
fun longestCommonSubsequence(text1: String, text2: String): Int {
    val length1 = text1.length
    val length2 = text2.length
    val chars1 = text1.toCharArray()
    val chars2 = text2.toCharArray()
    //  dp[i][j] 表示 text1 的前 i 个元素与 text2 的前 j 个元素的最长公共子序列的长度
    val dp = Array(length1 + 1) { IntArray(length2 + 1) }
    chars1.forEachIndexed { i, c1 ->
        chars2.forEachIndexed { j, c2 ->
            dp[i + 1][j + 1] = if (c1 == c2) dp[i][j] + 1 else dp[i][j + 1].coerceAtLeast(dp[i + 1][j])
        }
    }
    return dp[length1][length2]
}
```

**位运算解法[^2]**
```kotlin
@OptIn(ExperimentalUnsignedTypes::class)
fun longestCommonSubsequence(text1: String, text2: String): Int {
    var A = text1
    var B = text2
    // 不交换也是 ok 的，交换可以减少循环、提高运行效率
    if (A.length < B.length) A = B.also { B = A }
    val m = A.length
    val n = B.length
    if (m == 0 || n == 0) return 0

    val w = (m + 63) shr 6
    val S = Array(256) { ULongArray(w) }
    var set = 1UL
    var j = 0
    for (i in 0 until m) {
        S[A[i].code][j] = S[A[i].code][j] or set
        set = set shl 1
        if (set == 0UL) {
            set = 1UL
            j++
        }
    }

    val L = ULongArray(w)
    for (i in 0 until n) {
        var b1 = 1UL
        var b2 = 0UL
        for (j in 0 until w) {
            val X = L[j] or S[B[i].code][j]
            val c = L[j] shr 63
            val V = X - ((L[j] shl 1) or (b1 + b2))
            b1 = c
            b2 = if (V > X) 1UL else 0UL
            L[j] = X and V.inv() // 另一种方式 L[j] = V.xor(X).and(X)
        }
    }
    return L.fold(0) { acc, i -> acc + i.countOneBits() }
}
```

## LCS of three strings

Given two strings text1 text2 and text3, return the length of their longest common subsequence. If there is no common subsequence, return 0.

```kotlin
fun longestCommonSubsequence(text1: String, text2: String, text3: String): Int {
    val length1 = text1.length
    val length2 = text2.length
    val length3 = text3.length
    val chars1 = text1.toCharArray()
    val chars2 = text2.toCharArray()
    val chars3 = text2.toCharArray()
    //  dp[i][j][k] 表示 text1 的前 i 个元素、 text2 的前 j 个元素与 text3 的前 k 个元素的最长公共子序列的长度
    val dp = Array(length1 + 1) { Array(length2 + 1) { IntArray(length3 + 1) } }
    chars1.forEachIndexed { i, c1 ->
        chars2.forEachIndexed { j, c2 ->
            chars3.forEachIndexed { k, c3 ->
                dp[i + 1][j + 1][k + 1] = if (c1 == c2 && c2 == c3) {
                    dp[i][j][k] + 1
                } else {
                    maxOf(dp[i][j + 1][k + 1], dp[i + 1][j][k + 1], dp[i + 1][j + 1][k])
                }
            }
        }
    }
    return dp[length1][length2][length3]
}
```

**Footnote**

[^1]: [https://oi-wiki.org/dp/basic/#最长公共子序列](https://oi-wiki.org/dp/basic/#%E6%9C%80%E9%95%BF%E5%85%AC%E5%85%B1%E5%AD%90%E5%BA%8F%E5%88%97){:target="_blank"}

[^2]: [A Bit-String Longest-Common-Subsequence Algorithm](https://users.monash.edu/~lloyd/tildeStrings/Alignment/86.IPL.html){:target="_blank"}

<!-- https://codeforces.com/blog/entry/127488 -->
<!-- https://github.com/ShahjalalShohag/code-library/blob/main/Strings/Bit%20LCS.cpp -->
<!-- https://wenku.baidu.com/view/ed99e4f77c1cfad6195fa776.html?_wkts_=1719027557244&needWelcomeRecommand=1 -->
<!-- https://users.monash.edu/~lloyd/tildeStrings/Alignment/86.IPL/ -->
<!-- https://www.cnblogs.com/-Wallace-/p/bit-lcs.html -->
<!-- https://www.geeksforgeeks.org/lcs-longest-common-subsequence-three-strings/ -->