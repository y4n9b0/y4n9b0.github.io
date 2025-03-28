---
layout: post
title: 字符串匹配
date: 2025-03-28 10:00:00 +0800
categories: algorithm
tags: algorithm
published: true
---

* content
{:toc}

在浏览网页或者文本时，我们常常通过搜索关键字去定位到想要的信息部分，可否想过计算机是如何在众多的字符串中快速匹配到我们想要的关键字？

本文简单聊一聊一些常见的字符串匹配算法。

## 定义

字符串匹配又称模式匹配（pattern matching）。该问题可以概括为「给定字符串 S 和 T，在主串 S 中寻找子串 T」。字符 T 称为模式串 (pattern)。

## 类型

* 单串匹配：给定一个模式串和一个待匹配串，找出前者在后者中的所有位置。
* 多串匹配：给定多个模式串和一个待匹配串，找出这些模式串在后者中的所有位置。
    * 出现多个待匹配串时，将它们直接连起来便可作为一个待匹配串处理。
    * 可以直接当做单串匹配，但是效率不够高。
* 其他类型：例如匹配一个串的任意后缀，匹配多个串的任意后缀···

## Brute Force（暴力检索）

Brute Force 的基本思想是从主串 S 的第一个字符开始和模式串 T 的第一个字符进行比较，若相等，则继续比较二者的后续字符；否则，模式串 T 回退到第一个字符，重新和主串 S 的第二个字符进行比较。如此往复，直到 S 或 T 中所有字符比较完毕。

```kotlin
fun bruteForceSearch(text: String, pattern: String): Int {
    val n = text.length
    val m = pattern.length

    for (i in 0..(n - m)) {
        if (text.substring(i, i + m) == pattern) return i
    }
    return -1
}
```

## Robin-Karp（哈希检索）

Robin-Karp 算法是 1987 年由 Richard M. Karp 和 Michael O. Rabin 发明。该算法是对 Brute Force 算法的一个改进：在 Brute Force 算法中，每一个字符都需要进行比较，并且当我们发现首字符匹配时仍然需要比较剩余的所有字符。而在 Robin-Karp 算法中，就尝试只进行一次比较来判定两者是否相等。 

其具体做法是：先求出模式串的哈希值，再求出文本串每个长度为模式串长度的子串的哈希值，分别与模式串的哈希值比较。如果哈希值不同，则两者必定不匹配，如果相同，由于哈希冲突存在，还需要按照 Brute Force 算法再次判定。 

```kotlin
fun rabinKarpWithHashCode(text: String, pattern: String): Int {
    val m = pattern.length
    val n = text.length

    val patternHash = pattern.hashCode()
    for (i in 0..(n - m)) {
        val subText = text.substring(i, i + m)
        if (subText.hashCode() == patternHash && subText == pattern) return i
    }
    return -1
}
```

Robin-Karp 算法进一步优化，可以用 **滚动哈希**（Rolling Hash）而不是 hashCode()，这样可以避免重复计算哈希值，复杂度从 O(m * n) + O(m) 降到 O(n) + O(m)，需要手动维护一个哈希计算，例如：

* 使用 Horner’s method 计算多项式哈希。
* 用一个滑动窗口更新哈希值（避免重复计算）。

```kotlin
fun rabinKarpWithRollingHash(text: String, pattern: String): Int {
    val m = pattern.length
    val n = text.length
    if (m > n) return -1

    val base = 31 // 经验优化的质数
    val mod = 1_000_000_007 // 防止哈希溢出

    // mod * base + Char.MAX_VALUE.code > Int.MAX_VALUE，使用 Long 类型防止溢出
    var patternHash = 0L // 计算 pattern 的 hash 值
    var textHash = 0L
    var highestPower = 1L // 最高位的权重 (base^(m-1))

    for (i in 0 until m) {
        patternHash = (patternHash * base + pattern[i].code) % mod
        textHash = (textHash * base + text[i].code) % mod
        if (i < m - 1) highestPower = (highestPower * base) % mod
    }

    for (i in 0..(n - m)) {
        // 哈希值匹配，进行字符串逐字符比对
        if (patternHash == textHash && text.substring(i, i + m) == pattern) return i
        // 计算下一个窗口的哈希值
        if (i < n - m) {
            textHash = (textHash - text[i].code * highestPower % mod + mod) % mod // 移除最高位
            textHash = (textHash * base + text[i + m].code) % mod // 加入新字符
        }
    }

    return -1
}
```

31 作为 base 是一个很有意思的素数：$$31 = 32 - 1 = 2^5 - 1$$。一个数乘上 31 等于将其左移 5 位再减上自身，运算速度理论上比直接作乘法要快，当然这里还得考虑左移 5 位不会溢出。

```kotlin
fun rabinKarpWithRollingHash(text: String, pattern: String): Int {
    val m = pattern.length
    val n = text.length
    if (m > n) return -1

    val base = 31 // a * 31 == a.shl(5) - a
    val mod = 1_000_000_007 // 防止哈希溢出

    // mod * base + Char.MAX_VALUE.code > Int.MAX_VALUE，使用 Long 类型防止溢出
    var patternHash = 0L // 计算 pattern 的 hash 值
    var textHash = 0L
    var highestPower = 1L // 最高位的权重 (base^(m-1))

    for (i in 0 until m) {
        patternHash = (patternHash.shl(5) - patternHash + pattern[i].code) % mod
        textHash = (textHash.shl(5) - textHash + text[i].code) % mod
        if (i < m - 1) highestPower = (highestPower.shl(5) - highestPower) % mod
    }

    for (i in 0..(n - m)) {
        // 哈希值匹配，进行字符串逐字符比对
        if (patternHash == textHash && text.substring(i, i + m) == pattern) return i
        // 计算下一个窗口的哈希值
        if (i < n - m) {
            textHash = (textHash - text[i].code * highestPower % mod + mod) % mod // 移除最高位
            textHash = (textHash.shl(5) - textHash + text[i + m].code) % mod // 加入新字符
        }
    }

    return -1
}
```

Robin-Karp 算法也可以进行多模式匹配，据说论文查重等应用中一般都是使用此算法。

## KMP 算法

* [1977 年 KMP 算法论文](https://epubs.siam.org/doi/abs/10.1137/0206024){:target="_blank"}
* [1980 年 Rytter 纠正 Knuth 的论文](https://epubs.siam.org/doi/10.1137/0209037){:target="_blank"}

## Boyer-Moore

https://oi-wiki.org/string/bm/
* [1977 年 Boyer–Moore 算法论文](https://dl.acm.org/doi/10.1145/359842.359859){:target="_blank"}
* [1979 年介绍 Galil 算法的论文](https://dl.acm.org/doi/10.1145/359146.359148){:target="_blank"}

## Sunday

## Aho-Corasick

## BNDM

## Bob 算法

maybe someday:zany_face:

最后，可以使用 [LeetCode 28 Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/description/){:target="_blank"} 来测试以上各种算法代码。

<!-- https://oi-wiki.org/string/match/ -->
<!-- https://www.cnblogs.com/1996swg/p/7116017.html -->
<!-- https://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html -->