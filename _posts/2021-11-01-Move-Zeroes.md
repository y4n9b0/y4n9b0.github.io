---
layout: post
title: Move Zeroes
date: 2021-11-01 16:30:00 +0800
categories: algorithm
tags: leetcode
published: true
---

* content
{:toc}

## LeetCode 283

Given an integer array nums, move all 0's to the end of it while maintaining the relative order of the non-zero elements.<br>
Note that you must do this in-place without making a copy of the array.

```kotlin
fun moveZeroes(nums: IntArray): Unit {
    var index = 0
    for(i in nums.indices){
        if (nums[i] != 0){
            nums[index++] = nums[i]
        }
    }
    for (i in index until nums.size){
        nums[i] = 0
    }
}
```

```kotlin
fun moveZeroes(nums: IntArray): Unit {
    var index = 0
    for(i in nums.indices){
        if (nums[i] != 0){
            nums[index] = nums[i].apply { nums[i] = nums[index] }
            index++
        }
    }
}
```

```kotlin
fun moveZeroes(nums: IntArray): Unit {
    var index = 0
    for(i in nums.indices){
        if (nums[i] != 0){
            if(i > index){
                nums[index] = nums[i].apply { nums[i] = nums[index] }
            }
            index++
        }
    }
}
```

```kotlin
fun moveZeroes(nums: IntArray): Unit {
    var index = 0
    for(i in nums.indices){
        if (nums[i] != 0){
            if(i > index){
                nums[index] = nums[i]
                nums[i] = 0
            }
            index++
        }
    }
}
```
