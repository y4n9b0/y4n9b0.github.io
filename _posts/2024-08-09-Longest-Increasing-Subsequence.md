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

**动态规划**

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

复杂度 $$O(n^2)$$

**贪心 + 二分查找**

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

复杂度 $$O(n*\log{_2}{n})$$

**动态规划优化版**

```kotlin
// todo
```

## 求 LIS 具体序列

[ZOJ 4028](https://pintia.cn/problem-sets/91827364500/exam/problems/type/7?problemSetProblemId=91827370262&page=30){:target="_blank"}

```kotlin
// todo
```

<!-- https://blog.csdn.net/lxt_Lucia/article/details/81206439 -->
<!-- https://leetcode.com/problems/longest-increasing-subsequence/solutions/1326308/c-python-dp-binary-search-bit-segment-tree-solutions-picture-explain-o-nlogn/ -->
<!-- https://blog.csdn.net/lxt_Lucia/article/details/89228550 -->