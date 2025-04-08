---
layout: post
title: 圆形图片
date: 2025-04-08 20:30:00 +0800
categories: Android
tags: Image
published: true
---

* content
{:toc}

在 Android 中圆形图片的设计随处可见，代码实现不外乎两类：

1. 对加载的图片 drawable 进行圆形裁剪
2. 对 ImageView 本身进行圆形裁剪

对于第一类，ImageView 的尺寸保持原形，如果设置了背景的话，背景依然可见。<br>
第二类具体的实现方式又有很多种：ShapeableImageView 以及 ViewOutlineProvider 等

## 使用 Glide 裁剪 drawable
```kotlin
val options = RequestOptions().transform(RoundedCorners(radius))
Glide.with(this).load(imageUrl).apply(options).into(imageView)
```

## ShapeableImageView
```kotlin
// layout.xml
<com.google.android.material.imageview.ShapeableImageView
    android:id="@+id/shapeable_image_view"
    android:layout_width="100dp"
    android:layout_height="100dp"
    app:shapeAppearance="@style/CircleStyle" />
```

```
// style.xml
<style name="CircleStyle">
    <item name="cornerFamily">rounded</item>
    <item name="cornerSize">50%</item>
</style>
```

```kotlin
Glide.with(this).load(imageUrl).into(shapeableImageView)
```

* 使用 ShapeableImageView 需要依赖 material
* ShapeableImageView 不仅可以实现圆形，也可以实现其他形状，还可以设置边框等

shapeAppearance 也可以不使用 xml style，而是代码的方式动态实现：

```kotlin
shapeableImageView.shapeAppearanceModel = ShapeAppearanceModel.builder()
    .setAllCornerSizes(radius).build()
Glide.with(this).load(imageUrl).into(shapeableImageView)
```

## ViewOutlineProvider

```kotlin
imageView.apply {
    setLayerType(View.LAYER_TYPE_SOFTWARE, null)
    clipToOutline = true
    outlineProvider = object : ViewOutlineProvider() {
        override fun getOutline(view: View, outline: Outline) {
            val w = view.width - view.paddingLeft - view.paddingRight
            val h = view.height - view.paddingTop - view.paddingBottom
            val offsetHorizontal = (max(w, h) - h) / 2
            val offsetVertical = (max(w, h) - w) / 2
            val radius = min(w, h) / 2f
            outline.setRoundRect(
                view.paddingLeft + offsetHorizontal,
                view.paddingTop + offsetVertical,
                view.width - view.paddingRight - offsetHorizontal,
                view.height - view.paddingBottom - offsetVertical,
                radius
            )
        }
    }
}
Glide.with(this).load(imageUrl).apply(options).into(imageView)
```

当然，也可以把 outlineProvider 封装成一个单独的类，这样就可以用于其他任何 View，而这正是 ViewOutlineProvider 的强大之处

```kotlin
import android.graphics.Outline
import android.view.View
import android.view.ViewOutlineProvider
import kotlin.math.max
import kotlin.math.min

class RoundedOutlineProvider : ViewOutlineProvider() {
    override fun getOutline(view: View, outline: Outline) {
        val w = view.width - view.paddingLeft - view.paddingRight
        val h = view.height - view.paddingTop - view.paddingBottom
        val offsetHorizontal = (max(w, h) - h) / 2
        val offsetVertical = (max(w, h) - w) / 2
        val radius = min(w, h) / 2f
        outline.setRoundRect(
            view.paddingLeft + offsetHorizontal,
            view.paddingTop + offsetVertical,
            view.width - view.paddingRight - offsetHorizontal,
            view.height - view.paddingBottom - offsetVertical,
            radius
        )
    }
}
```

需要注意的是，ViewOutlineProvider 不能在 API 21（Android 5.0）以下使用。