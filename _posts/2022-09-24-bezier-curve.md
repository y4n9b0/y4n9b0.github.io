---
layout: post
title:  Bezier curve
date:   2022-09-24 16:30:00 +0800
categories: animation
tags: bezier
published: true
---

* content
{:toc}

## 前言

最近搞动画，开发出来的动效跟设计师给视频 demo 不太一样，整了一个多小时，才把坑填平（就是下文的 EaseCubicInterpolator 插值器实例不能复用于多个动画）。<br>
老了，趁周末加班记一下，免得以后再掉进去

## 系统 API 版本（最佳写法、强烈推荐）

支持一二三阶贝塞尔曲线插值器，全部委托给系统 API，高效、稳定、同一插值器实例可复用。<br>
如果一定要说有什么缺点，那就是无法轻易看到 native 层的具体插值实现。

```kotlin
package com.y4n9b0.animation.interpolator

import android.graphics.PointF
import android.view.animation.Interpolator
import android.view.animation.LinearInterpolator
import androidx.annotation.FloatRange
import androidx.core.view.animation.PathInterpolatorCompat

/**
 * The end points (0, 0) and (1, 1) are assumed.
 *
 * 一阶（线性）贝塞尔曲线, no control points
 * https://en.wikipedia.org/wiki/Bézier_curve#Linear_Bézier_curves
 * [android.view.animation.LinearInterpolator]
 *
 * 二阶（平方）贝塞尔曲线, one control point (controlX, controlY)
 * https://en.wikipedia.org/wiki/Bézier_curve#Quadratic_Bézier_curve
 * 三阶（立方）贝塞尔曲线, two control points (controlX1, controlY1) and (controlX2, controlY2)
 * https://en.wikipedia.org/wiki/Bézier_curve#Cubic_Bézier_curves
 * [androidx.core.view.animation.PathInterpolatorCompat.create]
 */
class BezierInterpolator : Interpolator {

    private val impl: Interpolator

    constructor() {
        impl = LinearInterpolator()
    }

    constructor(controlX: Float, controlY: Float) {
        impl = PathInterpolatorCompat.create(controlX, controlY)
    }

    constructor(controlPoint: PointF) : this(controlPoint.x, controlPoint.y)

    constructor(controlX1: Float, controlY1: Float, controlX2: Float, controlY2: Float) {
        impl = PathInterpolatorCompat.create(controlX1, controlY1, controlX2, controlY2)
    }

    constructor(controlPoint1: PointF, controlPoint2: PointF) : this(
        controlPoint1.x,
        controlPoint1.y,
        controlPoint2.x,
        controlPoint2.y
    )

    override fun getInterpolation(@FloatRange(from = 0.0, to = 1.0) input: Float) =
        impl.getInterpolation(input)
}
```

## 牛顿迭代法

* 优点：高效、精确，同一插值器实例可复用。上面的系统 API 版本在 Android 32 上的默认精度是 0.002f [PathInterpolator]，而牛顿迭代理论上只要迭代次数够多，精度可以趋近理论值。代码中的 magic number 10 就是最大迭代次数（绝大多数情况下迭代 2-3 次已经足够了）

#### 牛顿迭代三阶贝塞尔曲线
```kotlin
package com.y4n9b0.animation.interpolator

import android.graphics.PointF
import android.view.animation.Interpolator
import androidx.annotation.FloatRange
import kotlin.math.abs

/**
 * 三阶（立方）贝塞尔曲线, two control points (controlX1, controlY1) and (controlX2, controlY2)
 * https://en.wikipedia.org/wiki/Bézier_curve#Cubic_Bézier_curves
 * [androidx.core.view.animation.PathInterpolatorCompat.create]
 *
 * 端点是 (0, 0) 和 (1, 1)，构造函数的参数是两个控制点的 x 和 y 坐标
 */
class CubicBezierInterpolator(
    private val controlPoint1: PointF,
    private val controlPoint2: PointF,
    @FloatRange(from = 0.0) private val precision: Float = 0.001f
) : Interpolator {

    private val pointA: PointF = PointF()
    private val pointB: PointF = PointF()
    private val pointC: PointF = PointF()

    constructor (
        controlX1: Float,
        controlY1: Float,
        controlX2: Float,
        controlY2: Float,
        @FloatRange(from = 0.0) precision: Float = 0.001f
    ) : this(
        PointF(controlX1, controlY1),
        PointF(controlX2, controlY2),
        precision
    )

    override fun getInterpolation(@FloatRange(from = 0.0, to = 1.0) input: Float): Float {
        return getCoordinateY(getXForTime(input))
    }

    private fun getCoordinateY(time: Float): Float {
        pointC.y = 3 * controlPoint1.y
        pointB.y = 3 * (controlPoint2.y - controlPoint1.y) - pointC.y
        pointA.y = 1 - pointC.y - pointB.y
        return time * (pointC.y + time * (pointB.y + time * pointA.y))
    }

    private fun getXForTime(time: Float): Float {
        var x = time
        for (i in 1 until 10) {
            val z = getCoordinateX(x) - time
            if (abs(z) < precision) break
            x -= z.div(getXDerivate(x))
        }
        return x
    }

    private fun getCoordinateX(time: Float): Float {
        pointC.x = 3 * controlPoint1.x
        pointB.x = 3 * (controlPoint2.x - controlPoint1.x) - pointC.x
        pointA.x = 1 - pointC.x - pointB.x
        return time * (pointC.x + time * (pointB.x + time * pointA.x))
    }

    private fun getXDerivate(time: Float): Float {
        return pointC.x + time * (2 * pointB.x + time * 3 * pointA.x)
    }
}
```

#### 牛顿迭代二阶贝塞尔曲线

```kotlin
package com.y4n9b0.animation.interpolator

import android.graphics.PointF
import android.view.animation.Interpolator
import androidx.annotation.FloatRange
import kotlin.math.abs

/**
 * 二阶（平方）贝塞尔曲线, one control point (controlX, controlY)
 * https://en.wikipedia.org/wiki/Bézier_curve#Quadratic_Bézier_curve
 * [androidx.core.view.animation.PathInterpolatorCompat.create]
 *
 * 端点是 (0, 0) 和 (1, 1)，构造函数的参数是控制点的 x 和 y 坐标
 */
class QuadraticBezierInterpolator(
    private val controlPoint: PointF,
    @FloatRange(from = 0.0) private val precision: Float = 0.001f
) : Interpolator {

    private val pointA: PointF = PointF()
    private val pointB: PointF = PointF()

    constructor (
        controlX: Float,
        controlY: Float,
        @FloatRange(from = 0.0) precision: Float = 0.001f
    ) : this(PointF(controlX, controlY), precision)

    override fun getInterpolation(@FloatRange(from = 0.0, to = 1.0) input: Float): Float {
        return getCoordinateY(getXForTime(input))
    }

    private fun getCoordinateY(time: Float): Float {
        pointB.y = 2 * controlPoint.y
        pointA.y = 1 - pointB.y
        return time * (pointB.y + time * pointA.y)
    }

    private fun getXForTime(time: Float): Float {
        var x = time
        for (i in 1 until 10) {
            val z = getCoordinateX(x) - time
            if (abs(z) < precision) break
            x -= z.div(getXDerivate(x))
        }
        return x
    }

    private fun getCoordinateX(time: Float): Float {
        pointB.x = 2 * controlPoint.x
        pointA.x = 1 - pointB.x
        return time * (pointB.x + time * pointA.x)
    }

    private fun getXDerivate(time: Float): Float {
        return pointB.x + time * 2 * pointA.x
    }
}
```

## github EaseCubicInterpolator（不建议使用）

这种写法比较取巧，通过缓存之前的纪录以达到快速运算，缺点因为有缓存同一个插值器实例不能用于多个动画。<br>
具体代码直接在 github 上搜相关类名即可。

## 一个可以复用但是效率极低的三阶贝塞尔曲线插值器

老夫最开始造的一个轮子，同一个插值器实例可以被多个动画复用，但效率极其低下（精度万分之一时，再插一万个值，把一加7pro都搞 ANR 了），贴出来作为反面教材😝

```kotlin
package com.y4n9b0.animation.interpolator

import android.view.animation.Interpolator
import androidx.annotation.FloatRange
import kotlin.math.ceil

/**
 * 三阶（立方）贝塞尔曲线, two control points (controlX1, controlY1) and (controlX2, controlY2)
 * https://en.wikipedia.org/wiki/Bézier_curve#Cubic_Bézier_curves
 * [androidx.core.view.animation.PathInterpolatorCompat.create]
 *
 * 端点是 (0, 0) 和 (1, 1)，构造函数的参数是两个控制点的 x 和 y 坐标
 */
class CubicBezierInterpolator(
    private val controlX1: Float,
    private val controlY1: Float,
    private val controlX2: Float,
    private val controlY2: Float,
    @FloatRange(from = 0.0) private val precision: Float = 0.001f
) : Interpolator {

    private val accuracy by lazy {
        val v = ceil(1f.div(precision))
        var a = 1
        while (a < v){
            a = a.shl(1)
        }
        a
    }

    override fun getInterpolation(@FloatRange(from = 0.0, to = 1.0) input: Float): Float {
        var t = 0f
        for (i in 0..accuracy) {
            t = i.toFloat().div(accuracy)
            if (cubicCurve(t, 0f, controlX1, controlX2, 1f) >= input) break
        }
        return cubicCurve(t, 0f, controlY1, controlY2, 1f)
    }

    private fun cubicCurve(t: Float, v0: Float, v1: Float, v2: Float, v3: Float): Float {
        val tt = t * t
        val ttt = tt * t
        val u = 1 - t
        val uu = u * u
        val uuu = uu * u
        return uuu * v0 + 3 * uu * t * v1 + 3 * u * tt * v2 + ttt * v3
    }
}
```

## 一个在线查看三阶贝塞尔曲线的网站

[cubic-bezier.com](https://cubic-bezier.com/){:target="_blank"}

<!-- De Casteljau's Algorithm -->
<!-- https://www.cubic.org/docs/bezier.htm -->
<!-- https://en.wikipedia.org/wiki/De_Casteljau%27s_algorithm -->
<!-- https://pages.mtu.edu/~shene/COURSES/cs3621/NOTES/spline/Bezier/de-casteljau.html -->
<!-- https://pages.mtu.edu/~shene/COURSES/cs3621/NOTES/spline/Bezier/de-casteljau-correct.html -->

<!-- Newton's method -->
<!-- https://www.youtube.com/watch?v=1uN8cBGVpfs -->

<!-- Bezier curve -->
<!-- https://math.stackexchange.com/questions/2571471/understanding-of-cubic-b%C3%A9zier-curves-in-one-dimension -->
<!-- https://math.stackexchange.com/questions/26846/is-there-an-explicit-form-for-cubic-b%C3%A9zier-curves -->
<!-- http://st-on-it.blogspot.com/2011/05/calculating-cubic-bezier-function.html -->
<!-- https://en.wikipedia.org/wiki/Bézier_curve#Cubic_Bézier_curves -->
<!-- https://github.com/codesoup/android-cubic-bezier-interpolator -->
<!-- https://github.com/rdallasgray/bez -->
<!-- https://zhuanlan.zhihu.com/p/470453595 -->
<!-- https://cubic-bezier.com/ -->
<!-- https://gamedev.stackexchange.com/questions/5373/moving-ships-between-two-planets-along-a-bezier-missing-some-equations-for-acce -->
<!-- https://en.wikipedia.org/wiki/Cubic_function -->
<!-- https://en.wikipedia.org/wiki/Cubic_equation -->