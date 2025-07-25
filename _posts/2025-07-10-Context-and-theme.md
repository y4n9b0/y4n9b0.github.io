---
layout: post
title: Context and Theme
date: 2025-07-10 11:00:00 +0800
categories: Android
tags: Android
published: true
---

* content
{:toc}

## Material Components Theme

写了一个自定义 View，其内部组合使用了 ShapeableImageView，然后发现日志中有以下错误：

![log]({{ '/styles/images/context-and-theme/log.jpeg' | prepend: site.baseurl }}){:width="700" height="auto"} 

报错日志说必须使用 Theme.AppCompat 或其子类，当我仔细检查后发现使用的 theme 是继承自 Theme.AppCompat.Light，理论上不应该。

其实不然，这里有个深坑，ShapeableImageView 属于 Material Components Library，需要依赖 Material Components 的属性，AppCompat 无法提供，它要求使用的是：Theme.MaterialComponents（或其子类）而不是 Theme.AppCompat。

为什么报错信息会提到 “AppCompat widget”？<br>
这是因为 MaterialComponents 库是基于 AppCompat 扩展而来，它的很多 View 内部都继承了 AppCompatImageView 等 AppCompat 基类；
所以它在检查当前主题是否兼容时，会沿用“AppCompat widget”这一措辞。

## difference of Context between Application and Activity

### theme

是不是以为只是将应用 Theme 的继承由 Theme.AppCompat.Light 替换为 Theme.MaterialComponents.Light 就完了，如果这么简单的话也就不会有这篇文章。

事实上当替换为 Material Components Theme 后，错误并没有消除。
这里就不得不提到我的使用方式了，在代码动态构造该 View 的时候传入了 Appliciont 作为 context，之所以传入 Appliciont 是因为，该控件需要传入一个三方库显示用，而这个三方库需要在 Appliciont 里进行初始化注册。

如果仅仅是在 Activity 里动态构造是一点问题没有，这是因为 Activity 继承自 ContextThemeWrapper，而 Appliciont 继承自 ContextWrapper。
相信到这里大多数人已经明白了，Application 作为 context 构造自定义 View 是不带 theme 的，该自定义 View 内部使用了 Material Components 的组件，
那么就无法通过  Material Components Theme 获取到对应的 Material Components 属性。

由于 ContextThemeWrapper 是继承自 ContextWrapper，且可以通过构造函数传入 Theme，所以解决方案也很简单：

```kotlin
// old
val MyView = MyView(application)

// new
val MyView = MyView(ContextThemeWrapper(application, R.style.My_App_Theme))
```

### Bridge themes

Theme.AppCompat.Light 替换为 Theme.MaterialComponents.Light 之后，又发现某些 Button 的背景显示异常。
原因是当应用使用 Theme.MaterialComponents.* 时，Button 会在运行时被替换为 com.google.android.material.button.MaterialButton，
MaterialButton 的 style 包含了默认的 backgroundTint（使用 attr/colorPrimary），导致 Button 设置的背景被 tinted。
解决方式有三：
1. 设置 Button `app:backgroundTint="@null"`
2. 使用 `androidx.appcompat.widget.AppCompatButton` 替换 Button
3. 使用 `Theme.MaterialComponents.*.Bridge` 替换 `Theme.MaterialComponents.*`，
   Bridge themes 继承自 AppCompat themes，同时又定义了 Material Components theme attributes，
   部分 Bridge themes 如下：
   ```
   Theme.MaterialComponents.Bridge
   Theme.MaterialComponents.Light.Bridge
   Theme.MaterialComponents.NoActionBar.Bridge
   Theme.MaterialComponents.Light.NoActionBar.Bridge
   Theme.MaterialComponents.Light.DarkActionBar.Bridge  
   ```
   <!-- https://stackoverflow.com/a/63331089 -->
   <!-- https://stackoverflow.com/a/67466323 -->

通常建议使用方式 3，一方面改动最小，另一方面方式一和二都只是解决了 Button 这一控件的问题，还可能有其他控件存在 theme 属性问题。

### token

Application 和 Activity 作为 context 的另一个区别就是，Application 不能用于 Dialog，否则会 BadTokenException。

因为 Activity 获取到的 WindManager 服务，即 WindowManagerImpl 的 mParentWindow 属性不为空，而 Application 获取的 mParentWindow 属性为 null，导致 getWindowToken 时，获取 token 为 null。

当然，理论上也可以先试用 Application 作为 context 参数传入 Dialog，然后在 show 之前将 Activity 的 window token 赋值给 Dialog

```kotlin
// do in Activity
val dialog = MyDialog(applicationContext)
dialog.window?.attributes?.token = window.attributes.token
dialog.show()
```

但是，强烈不建议这样使用，一是担心后续还会有其他问题；二是赋值 token 就需要 Activity，已经能拿到 Activity 了何必还要用 Application context。

## finally

* 虽然 Material Components 组件没有找到对应的属性，但应用并不崩溃，系统只是抛出了 error 级别日志，然后继续运行，这也导致很多人忽略了这些细节。理所当然地，如果这种场景下设置了对应的属性那肯定不会生效。
* Framework 的报错日志并不完全靠谱，甚至有时候会误导，多搜索、多思考、怀疑一切。