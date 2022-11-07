---
layout: post
title:  Color.parseColor
date:   2022-11-07 11:00:00 +0800
categories: publish
tags: color
published: true
---

* content
{:toc}

对于 Android UI 校正师来说，把 figma/sketch 上设计师标注的颜色转换到代码中，应该是司空见惯的场景，<br>
layout 布局里可以直接拷贝过去，代码动态设置则往往需要使用 `Color.parseColor`：

```kotlin
@ColorInt val color = Color.parseColor("#4049CB") // 以颜色 #4049CB 为例
textView.setTextColor(color)
```

事实上，对于任意给定颜色字符串，其对应解析的颜色值都是唯一确定的。<br>
那么，有没有一种跳过 parse 直接确定对应颜色值的方法呢？<br>
答案是肯定的，只需要将对应的颜色写成十六进制的 Long 类型，然后强转成 Int 即可：

```kotlin
@ColorInt private val color = 0xFF4049CBL.toInt() // equals to Color.parseColor("#4049CB")
textView.setTextColor(color)
```

**Note** 对于没有标注透明度的颜色必须补齐透明度前缀 0xFF，`0xFF4049CBL.toInt()` 和 `0x4049CBL.toInt()` 是两种完全不同的颜色，
后者等同于 `0x004049CBL.toInt()`，对应的颜色字符串为 #004049CB。

其实，这就是 Color.parseColor 方法的流程，只不过我们人工将其提前化。这样做有如下好处：

* 提前确定了具体的 ColorInt 值，可以使用 const val 修饰符，做到编译期替换
* 省去了运行时解析，代码执行效率更高了

当然，编译期替换本身也就是为了省去运行时计算，所以也可以说好处就一点：为了执行效率更高。<br>

`Color.parseColor` 唯一的用处在于无法确定颜色字符串的时候，比如其他接口传入一个 color string 的变量。
