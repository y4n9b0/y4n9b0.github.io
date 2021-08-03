---
layout: post
title: Visibility Extension of Android View
date: 2021-08-03 17:00:00 +0800
categories: Kotlin
tags: extension-function
published: true
---

* content
{:toc}

如下的扩展方法应该是大家都很容易写出来的：

```kotlin
fun View.visible(): View = apply { visibility = View.VISIBLE }

fun View.invisible(): View = apply { visibility = View.INVISIBLE }

fun View.gone(): View = apply { visibility = View.GONE }
```

实际业务中，绝大部分情况（甚至可以说全部）都是需要根据某种条件判断再决定是否需要显示 view，所以很自然地想到将判断条件一起加入扩展：

```kotlin
fun View.visibleIf(visible: Boolean): View = apply {
    visibility = if (visible) View.VISIBLE else View.GONE
}
```

但是如此一来，一个判断条件最多只能涵盖两种情况，无法完全覆盖 View.VISIBLE、View.INVISIBLE、View.GONE 三种 case，
上面的代码我们采用了实际应用更多的 View.GONE，而无视了 View.INVISIBLE。想做到 case 完备，要么增加判断条件，要么增加扩展 api。

```kotlin
fun View.visibleIf(visible: Boolean, elseTakeGone: Boolean = true): View = apply {
    visibility = if (visible) View.VISIBLE else if (elseTakeGone) View.GONE else View.INVISIBLE
}
```

如上代码增加判断条件，`elseTakeGone` 用于决定 else 分支使用 INVISIBLE 还是 GONE，默认值 true 使用 GONE。该方法存在的问题是，
如果使用 INVISIBLE，则调用方的代码类似这样 `view.visibleIf(bool, false)`，两个同样 Boolean 类型的参数一定程度上容易造成理解歧义。

```kotlin
@IntDef(value = [View.VISIBLE, View.INVISIBLE, View.GONE])
@Retention(AnnotationRetention.SOURCE)
annotation class Visibility

fun View.visibleIf(visible: Boolean, @Visibility `else`: Int = View.GONE): View = apply {
    visibility = if (visible) View.VISIBLE else `else`
}
```

改进后的方法使用 visibility 的 Int 值作为第二个参数，直观了很多。但是该方法也有个明显的问题，
Kotlin 代码并不会对 IntDef 的进行检查告警[discuss-7029](https://discuss.kotlinlang.org/t/intdef-and-stringdef-not-being-checked-at-compile-time/7029){:target="_blank"}，
这就导致调用方可以传 View.VISIBLE、View.INVISIBLE、View.GONE 之外的任何整数，尽管继续执行也不会报错，但这不是我们希望看到的。

```kotlin
enum class Visibility(val viewVis: Int) {
    Visible(View.VISIBLE),
    Invisible(View.INVISIBLE),
    Gone(View.GONE)
}

fun View.visibleIf(visible: Boolean, `else`: Visibility = Visibility.Gone): View = apply {
    visibility = if (visible) View.VISIBLE else `else`.viewVis
}
```

如上的代码使用枚举完美地解决了 IntDef 无法告警的问题，同时我们也看到代码更加复杂了，而且枚举会带来一定程度的性能问题（但据说简单的枚举是可以被编译器优化成整型的）。
相比起来个人更倾向于上面的 IntDef，简单明了，虽然我们没法做到禁止调用方传其他整型值，但至少调用方是清楚自己传了不应该传的值。程序的正确性更应该由编写的人自行保证 而不是交由编译器，IDE 之类的工具只是尽可能多地给程序员提供帮助。

上面的都是通过增加参数来解决，下面说说增加 api 的思路：

```kotlin
fun View.invisibleIf(invisible: Boolean): View = apply {
    visibility = if (invisible) View.INVISIBLE else View.VISIBLE
}

fun View.goneIf(gone: Boolean): View = apply {
    visibility = if (gone) View.GONE else View.VISIBLE
}
```

由于可见都是 View.VISIBLE，而不可见有 View.INVISIBLE 和 View.GONE 两种，故我们反过来定义 api，使用 `invisibleIf` 和 `goneIf`。该方案有两个缺点，一是 api 个数增加了（貌似废话），
优秀 library 的设计准则之一应该是用足够少的 api 涵盖足够多的使用 case；另一个缺点是由于反过来定义，思维上有点别扭，比较直观的思维都是考虑什么时候 view 可见。

啰里八嗦了一大堆，貌似最后也没个完美的方案，但生活不就是这样么，哪有什么完美的事物。就这个扩展来说，个人还是更倾向使用 IntDef 的单个 api 方案。
