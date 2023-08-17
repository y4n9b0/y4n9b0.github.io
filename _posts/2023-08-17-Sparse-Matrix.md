---
layout: post
title: 稀疏矩阵
date: 2023-08-17 11:00:00 +0800
categories: Matrix
tags: Matrix
published: true
---

* content
{:toc}

## Sparse Matrix

矩阵非零元素 NNZ（Numbers of Non-Zero）占全部元素的百分比称之为矩阵的稠密度。<br>
Sparse Matrix（稀疏矩阵），一般是指稠密度 5% 以下的矩阵；反之，非零元素数目与所有元素总数相差不大则称之为稠密矩阵（Dense Matrix）。<br>
这里的矩阵一般都是指二维矩阵，实际场景中二维矩阵占绝大多数，多维矩阵情况比较复杂，不作讨论。<br>
对于稀疏矩阵采用二维数组 A[m][n] 存储时，需要 m * n 个元素大小的空间，由于这 m * n 个空间里大部分的元素都是 0，
所以可以通过一些手段优化存储空间。

## COO

COO 全称 Coordinate Matrix（坐标矩阵）

* 采用三元组(row, col, data)的形式来存储矩阵中非零元素的信息
* 三个数组 row、col 和 data 分别保存非零元素的行下标、列下标与值
* coo<sub>row[i]col[i]</sub> = data[i]，即矩阵的第 row[i] 行、第 col[i] 列的值为 data[i]

每个二维数组中的元素对应着三个变量：行下标、列下标、元素值。
坐标矩阵用三个一维数组把所有非零元素的三个变量按顺序分别存储起来，三个一维数组的长度都等于非零元素的个数。
可以得知，当矩阵稠密度低于 $$\frac{1}{3}$$ 时，存储空间比二维数组少，且稠密度越低需要的存储空间越少。

A[5][4] = $$
\begin{bmatrix}
1 & 0 & 2 & 0 \\
0 & 0 & 3 & 0 \\
0 & 0 & 0 & 0 \\
0 & 5 & 6 & 0 \\
0 & 0 & 0 & 4
\end{bmatrix}
$$

以如上矩阵为例，很容易写出：<br>
row &nbsp;= [0, 0, 1, 3, 3, 4]<br>
col &nbsp;&nbsp;= [0, 2, 2, 1, 2, 3]<br>
data = [1, 2, 3, 5, 6, 4]

## CSR

CSR 全称 Compressed Sparse Row Matrix（压缩稀疏行格式），同样也是用三个一维数组表示：

* data: 按行罗列所有的非零元素的值，数组长度等于非零元素个数
* indices: 按行罗列所有非零元素的列索引，数组长度等于非零元素个数
* indptr: 第 i 个元素记录了前 i - 1 行包含的非零元素的数量，数组长度等于矩阵行数加 1，indptr[0] 恒等于 0

同样以上述矩阵 A[5][4] 为例：<br>
data &nbsp;&nbsp;&nbsp;&nbsp;= [1, 2, 3, 5, 6, 4]<br>
indices = [0, 2, 2, 1, 2, 3]<br>
indptr &nbsp;&nbsp;= [0, 2, 3, 3, 5, 6]

可以得知，对于每一个矩阵都有着唯一的三个数组 data、indices、indptr。现在我们只需要通过这三个数组反推出矩阵 A[5][4] 
便可证明两者表达方式之间存在一一对应的等价关系。

假设要读取第 3 行、第 2 列（下标从 0 开始）的元素 A<sub>32</sub> = 6
* 首先确定矩阵 A 第 i = 3 行的非零元素数量：indptr[i+1] - indptr[i] = indptr[4] - indptr[3] = 5 - 3 = 2，由 indptr 的定义我们得知第 3 行有 2 个非零元素
* 接下来我们需要找出第 3 行非零元素在 data 和 indices 数组中的起始 index（data 和 indices 是一一对应的，所以共用一个 index），也即第 3 行第 1 个非零元素的 index<sub>0</sub> = indptr[i] = indptr[3] = 3，由于 data 和 indices 是按行顺序存储非零元素，所以可以得知第 3 行第 2 个非零元素对应的 index<sub>1</sub> = index<sub>0</sub> + 1 = 4
* 最后我们可以写出这两个元素：<br>
  第 i = 3 行第一个非零元素的列索引 j = indices[index<sub>0</sub>] = indices[3] = 1，对应的元素 A<sub>ij</sub> = A<sub>31</sub> = data[index<sub>0</sub>] = data[3] = 5。<br>
  第 i = 3 行第二个非零元素的列索引 j = indices[index<sub>1</sub>] = indices[4] = 2，对应的元素 A<sub>ij</sub> = A<sub>32</sub> = data[index<sub>1</sub>] = data[4] = 6。<br>
  实际应用中，我们不用遍历出该行的每个元素，只需要按顺序依次寻找，一旦找到列坐标符合我们要寻找的元素，就可以停止遍历。如果遍历完还未找到符合列坐标的元素，说明需要查找的该元素值为 0。

CSR 的 kotlin 简易实现：
```kotlin
val matrix = arrayOf(
    intArrayOf(1, 0, 2, 0),
    intArrayOf(0, 0, 3, 0),
    intArrayOf(0, 0, 0, 0),
    intArrayOf(0, 5, 6, 0),
    intArrayOf(0, 0, 0, 4)
)

fun main() {
    val row = 3
    val col = 2
    val item = findCsrItem(row, col)
    println("row=$row col=$col item=$item")
}

fun findCsrItem(row: Int, col: Int): Int {
    val triple = matrix2Csr()
    val data = triple.first
    val indices = triple.second
    val indptr = triple.third
    val count = getItemCountByRow(row, indptr)
    val initialIndex = getInitialIndexByRow(row, indptr)
    repeat(count) {
        val index = initialIndex + it
        if (col == indices[index]) return data[index]
    }
    return 0
}

/**
 * @return 返回一个 Triple
 *         first - data 数组，按行存放所有非零元素
 *         second - indices 数组，所有非零元素的列索引
 *         third - indptr 数组，indices pointer，第 i 个元素记录了前 i - 1 行包含的非零元素的数量
 */
fun matrix2Csr(): Triple<IntArray, IntArray, IntArray> {
    val data = mutableListOf<Int>()
    val indices = mutableListOf<Int>()
    val indptr = mutableListOf<Int>()
    indptr.add(0)
    matrix.forEach { array ->
        array.forEachIndexed { col, item ->
            if (item != 0) {
                data.add(item)
                indices.add(col)
            }
        }
        indptr.add(data.size)
    }
    return Triple(data.toIntArray(), indices.toIntArray(), indptr.toIntArray())
}

fun getItemCountByRow(row: Int, indptr: IntArray): Int {
    return indptr[row + 1] - indptr[row]
}

fun getInitialIndexByRow(row: Int, indptr: IntArray): Int {
    return indptr[row]
}
```

<!-- https://zhuanlan.zhihu.com/p/623366713 -->
<!-- https://blog.csdn.net/weixin_44852067/article/details/129839752 -->
<!-- https://blog.csdn.net/Franklins_Fan/article/details/122528983 -->
<!--
分数的几种 markdown 写法：
$$\frac13$$
$$\frac{1}{3}$$
$${1} \over {3}$$
$${1 \over 3}$$
-->