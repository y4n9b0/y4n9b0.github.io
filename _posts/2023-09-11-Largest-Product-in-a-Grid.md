---
layout: post
title: 矩阵中的最大乘积
date: 2023-09-11 22:30:00 +0800
categories: math
tags: math
published: true
---

* content
{:toc}

## 问题

```
08 02 22 97 38 15 00 40 00 75 04 05 07 78 52 12 50 77 91 08
49 49 99 40 17 81 18 57 60 87 17 40 98 43 69 48 04 56 62 00
81 49 31 73 55 79 14 29 93 71 40 67 53 88 30 03 49 13 36 65
52 70 95 23 04 60 11 42 69 24 68 56 01 32 56 71 37 02 36 91
22 31 16 71 51 67 63 89 41 92 36 54 22 40 40 28 66 33 13 80
24 47 32 60 99 03 45 02 44 75 33 53 78 36 84 20 35 17 12 50
32 98 81 28 64 23 67 10 26 38 40 67 59 54 70 66 18 38 64 70
67 26 20 68 02 62 12 20 95 63 94 39 63 08 40 91 66 49 94 21
24 55 58 05 66 73 99 26 97 17 78 78 96 83 14 88 34 89 63 72
21 36 23 09 75 00 76 44 20 45 35 14 00 61 33 97 34 31 33 95
78 17 53 28 22 75 31 67 15 94 03 80 04 62 16 14 09 53 56 92
16 39 05 42 96 35 31 47 55 58 88 24 00 17 54 24 36 29 85 57
86 56 00 48 35 71 89 07 05 44 44 37 44 60 21 58 51 54 17 58
19 80 81 68 05 94 47 69 28 73 92 13 86 52 17 77 04 89 55 40
04 52 08 83 97 35 99 16 07 97 57 32 16 26 26 79 33 27 98 66
88 36 68 87 57 62 20 72 03 46 33 67 46 55 12 32 63 93 53 69
04 42 16 73 38 25 39 11 24 94 72 18 08 46 29 32 40 62 76 36
20 69 36 41 72 30 23 88 34 62 99 69 82 67 59 85 74 04 36 16
20 73 35 29 78 31 90 01 74 31 49 71 48 86 81 16 23 57 05 54
01 70 54 71 83 51 54 69 16 92 33 48 61 43 52 01 89 19 67 48
```
在如上这个矩阵中，四个呈一直线（竖直、水平或对角线）相邻的数的乘积最大是多少？

## 暴力枚举

就算是 bruce force，抠腚也是有技巧的。矩阵的每一个点都有四个方向的直线：水平、垂直、右对角线、左对角线。对于矩阵每个点，遍历其所有方向，步进加一判断乘积。

```kotlin
fun main() {
    val str: String = """
        08 02 22 97 38 15 00 40 00 75 04 05 07 78 52 12 50 77 91 08
        49 49 99 40 17 81 18 57 60 87 17 40 98 43 69 48 04 56 62 00
        81 49 31 73 55 79 14 29 93 71 40 67 53 88 30 03 49 13 36 65
        52 70 95 23 04 60 11 42 69 24 68 56 01 32 56 71 37 02 36 91
        22 31 16 71 51 67 63 89 41 92 36 54 22 40 40 28 66 33 13 80
        24 47 32 60 99 03 45 02 44 75 33 53 78 36 84 20 35 17 12 50
        32 98 81 28 64 23 67 10 26 38 40 67 59 54 70 66 18 38 64 70
        67 26 20 68 02 62 12 20 95 63 94 39 63 08 40 91 66 49 94 21
        24 55 58 05 66 73 99 26 97 17 78 78 96 83 14 88 34 89 63 72
        21 36 23 09 75 00 76 44 20 45 35 14 00 61 33 97 34 31 33 95
        78 17 53 28 22 75 31 67 15 94 03 80 04 62 16 14 09 53 56 92
        16 39 05 42 96 35 31 47 55 58 88 24 00 17 54 24 36 29 85 57
        86 56 00 48 35 71 89 07 05 44 44 37 44 60 21 58 51 54 17 58
        19 80 81 68 05 94 47 69 28 73 92 13 86 52 17 77 04 89 55 40
        04 52 08 83 97 35 99 16 07 97 57 32 16 26 26 79 33 27 98 66
        88 36 68 87 57 62 20 72 03 46 33 67 46 55 12 32 63 93 53 69
        04 42 16 73 38 25 39 11 24 94 72 18 08 46 29 32 40 62 76 36
        20 69 36 41 72 30 23 88 34 62 99 69 82 67 59 85 74 04 36 16
        20 73 35 29 78 31 90 01 74 31 49 71 48 86 81 16 23 57 05 54
        01 70 54 71 83 51 54 69 16 92 33 48 61 43 52 01 89 19 67 48
      """.trimIndent()
    val matrix: Array<IntArray> = str.split("\n").map { line ->
        line.split(" ").map { it.toInt() }.toIntArray()
    }.toTypedArray()

    assert(largestProductOfMatrix(4, matrix) == 70600674L)
}

fun largestProductOfMatrix(n: Int, matrix: Array<IntArray>): Long {
    var max = 0L
    repeat(matrix.size) { x ->
        repeat(matrix.size) { y ->
            max = largestProductOfPoint(x, y, n, matrix).coerceAtLeast(max)
        }
    }
    return max
}

fun largestProductOfPoint(x: Int, y: Int, n: Int, matrix: Array<IntArray>): Long {
    // directions 恒定不变，声明为顶层变量更佳，定义在此处方便理解
    val directions: Array<Pair<Int, Int>> = arrayOf(
        Pair(0, 1),    // 水平向右
        Pair(1, 0),    // 垂直向下
        Pair(1, 1),    // 右对角线向下
        Pair(1, -1),   // 左对角线向下
    )
    val size = matrix.size
    var max = 0L
    directions.forEach { direction ->
        var sum = 1L
        for (step in 0 until n) {
            val dx = x + step * direction.first
            val dy = y + step * direction.second
            if (dx < 0 || dx >= size || dy < 0 || dy >= size) break
            sum *= matrix[dx][dy]
        }
        if (sum > max) {
            max = sum
        }
    }
    return max
}
```

## 递归

考虑矩阵 `n*n` 由其最后一行和最后一列加上前 `(n-1)*(n-1)` 个元素矩阵组成，那么对于矩阵 `n*n` 求最大乘积可演变为求前 `(n-1)*(n-1)` 个元素矩阵的最大乘积，也即动态规划。从效率上来说与前面的暴力枚举没有不同，递归反而还多了 stackoverflow 的风险，算是一种新思路吧。

```kotlin
fun main() {
    val str: String = """
        08 02 22 97 38 15 00 40 00 75 04 05 07 78 52 12 50 77 91 08
        49 49 99 40 17 81 18 57 60 87 17 40 98 43 69 48 04 56 62 00
        81 49 31 73 55 79 14 29 93 71 40 67 53 88 30 03 49 13 36 65
        52 70 95 23 04 60 11 42 69 24 68 56 01 32 56 71 37 02 36 91
        22 31 16 71 51 67 63 89 41 92 36 54 22 40 40 28 66 33 13 80
        24 47 32 60 99 03 45 02 44 75 33 53 78 36 84 20 35 17 12 50
        32 98 81 28 64 23 67 10 26 38 40 67 59 54 70 66 18 38 64 70
        67 26 20 68 02 62 12 20 95 63 94 39 63 08 40 91 66 49 94 21
        24 55 58 05 66 73 99 26 97 17 78 78 96 83 14 88 34 89 63 72
        21 36 23 09 75 00 76 44 20 45 35 14 00 61 33 97 34 31 33 95
        78 17 53 28 22 75 31 67 15 94 03 80 04 62 16 14 09 53 56 92
        16 39 05 42 96 35 31 47 55 58 88 24 00 17 54 24 36 29 85 57
        86 56 00 48 35 71 89 07 05 44 44 37 44 60 21 58 51 54 17 58
        19 80 81 68 05 94 47 69 28 73 92 13 86 52 17 77 04 89 55 40
        04 52 08 83 97 35 99 16 07 97 57 32 16 26 26 79 33 27 98 66
        88 36 68 87 57 62 20 72 03 46 33 67 46 55 12 32 63 93 53 69
        04 42 16 73 38 25 39 11 24 94 72 18 08 46 29 32 40 62 76 36
        20 69 36 41 72 30 23 88 34 62 99 69 82 67 59 85 74 04 36 16
        20 73 35 29 78 31 90 01 74 31 49 71 48 86 81 16 23 57 05 54
        01 70 54 71 83 51 54 69 16 92 33 48 61 43 52 01 89 19 67 48
      """.trimIndent()
    val matrix: Array<IntArray> = str.split("\n").map { line ->
        line.split(" ").map { it.toInt() }.toIntArray()
    }.toTypedArray()

    assert(largestProductOfMatrix(matrix.size, 4, matrix) == 70600674L)
}

fun largestProductOfMatrix(size: Int, n: Int, matrix: Array<IntArray>): Long {
    assert(n <= matrix.size)
    var max = 0L
    if (n == size) {
        repeat(n) { x ->
            var sumOfHorizontal = 1L
            var sumOfVertical = 1L
            repeat(n) { y ->
                sumOfHorizontal *= matrix[x][y]
                sumOfVertical *= matrix[y][x]
            }
            max = maxOf(max, sumOfHorizontal, sumOfVertical)
        }
        var sumOfDiagonalRight = 1L
        var sumOfDiagonalLeft = 1L
        repeat(n) {
            sumOfDiagonalRight *= matrix[it][it]
            sumOfDiagonalLeft *= matrix[n - 1 - it][it]
        }
        return maxOf(max, sumOfDiagonalRight, sumOfDiagonalLeft)
    }
    repeat(size) {
        max = maxOf(
            max,
            largestProductOfPoint(it, size - 1, n, matrix),
            largestProductOfPoint(size - 1, it, n, matrix)
        )
    }
    return largestProductOfMatrix(size - 1, n, matrix).coerceAtLeast(max)
}

fun largestProductOfPoint(x: Int, y: Int, n: Int, matrix: Array<IntArray>): Long {
    // 注意这里 directions 与暴力枚举有区别，递归是反方向步进
    val directions: Array<Pair<Int, Int>> = arrayOf(
        Pair(0, 1),     // 水平向右
        Pair(-1, 0),    // 垂直向上
        Pair(-1, 1),    // 右对角线向上
        Pair(-1, -1),   // 左对角线向上
    )
    val size = matrix.size
    var max = 0L
    directions.forEach { direction ->
        var sum = 1L
        for (step in 0 until n) {
            val dx = x + step * direction.first
            val dy = y + step * direction.second
            if (dx < 0 || dx >= size || dy < 0 || dy >= size) break
            sum *= matrix[dx][dy]
        }
        if (sum > max) {
            max = sum
        }
    }
    return max
}
```

## 方向枚举+滑动窗口

可以肯定滑动窗口会带来效率的提升。

```kotlin
fun main() {
    val str: String = """
        08 02 22 97 38 15 00 40 00 75 04 05 07 78 52 12 50 77 91 08
        49 49 99 40 17 81 18 57 60 87 17 40 98 43 69 48 04 56 62 00
        81 49 31 73 55 79 14 29 93 71 40 67 53 88 30 03 49 13 36 65
        52 70 95 23 04 60 11 42 69 24 68 56 01 32 56 71 37 02 36 91
        22 31 16 71 51 67 63 89 41 92 36 54 22 40 40 28 66 33 13 80
        24 47 32 60 99 03 45 02 44 75 33 53 78 36 84 20 35 17 12 50
        32 98 81 28 64 23 67 10 26 38 40 67 59 54 70 66 18 38 64 70
        67 26 20 68 02 62 12 20 95 63 94 39 63 08 40 91 66 49 94 21
        24 55 58 05 66 73 99 26 97 17 78 78 96 83 14 88 34 89 63 72
        21 36 23 09 75 00 76 44 20 45 35 14 00 61 33 97 34 31 33 95
        78 17 53 28 22 75 31 67 15 94 03 80 04 62 16 14 09 53 56 92
        16 39 05 42 96 35 31 47 55 58 88 24 00 17 54 24 36 29 85 57
        86 56 00 48 35 71 89 07 05 44 44 37 44 60 21 58 51 54 17 58
        19 80 81 68 05 94 47 69 28 73 92 13 86 52 17 77 04 89 55 40
        04 52 08 83 97 35 99 16 07 97 57 32 16 26 26 79 33 27 98 66
        88 36 68 87 57 62 20 72 03 46 33 67 46 55 12 32 63 93 53 69
        04 42 16 73 38 25 39 11 24 94 72 18 08 46 29 32 40 62 76 36
        20 69 36 41 72 30 23 88 34 62 99 69 82 67 59 85 74 04 36 16
        20 73 35 29 78 31 90 01 74 31 49 71 48 86 81 16 23 57 05 54
        01 70 54 71 83 51 54 69 16 92 33 48 61 43 52 01 89 19 67 48
      """.trimIndent()
    val matrix: Array<IntArray> = str.split("\n").map { line ->
        line.split(" ").map { it.toInt() }.toIntArray()
    }.toTypedArray()

    assert(largestProductOfMatrix(4, matrix) == 70600674L)
}

fun largestProductOfMatrix(n: Int, matrix: Array<IntArray>): Long {
    var max = 0L
    repeat(matrix.size) {
        // 垂直向下
        max = largestProduct(n, matrix[it]).coerceAtLeast(max)

        // 水平向右
        max = largestProduct(n, matrix.map { array -> array[it] }.toIntArray()).coerceAtLeast(max)

        // 右对角线，从中间向右上遍历
        matrix.filterIndexed { index, _ ->
            index + it < matrix.size
        }.takeIf { array ->
            array.size >= n
        }?.mapIndexed { index, array ->
            array[index + it]
        }?.toIntArray()?.apply {
            max = largestProduct(n, this).coerceAtLeast(max)
        }
        // 右对角线，从中间向左下遍历
        matrix.filterIndexed { index, _ ->
            index > it
        }.takeIf { array ->
            array.size >= n
        }?.mapIndexed { index, array ->
            array[index]
        }?.toIntArray()?.apply {
            max = largestProduct(n, this).coerceAtLeast(max)
        }

        // 左对角线，从中间向左上遍历
        matrix.filterIndexed { index, _ ->
            index + it < matrix.size
        }.takeIf { array ->
            array.size >= n
        }?.mapIndexed { index, array ->
            array[matrix.size - 1 - it - index]
        }?.toIntArray()?.apply {
            max = largestProduct(n, this).coerceAtLeast(max)
        }
        // 左对角线，从中间向右下遍历
        matrix.filterIndexed { index, _ ->
            index > it
        }.takeIf { array ->
            array.size >= n
        }?.mapIndexed { index, array ->
            array[matrix.size - 1 - index]
        }?.toIntArray()?.apply {
            max = largestProduct(n, this).coerceAtLeast(max)
        }
    }
    return max
}

// 滑动窗口
fun largestProduct(n: Int, array: IntArray): Long {
    var max = 0L
    var sum = 1L
    var windowZeroCount = 0
    array.forEachIndexed { index, num ->
        if (num == 0) {
            windowZeroCount++
        } else {
            sum *= num
        }
        if (index >= n) {
            if (array[index - n] == 0) {
                windowZeroCount--
            } else {
                sum /= array[index - n]
            }
        }
        if (windowZeroCount == 0 && sum > max) {
            max = sum
        }
    }
    return max
}
```

## 一些小技巧

扩充数组可以省去很多边界判断，比如 `20*20` 矩阵求最大的 4 个相邻乘积，我们可以把矩阵扩大为 `24*24`，最后 4 行和最后 4 列全部填充 0。

<!-- https://pe-cn.github.io/11/ -->
