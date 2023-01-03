---
layout: post
title:  圆形 ImageView
date:   2022-10-11 16:30:00 +0800
categories: publish
tags: Android
published: true
---

* content
{:toc}

## 前言

Android 圆形图片相信大家都不陌生，圆角图片的加载有两种思路，一种是加载的过程中对Bitmap做裁剪，另一种是Bitmap没有裁剪，但是对ImageView显示的时候做裁剪。

如果圆形图片再加上边框(border)就会稍微麻烦一点，这时候我们可以使用 Android 系统 material 库里的 ShapeableImageView，当然没边框也可以使用，很好很强大。

但是 ShapeableImageView 只支持单一颜色的 border，对于渐变颜色的 border 却是无可奈何。最近就被设计师圆形头像加渐变边框给上了一课，由于时间紧急，就尝试着在 github 上碰碰运气，[CircularImageView](https://github.com/lopspower/CircularImageView){:target="_blank"} 刚好支持渐变边框。但实际使用过程中却存在 Glide 无法成功加载头像的严重 bug，该 bug 非常知名，以至于 Glide 将其直接写在了 github 主页说明里 [Glide compatibility](https://github.com/bumptech/glide#compatibility){:target="_blank"}。尝试过网上的一些解决方案都无效，几经周折最终解决了问题(谨慎怀疑 Glide 在 compatibility 里也是胡说八道，并没有深入去研究问题根源)，记录一下结果。

解决方案有两种，一种是克隆 CircularImageView 源码修改，一种是调整 Glide 的使用方法。如下：

## 法一

CircularImageView onDraw 调用 loadBitmap，loadBitmap 通过 civDrawable 缓存 drawable，并判断缓存的和当前是否一致，不一致再更新，问题就出在这里<br>
移除 drawable 的缓存和比较，loadBitmap 直接更新，Glide 即可正常加载。由于 civDrawable 仅用于缓存比较，所以该变量及其使用地方可以一并删除。
```kotlin
class CircularImageView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : AppCompatImageView(context, attrs, defStyleAttr) {

    // Object used to draw
    private var civImage: Bitmap? = null
    private var civDrawable: Drawable? = null

    init {
        init(context, attrs, defStyleAttr)
    }

    private fun init(context: Context, attrs: AttributeSet?, defStyleAttr: Int) {
        ···
    }

    override fun onDraw(canvas: Canvas) {
        // Load the bitmap
        loadBitmap()

        // Check if civImage isn't null
        if (civImage == null) return
        ···
    }

    private fun loadBitmap() {
        if (civDrawable == drawable) return // 罪魁祸首

        civDrawable = drawable
        civImage = drawableToBitmap(civDrawable)
        updateShader()
    }

    ···
}
```
完整的源码参考 [CircularImageView.kt](https://github.com/y4n9b0/CircularImageView/blob/master/circularimageview/src/main/java/com/mikhaellopez/circularimageview/CircularImageView.kt){:target="_blank"}

## 法二

Glide 的默认用法：
```kotlin
    Glide.with(context)
        .load(imageUrl)
        .placeholder(defaultDrawableResId)
        .into(circularImageView)
```
替换成使用 listener 监听 load drawable 成功，再手动设置进 CircularImageview，即可正常显示：
```kotlin
    Glide.with(context)
        .load(imageUrl)
        .placeholder(defaultDrawableResId)
        .listener(object : RequestListener<Drawable> {
            override fun onLoadFailed(
                e: GlideException?,
                model: Any?,
                target: Target<Drawable>?,
                isFirstResource: Boolean
            ): Boolean {
                // do something while load failed
                return true
            }

            override fun onResourceReady(
                resource: Drawable?,
                model: Any?,
                target: Target<Drawable>?,
                dataSource: DataSource?,
                isFirstResource: Boolean
            ): Boolean {
                circularImageView.setImageDrawable(resource)
                return true
            }
        })
        .submit()
```

<!-- https://mp.weixin.qq.com/s/dExd6EUcvmkUCO-RZHQOWw -->