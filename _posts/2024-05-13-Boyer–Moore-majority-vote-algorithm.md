---
layout: post
title: 摩尔投票算法
date: 2024-05-13 13:30:00 +0800
categories: algorithm
tags: vote
published: true
---

* content
{:toc}

[Boyer–Moore majority vote algorithm](https://www.cs.utexas.edu/users/moore/best-ideas/mjrty/index.html){:target="_blank"}

## problem

[Kotlin Heroes: Practice 10 - Problem A](https://codeforces.com/contest/1959/problem/A){:target="_blank"}

You are given an array 𝑎 consisting of 𝑛(𝑛≥3) positive integers. 
It is known that in this array, all the numbers except one are the same 
(for example, in the array [4,11,4,4] all numbers except one are equal to 4).

Print the index of the element that does not equal others. The numbers in the array are numbered from one.

**Input**<br>
The first line contains a single integer 𝑡(1≤𝑡≤100). Then 𝑡 test cases follow.<br>
The first line of each test case contains a single integer 𝑛(3≤𝑛≤100) — the length of the array 𝑎.<br>
The second line of each test case contains 𝑛 integers 𝑎1,𝑎2,…,𝑎𝑛(1≤𝑎𝑖≤100).<br>
It is guaranteed that all the numbers except one in the 𝑎 array are the same.

**Output**<br>
For each test case, output a single integer — the index of the element that is not equal to others.

**Example**
```
input
4
4
11 13 11 11
5
1 4 4 4 4
10
3 3 3 3 10 3 3 3 3 3
3
20 20 10

output
2
1
5
3
```

## solution

```kotlin
fun main() {
    repeat(readln().toInt()) { println(solve()) }
}

fun solve(): Int {
    readln()
    val list = readln().split(' ').map(String::toInt)
    var majority = 0
    var counter = 0
    repeat(3) {
        if (counter == 0) majority = list[it]
        counter += if (majority == list[it]) 1 else -1
    }
    return list.indexOfFirst { it != majority } + 1
}
```
