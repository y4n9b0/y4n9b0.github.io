---
layout: post
title: B-Spline Curve
date: 2026-04-16 16:30:00 +0800
categories: animation
tags: bezier
published: true
---

* content
{:toc}

## B 样条曲线

**贝塞尔曲线**<br>
Bezier 曲线一条由 n+1 个控制点决定的 n 阶曲线。具有如下特点：

* 移动一个点，整条曲线都会变
* 阶数爆炸：n+1 个控制点 → n次多项式

**B-Spline**<br>
全称 Basis Spline，即 B 样条曲线，是对贝塞尔曲线的一般化，它的诞生是为了解决高阶贝塞尔曲线的一系列缺陷。

* 改一个控制点 → 只影响局部
* 固定阶数（由 degree 决定）
* 可调“形状”的能力（knot 控制）
* 天然支持拼接（连续性自动保证）

**样条**<br>
在计算机处理之前，设计一般是通过在纸上的手绘完成，该过程需要借助许多绘画工具。
样条（Spline）是早期的工人和技术员（特别是造船业），用来画光滑形状的工具，它是一根柔软但有弹性的长条物，有些像尺子。
使用方法很简单，将两端和几个点用钉子固定之后，便可以产生顺滑的曲线。

**节点向量（Knots）**<br>
// TODO

## B 样条插值器

```kotlin
import android.view.animation.Interpolator

class BSplineInterpolator(
    private val points: List<Float>,
    private val degree: Int = 3,
    private val knots: List<Float>? = null
) : Interpolator {

    private val n = points.lastIndex

    init {
        require(degree >= 1) { "degree 必须 >= 1" }
        require(points.size >= degree + 1) { "B-Spline 需要至少 degree+1 个控制点" }

        knots?.let { u ->
            require(u.size == points.size + degree + 1) {
                "knots 长度必须为 controlPoints.size + degree + 1"
            }
            require(u.zipWithNext().all { (a, b) -> a <= b }) {
                "knots 必须是非递减序列"
            }
            require(u[degree] < u[n + 1]) {
                "无效 knot 区间: knots[degree] 必须小于 knots[n+1]"
            }
        }
    }

    private val actualKnots: FloatArray by lazy {
        knots?.toFloatArray() ?: buildUniformClampedKnots()
    }

    override fun getInterpolation(input: Float): Float {
        val x = input.coerceIn(0f, 1f)
        val start = actualKnots[degree]
        val end = actualKnots[n + 1]
        val t = start + x * (end - start)
        return deBoor(t)
    }

    private fun deBoor(t: Float): Float {
        val i = findSpan(t)
        val d = FloatArray(degree + 1) { points[i - degree + it] }

        for (r in 1..degree) {
            for (j in degree downTo r) {
                val left = i - degree + j
                val right = i + j - r + 1
                val denom = actualKnots[right] - actualKnots[left]
                val alpha = if (denom == 0f) 0f else (t - actualKnots[left]) / denom
                d[j] = (1f - alpha) * d[j - 1] + alpha * d[j]
            }
        }
        return d[degree]
    }

    private fun findSpan(t: Float): Int {
        if (t <= actualKnots[degree]) return degree
        if (t >= actualKnots[n + 1]) return n

        if (knots == null) {
            val u = (t - actualKnots[degree]) / (actualKnots[n + 1] - actualKnots[degree])
            val span = (u * (n - degree + 1)).toInt() + degree
            return span.coerceIn(degree, n)
        }

        var low = degree
        var high = n + 1
        while (high - low > 1) {
            val mid = (low + high) ushr 1
            if (t < actualKnots[mid]) {
                high = mid
            } else {
                low = mid
            }
        }
        return low
    }

    private fun buildUniformClampedKnots(): FloatArray {
        val knotCount = points.size + degree + 1
        return FloatArray(knotCount) { i ->
            when {
                i <= degree -> 0f
                i >= n + 1 -> 1f
                else -> (i - degree).toFloat() / (n - degree + 1)
            }
        }
    }
}
```

## de Boor 算法

德布尔算法 // TODO

## NURBS

* NU-nouniform，即非均匀，指有效节点向量区间距离非减不相等的。
* R-rational，有理，指权值（weight）各不相等。 权值代表各控制点对某一个拟合点的影响力，这里也可以发现普通的样条来说，各控制点对于同一个拟合点的影响力是相等的。
* BS-BSpline，B 样条。

全称合起来就是控制点影响力和节点向量都参差不齐的 B 样条。

NURBS 的发展始于 1950 年代，当时的工程师们正为缺少一种数学工具能在设计中精确地定义自由曲面而苦恼，为此他们在很长的时间里只能依赖于实体模型。
受大西洋彼岸的贝塞尔等人启发，现任职于 solid Edge 的 Dr.Ken.Versprille 于 1975 年在其博士学位论文 Computer-Aided Design Applications of the Rational B-Spline Approximation form 中首次提出 NURBS 的完整推导。

1991 年，国际标准化组织（ISO）颁布的工业产品数据交换标准 STEP 中，把 NURBS 作为定义工业产品几何形状的唯一数学方法。<br>
1992 年，国际标准化组织又将 NURBS 纳入到规定独立于设备的交互图形编程接口的国际标准 PHIGS（程序员层次交互图形系统）中，作为 PHIGS Plus 的扩充部分。
Bezier、有理B ezier、均匀 B 样条和非均匀 B 样条都被统一到 NURBS 中。

<!-- https://zhuanlan.zhihu.com/p/528642680 -->
<!-- https://zhuanlan.zhihu.com/p/696907070 -->