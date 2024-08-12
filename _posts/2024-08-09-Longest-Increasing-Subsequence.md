---
layout: post
title: 最长上升子序列
date: 2024-08-09 15:30:00 +0800
categories: algorithm
tags: LIS
published: true
---

* content
{:toc}

## LIS

[LeetCode 300](https://leetcode.com/problems/longest-increasing-subsequence/description/){:target="_blank"}

Given an integer array nums, return the length of the longest strictly increasing subsequence.

**动态规划** 复杂度 $$O(n^2)$$

```kotlin
class Solution {
    fun lengthOfLIS(nums: IntArray): Int {
        // dp[i] 表示前 i + 1 个数的最大上升子序列个数
        val dp = IntArray(nums.size) { 1 }
        for (i in nums.indices) {
            var max = 1
            for (j in 0 until i) {
                if (nums[j] < nums[i]) max = max.coerceAtLeast(dp[j] + 1)
            }
            dp[i] = max
        }
        return dp.max()
    }
}
```

**贪心 + 二分查找** 复杂度 $$O(n\log{n})$$

```kotlin
class Solution {
    fun lengthOfLIS(nums: IntArray): Int {
        val sub = mutableListOf<Int>()
        for (i in nums.indices) {
            var index = sub.binarySearch(nums[i], 0, sub.size)
            if (index < 0) index = -(index + 1)
            if (index == sub.size) sub.add(nums[i]) else sub[index] = nums[i]
        }
        return sub.size
    }
}
```

**动态规划 + 树状数组** 复杂度 $$O(n\log{n})$$

```kotlin
class Solution {
    fun lengthOfLIS(nums: IntArray): Int {
        var ans = 0
        val dp = IntArray(nums.size + 1) { 0 }
        nums.mapIndexed { i, n ->
            Pair(i + 1, n)
        }.sortedWith { (i1, n1), (i2, n2) ->
            if (n1 == n2) i2 - i1 else n1 - n2
        }.forEach { (index, num) ->
            var max = 0
            var i = index
            while (i > 0) {
                max = max.coerceAtLeast(dp[i])
                i -= i.and(-i)
            }
            max++
            i = index
            while (i <= nums.size) {
                dp[i] = dp[i].coerceAtLeast(max)
                i += i.and(-i)
            }
            ans = ans.coerceAtLeast(max)
        }
        return ans
    }
}
```

## 求 LIS 个数

[LeetCode 673](https://leetcode.com/problems/number-of-longest-increasing-subsequence/description/){:target="_blank"}

```kotlin
// todo
```

## 求 LIS 具体序列

[ZOJ 4028](https://pintia.cn/problem-sets/91827364500/exam/problems/type/7?problemSetProblemId=91827370262&page=30){:target="_blank"}

```kotlin
// todo
```

<!-- https://writings.sh/post/longest-increasing-subsequence-revisited -->
<!-- https://writings.sh/post/find-number-of-lis -->
<!-- https://writings.sh/post/binary-indexed-tree -->
<!-- https://blog.csdn.net/lxt_Lucia/article/details/81206439 -->
<!-- https://leetcode.com/problems/longest-increasing-subsequence/solutions/1326308/c-python-dp-binary-search-bit-segment-tree-solutions-picture-explain-o-nlogn/ -->
<!-- https://blog.csdn.net/lxt_Lucia/article/details/89228550 -->