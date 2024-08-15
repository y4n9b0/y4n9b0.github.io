---
layout: post
title: 摩尔投票算法
date: 2024-08-15 20:00:00 +0800
categories: algorithm
tags: vote
published: true
---

* content
{:toc}

[Boyer–Moore majority vote algorithm](https://www.cs.utexas.edu/users/moore/best-ideas/mjrty/index.html){:target="_blank"}，也叫多数投票算法，仅使用常数空间在 size=N 的序列中找到最多 K-1 个出现次数超过 $$\lfloor\tfrac{N}{K}\rfloor$$ 次的元素。

找众数问题，可以很容易地用哈希表计数来求解，但是摩尔投票算法的特点在于：常数级别的空间复杂度。不过，它只适用于事先知道众数元素出现次数不少于一定比例的情况。

## Majority Element

[LeetCode 169.多数元素](https://leetcode.com/problems/majority-element/){:target="_blank"}

```text
Given an array nums of size n, return the majority element.

The majority element is the element that appears more than ⌊n/2⌋ times. 
You may assume that the majority element always exists in the array.
```

**Example 1:**
```
Input: nums = [3,2,3]
Output: 3
```

**Example 2:**
```
Input: nums = [2,2,1,1,1,2,2]
Output: 2
``` 

**Constraints:**
* n == nums.length
* 1 <= n <= 5 * 10$$^4$$
* -$$10^9$$ <= nums[i] <= $$10^9$$

**Solution:**
```kotlin
class Solution {
    fun majorityElement(nums: IntArray): Int {
        var major = 0
        var count = 0
        nums.forEach {
            if (count == 0) major = it
            count += if (it == major) 1 else -1
        }
        return major
    }
}
```

##  Majority Element II

[LeetCode 229.多数元素 II](https://leetcode.com/problems/majority-element-ii/){:target="_blank"}

```text
Given an integer array of size n, find all elements that appear more than ⌊ n/3 ⌋ times.
```

**Example 1:**
```
Input: nums = [3,2,3]
Output: [3]
```

**Example 2:**
```
Input: nums = [1]
Output: [1]
``` 

**Example 2:**
```
Input: nums = [1,2]
Output: [1,2]
``` 

**Constraints:**
* 1 <= nums.length <= $$5 * 10^4$$
* -$$10^9$$ <= nums[i] <= $$10^9$$

**Solution 1:**
```kotlin
class Solution {
    fun majorityElement(nums: IntArray): List<Int> {
        var major1 = 0
        var major2 = 0
        var count1 = 0
        var count2 = 0
        nums.forEach {
            if (count1 > 0 && it == major1) {
                count1++
            } else if (count2 > 0 && it == major2) {
                count2++
            } else if (count1 == 0) {
                major1 = it
                count1++
            } else if (count2 == 0) {
                major2 = it
                count2++
            } else {
                count1--
                count2--
            }
        }

        count1 = 0
        count2 = 0
        nums.forEach {
            if (it == major1) count1++
            else if (it == major2) count2++
        }
        val ans = mutableListOf<Int>()
        if (count1 > nums.size / 3) ans.add(major1)
        if (count2 > nums.size / 3) ans.add(major2)
        return ans
    }
}
```

**Solution 2(通用版):**
```kotlin
private const val K = 3

class Solution {
    fun majorityElement(nums: IntArray): List<Int> {
        val candidates = IntArray(K) { Int.MIN_VALUE }
        val counts = IntArray(K) { 0 }
        nums.forEach { n ->
            repeat(K) { i ->
                if (counts[i] > 0 && candidates[i] == n) {
                    counts[i]++
                    return@forEach
                }
            }
            repeat(K) { i ->
                if (counts[i] == 0) {
                    candidates[i] = n
                    counts[i]++
                    return@forEach
                }
            }
            repeat(K) { i ->
                counts[i]--
            }
        }

        counts.fill(0)
        nums.forEach { n ->
            repeat(K) { i ->
                if (candidates[i] == n) {
                    counts[i]++
                    return@forEach
                }
            }
        }
        return candidates.filterIndexed { i, _ -> counts[i] > nums.size / K }
    }
}
```

<!-- https://writings.sh/post/boyer%E2%80%93moore-majority-vote -->