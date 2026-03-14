---
layout: post
title: kotlin 扩展函数奇技淫巧
date: 2026-03-14 15:30:00 +0800
categories: extension-function
tags: kotlin、jvm
published: true
---

* content
{:toc}

## Iterable

严格意义上来说 Iterable 是一个 kotlin standard library 里的接口。
标准库里对 Iterable 扩展了相当多的便捷函数（kotlin 2.2.20 的实现中有 160 个左右）。

如果我们封装某种数据结构，需要扩展一些方法以便外部不时之需，这个时候可以优先考虑实现 Iterable，等于白嫖一百多个便捷的扩展方法，几乎涵盖能想到的所有场景。而且随着 kotlin 版本的升级还会自动拥有高版本新增的扩展方法（如果 kotlin 高版本标准库有新增的话）。

### Iterable 定义

```kotlin
public actual interface Iterable<out T> {
    /**
     * Returns an iterator over the elements of this object.
     */
    public actual operator fun iterator(): Iterator<T>
}
```

从 Iterable 的接口定义不难看出，需要实现 `iterator` 方法返回一个 Iterator 对象。实际上最终起作用的还是 Iterator。

### Iterator 定义

```kotlin
public actual interface Iterator<out T> {
    /**
     * Returns the next element in the iteration.
     *
     * @throws NoSuchElementException if the iteration has no next element.
     */
    public actual operator fun next(): T

    /**
     * Returns `true` if the iteration has more elements.
     */
    public actual operator fun hasNext(): Boolean
}
```

Iterator 依然是一个接口，只需要实现两个方法：
* `hasNext` 是否有下一个元素
* `next` 返回下一个元素，如果没有下一个元素会直接抛异常 `NoSuchElementException`

## Grouping

Grouping 也是一个 kotlin standard library 里的接口，同时也有不少的扩展函数。在某些场景下相当有用。

### 统计字符串里的字符出现次数

```kotlin
fun countFrequency(text: String): Map<Char, Int> {
    return text.groupingBy { it }.eachCount()
}
```

### LeetCode 347

[Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/description/){:target="_blank"}

Given an integer array nums and an integer k, return the k most frequent elements. You may return the answer in any order.

```kotlin
class Solution {
    fun topKFrequent(nums: IntArray, k: Int): IntArray {
        return nums.asSequence()
            .groupingBy { it }
            .eachCount()
            .entries
            .sortedByDescending { it.value }
            .take(k)
            .map { it.key }
            .toIntArray()
    }
}
```

这种流式扣腚的丝滑、顺畅程度堪比蹲马桶上一泻千里。

### Grouping 定义

```kotlin
@SinceKotlin("1.1")
public interface Grouping<T, out K> {
    /** Returns an [Iterator] over the elements of the source of this grouping. */
    public fun sourceIterator(): Iterator<T>
    /** Extracts the key of an [element]. */
    public fun keyOf(element: T): K
}
```

不难看出，Grouping 的核心还是 Iterator。当然，几乎所有集合 API 最终都是 Iterator。

## runningReduce

reduce 和 fold 很多人都知道，但是用过 runningReduce 和 runningFold 的应该不多。

runningReduce 可以用来求前缀和：

```kotlin
fun prefixSum(nums: IntArray): IntArray {
    return nums.runningReduce { sum, n -> sum + n }.toIntArray()
}
```

相应地还有 runningReduceIndexed、runningFoldIndexed。