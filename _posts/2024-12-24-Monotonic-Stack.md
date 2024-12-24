---
layout: post
title: Monotonic Stack
date: 2024-12-24 17:30:00 +0800
categories: algorithm
tags: algorithm
published: true
---

* content
{:toc}

## 单调栈

顾名思义，单调栈即满足单调性的栈结构。如果新增的 element 比栈顶还大(自底向顶递减单调栈)/小(自底向顶递增单调栈)，就把栈顶元素 pop 出去。然后继续往下比，直到仍旧保持递减或是递增的顺序。

单调栈往往用于找出数组中每个元素的下一个更大或者更小的元素这类场景。

与单调队列相比，单调栈只在一端进行进出。

## LeetCode 1475

[Final Prices With a Special Discount in a Shop](https://leetcode.com/problems/final-prices-with-a-special-discount-in-a-shop/description/){:target="_blank"}

You are given an integer array prices where prices[i] is the price of the ith item in a shop.

There is a special discount for items in the shop. If you buy the ith item, then you will receive a discount equivalent to prices[j] where j is the minimum index such that j > i and prices[j] <= prices[i]. Otherwise, you will not receive any discount at all.

Return an integer array answer where answer[i] is the final price you will pay for the ith item of the shop, considering the special discount.

**Example 1:**

```
Input: prices = [8,4,6,2,3]
Output: [4,2,4,2,3]
Explanation: 
For item 0 with price[0]=8 you will receive a discount equivalent to prices[1]=4, therefore, the final price you will pay is 8 - 4 = 4.
For item 1 with price[1]=4 you will receive a discount equivalent to prices[3]=2, therefore, the final price you will pay is 4 - 2 = 2.
For item 2 with price[2]=6 you will receive a discount equivalent to prices[3]=2, therefore, the final price you will pay is 6 - 2 = 4.
For items 3 and 4 you will not receive any discount at all.
```

**Example 2:**

```
Input: prices = [1,2,3,4,5]
Output: [1,2,3,4,5]
Explanation: In this case, for all items, you will not receive any discount at all.
```

**Example 3:**

```
Input: prices = [10,1,1,6]
Output: [9,0,1,6]
```

**Constraints:**

* 1 <= prices.length <= 500
* 1 <= prices[i] <= 1000

**Approach 1: Brute Force**

```kotlin
class Solution {
  fun finalPrices(prices: IntArray): IntArray {
    repeat(prices.size) {
      var index = it + 1
      while (index < prices.size && prices[index] > prices[it]) index++
      if (index < prices.size) prices[it] -= prices[index]
    }
    return prices
  }
}
```

* 时间复杂度 $$O(n^2)$$
* 空间复杂度 $$O(n)$$

**Approach 2: Monotonic Stack**

```kotlin
class Solution {
  fun finalPrices(prices: IntArray): IntArray {
    val stack = Stack<Int>()
    prices.forEachIndexed { i, p ->
      while (stack.isNotEmpty() && prices[stack.peek()] >= p) prices[stack.pop()] -= p
      stack.push(i)
    }
    return prices
  }
}
```

从左到右依次处理数组，我们需要判断出当前价格对之前的任意一个价格能否构成优惠。栈内维护着之前还没有找到优惠的价格，循环将当前价格与栈顶价格进行比较，直到 pop 出栈顶所有大于或等于当前价格的元素（并处理他们的优惠），然后再将当前价格 push 到栈顶。显然栈内价格元素自底向顶是依次递增的。

* 时间复杂度 $$O(n)$$
* 空间复杂度 $$O(n)$$

**Approach 3: Mock Monotonic Stack**

```kotlin
class Solution {
  fun finalPrices(prices: IntArray): IntArray {
    val stack = IntArray(prices.size)
    var top = -1
    prices.forEachIndexed { i, p ->
      while (top > -1 && prices[stack[top]] >= p) prices[stack[top--]] -= p
      stack[++top] = i
    }
    return prices
  }
}
```

使用数组模拟栈，增加 top 变量记录栈顶位置，数组相对于栈减少了各种入栈/出栈操作，执行更快。

* 时间复杂度 $$O(n)$$
* 空间复杂度 $$O(n)$$

<!-- https://oi-wiki.org/ds/monotonous-stack/ -->
<!-- https://leetcode.com/problem-list/monotonic-stack/ -->
<!-- https://leetcode.com/problems/final-prices-with-a-special-discount-in-a-shop/editorial/ -->