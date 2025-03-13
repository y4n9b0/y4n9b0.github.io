---
layout: post
title: 差分数组
date: 2025-03-13 20:30:00 +0800
categories: algorithm
tags: algorithm
published: true
---

* content
{:toc}

## 前言

在算法问题中，如何高效处理数组区间的修改操作是一个常见且重要的话题。比如：

* 需要在一个数组的多个区间上进行加减操作，最终输出结果；
* 频繁调整数组部分区间的值，但需要快速恢复整体状态；
* 处理二维平面或更高维数据的区域更新问题。

通常，我们可以使用暴力遍历的方法，但当数组规模较大、操作次数较多时，暴力方法的效率会迅速成为瓶颈。这时，我们就需要一种能够在常数时间内完成区间修改的高效工具 —— 差分数组。

差分数组是一种借助“记录变化”而非直接修改原数组的巧妙方法。通过它，我们不仅可以快速处理区间修改，还可以在多次操作后迅速恢复最终的数组状态。

## 什么是差分数组

给定一个数组 arr，我们可以通过一个差分数组 diff 来表示 arr 的变化：

* $$diff[0]$$ = $$arr[0]$$
* $$diff[i]$$ = $$arr[i] - arr[i - 1]，(i > 0)$$

根据差分数组我们可以反推原数组：

* $$arr[0]$$ = $$diff[0]$$
* $$arr[1]$$ = $$diff[1] + arr[0] = diff[1] + diff[0]$$
* $$arr[2]$$ = $$diff[2] + arr[1] = diff[2] + diff[1] + diff[0]$$
* ···

不难看出，差分数组的前缀和即原数组 arr，所以差分数组的题目 tag 往往也包含前缀和。

## 区间更新

差分数组最常见的应用是区间更新。假设我们希望在数组 arr 的区间 [l, r] 中所有元素都加上一个值 v， 差分数组的做法是：

1. 创建一个差分数组 diff，数组 size 比原数组 size 大 1，初始化为 0
2. 对每次区间操作进行处理，更新差分数组：
    ```txt
    diff[l] += v
    diff[r + 1] -= v
    ```
3. 通过差分数组恢复更新后的原数组 arr

## 差分数组的复杂度

* 时间复杂度: O(n + q)，其中 n 是数组长度，q 是区间修改的操作次数。
* 空间复杂度: O(n)

## LeetCode 3355

[Zero Array Transformation I](https://leetcode.com/problems/zero-array-transformation-i/description/){:target="_blank"}

You are given an integer array nums of length n and a 2D array queries, where $$queries[i] = [l_i, r_i]$$.

For each queries[i]:

* Select a subset of indices within the range $$[l_i, r_i]$$ in nums.
* Decrement the values at the selected indices by 1.

A Zero Array is an array where all elements are equal to 0.

Return true if it is possible to transform nums into a Zero Array after processing all the queries sequentially, otherwise return false.

**Example 1:**

```
Input: nums = [1,0,1], queries = [[0,2]]
Output: true
Explanation:
  For i = 0:
    Select the subset of indices as [0,2] and decrement the values at these indices by 1.
    The array will become [0,0,0], which is a Zero Array.
```

**Example 2:**
```
Input: nums = [4,3,2,1], queries = [[1,3],[0,2]]
Output: false
Explanation:
  For i = 0:
    Select the subset of indices as [1,2,3] and decrement the values at these indices by 1.
    The array will become [4,2,1,0].
  For i = 1:
    Select the subset of indices as [0,1,2] and decrement the values at these indices by 1.
    The array will become [3,1,0,0], which is not a Zero Array.
```

**Constraints:**

* 1 <= nums.length <= $$10^5$$
* 0 <= nums[i] <= $$10^5$$
* 1 <= queries.length <= $$10^5$$
* queries[i].length == 2
* 0 <= $$l_i$$ <= $$r_i$$ < nums.length

**Approach:**

```kotlin
class Solution {
    fun isZeroArray(nums: IntArray, queries: Array<IntArray>): Boolean {
        val diffArray = IntArray(nums.size + 1)
        queries.forEach { (l, r) ->
            diffArray[l]--
            diffArray[r + 1]++
        }
        val prefixSum = diffArray.runningReduce { acc, n -> acc + n }
        nums.forEachIndexed { i, n ->
            if (n + prefixSum[i] > 0) return false
        }
        return true
    }
}
```