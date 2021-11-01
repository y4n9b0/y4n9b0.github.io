---
layout: post
title: Kth Largest Element in an Array
date: 2021-10-31 17:30:00 +0800
categories: algorithm
tags: leetcode
published: true
---

* content
{:toc}

## LeetCode 215

Given an integer array nums and an integer k, return the kth largest element in the array.<br>
Note that it is the kth largest element in the sorted order, not the kth distinct element.

## 先排序再遍历

先对数组进行排序，然后遍历第 k 大的数，思路简单且实现也简单，但即使是最优的快排算法复杂度也需要O(N*logN + k)，显然不是我们所期望的结果。

## 优先队列 PriorityQueue

```kotlin
fun findKthLargest(nums: IntArray, k: Int): Int {
    val queue: PriorityQueue<Int> = PriorityQueue<Int>()
    nums.forEach { queue.add(it) }
    for (index in 1..nums.size - k){
        queue.poll()
    }
    return queue.peek()
}
```

PriorityQueue 默认是小根堆，可以通过构造自定义的比较器实现大根堆：
```kotlin
fun findKthLargest(nums: IntArray, k: Int): Int {
    val queue: PriorityQueue<Int> = PriorityQueue<Int> { o1, o2 -> o2 - o1 }
    nums.forEach { queue.add(it) }
    for (index in 1 until k) {
        queue.poll()
    }
    return queue.peek()
}
```

当数组 size 很大，大到内存也无法加载的时候（数组 nums 可以通过文件流的形式输入），可以对上面的代码优化，仅维持一个 size 为 k 的 PriorityQueue
```kotlin
fun findKthLargest(nums: IntArray, k: Int): Int {
    val queue: PriorityQueue<Int> = PriorityQueue<Int>()
    nums.forEachIndexed { index, i ->
        if (index < k) {
            queue.add(i)
        } else if (i > queue.peek()){
            queue.poll()
            queue.add(i)
        }
    }
    return queue.peek()
}
```

## 快排思想

```kotlin
fun findKthLargest(nums: IntArray, k: Int): Int {
    return halfQuickSort(nums, k, 0, nums.size - 1)
}

fun halfQuickSort(nums: IntArray, k: Int, start: Int, end: Int): Int {
    var low = start
    var high = end
    val pivot = nums[start]
    while (low != high) {
        // 这里和pivot比较 作降序排列，左边大右边小，方便后面return语句的判断比较
        while (nums[high] <= pivot && low < high) high--
        while (nums[low] >= pivot && low < high) low++
        if (low < high) nums[low] = nums[high].apply { nums[high] = nums[low] }
    }
    nums[start] = nums[low]
    nums[low] = pivot
    // println("low=$low high=$high nums=${nums.contentToString()}")
    return when {
        low == k - 1 -> nums[low]
        low > k - 1 -> quickSortFind(nums, k, start, low - 1)
        else -> quickSortFind(nums, k, low + 1, end)
    }
}
```

这个类快排的算法复杂度分析很有意思。第一次交换，算法复杂度为O(N)，接下来的过程和快速排序不同，快速排序是要继续处理两边的数据，再合并，合并操作的算法复杂度是O(1)，于是总的算法复杂度是O(N*logN)（可以这么理解，每次交换用了N，一共logN次）。但是这里在确定枢纽元的相对位置（在K的左边或者右边）之后不用再对剩下的一半进行处理。也就是说第二次插入的算法复杂度不再是O(N)而是O(N/2)，这不还是一样吗？其实不一样，因为接下来的过程是1+1/2+1/4+........ = 2，换句话说就是一共是O(2N)的算法复杂度也就是O(N)的算法复杂度。通过把算法复杂度在每次递归中下降一些，最终让整个算法的复杂度下降极多。这里比较关键的是递归复杂度数列最终收敛成 N 的线性倍。
