---
layout: post
title: Android 启动应用时检测剪切板
date: 2025-12-20 14:00:00 +0800
categories: android
tags: clipboard
published: true
---

* content
{:toc}

## 一个需求

在启动应用的时候（冷启动/热启动）检测剪切板内容，发现 H5 链接进行跳转。

## Approach 1

```kotlin
import android.app.Application
import android.content.ClipData
import android.content.ClipboardManager
import android.content.Context
import android.os.Build
import android.util.Log
import android.util.Patterns
import androidx.lifecycle.DefaultLifecycleObserver
import androidx.lifecycle.LifecycleOwner
import androidx.lifecycle.ProcessLifecycleOwner
import java.util.regex.Pattern
import kotlin.properties.Delegates

object ClipboardChecker : DefaultLifecycleObserver {

    private const val TAG = "ClipboardChecker"

    // 应用当前是否处于后台
    private var inBackground = false
    // 本次启动是否是冷启动
    private var isColdStart = true
    // 本次启动是否已经检测过剪切板
    private var hasCurrentStartChecked = false

    private var application: Application by Delegates.notNull()

    fun init(application: Application) {
        this.application = application
        ProcessLifecycleOwner.get().lifecycle.addObserver(this)
    }

    /**
     * 检查剪切板内容
     *
     * @param pattern 匹配规则
     * @param onMatch 匹配命中的结果（列表），返回值表示是否需要清空剪切板
     */
    fun check(
        pattern: Pattern = Patterns.WEB_URL,
        onMatch: (List<String>) -> Boolean = { false },
    ) {
        if (hasCurrentStartChecked) {
            Log.d(TAG, "${if (isColdStart) "Cold start" else "Hot start"} has checked, ignore!")
            return
        }
        hasCurrentStartChecked = true
        val clipboard = application.getSystemService(Context.CLIPBOARD_SERVICE) as? ClipboardManager
        if (clipboard == null) {
            Log.d(TAG, "clipboard is null, ignore!")
            return
        }
        val clipData = clipboard.primaryClip
        if (clipData == null) {
            Log.d(TAG, "ClipData is null, ignore!")
            return
        }
        if (clipData.itemCount == 0) {
            Log.d(TAG, "ClipData is empty, ignore!")
            return
        }
        val text = (0 until clipData.itemCount).firstNotNullOfOrNull { i ->
            clipData.getItemAt(i).coerceToText(application)?.takeIf { it.isNotEmpty() }
        }
        if (text == null) {
            Log.d(TAG, "No valid clipData item, ignore!")
            return
        }
        Log.d(TAG, "clipboardText:\n$text")
        if (onMatch(match(text, pattern))) {
            Log.d(TAG, "clear clipboard")
            // 不要使用 Android P+ 的系统方法 clipboard.clearPrimaryClip()，部分手机上会 NullPointerException 
            val emptyClip = ClipData.newPlainText("", "")
            clipboard.setPrimaryClip(emptyClip)
        }
    }

    override fun onStart(owner: LifecycleOwner) {
        super.onStart(owner)
        hasCurrentStartChecked = false
        if (inBackground) {
            inBackground = false
            isColdStart = false
        }
    }

    override fun onStop(owner: LifecycleOwner) {
        super.onStop(owner)
        inBackground = true
    }

    private fun match(text: CharSequence, pattern: Pattern): List<String> {
        val result = mutableListOf<String>()
        val matcher = pattern.matcher(text)
        while (matcher.find()) {
            result.add(matcher.group())
        }
        return result
    }
}
```

封装一个 ClipboardChecker，在 Application 里调用 ClipboardChecker.init 方法初始化，然后在 BaseActivity（所有 Activity 的基类）
 onWindowFocusChanged 方法里判断是否获取窗口焦点调用 check 方法：

```kotlin
    override fun onWindowFocusChanged(hasFocus: Boolean) {
        super.onWindowFocusChanged(hasFocus)
        Log.d("BaseActivity", "hasFocus:$hasFocus")
        if (hasFocus) {
            ClipboardChecker.check { list ->
                list.firstOrNull { it.isNotEmpty() }?.apply {
                    Log.d("BaseActivity", "found clipboard url: $this")
                    // do your stuff
                }
                true
            }
        }
    }
```

需要注意一点，跳转 H5 链接使用应用内的 WebView 打开，如果使用外部浏览器打开又会触发当前应用前后台切换。
当然，实际业务中也没人会使用外部浏览器打开，毕竟跳转链接的目的是想直达跟自己应用相关的业务活动之类。

另外，实际测试中，部分 Android 手机在退回后台什么都不做，然后又快速恢复的情况下并不会触发前后台切换，只有在等待几分钟或者切换其他应用时才会回调进入后台，这个属于国产手机系统优化魔改行为，无解。

### 为什么不在 onResume 方法里检测

在 Android 10+ 对剪切板有改动：当应用没有获取到焦点的时候无法读取剪切板内容，所以在 onResume 里直接获取大概率无法成功，一种方式是延迟获取，
更加正确的做法是在 onWindowFocusChanged 方法里判断，hasFocus 为 true 时再去获取。

### 为什么不用 ClipboardManager.OnPrimaryClipChangedListener

Android 10+ 对后台监听剪切板变化同样有限制，且会引入更多不可控行为。

## Approach 2

上面的方式有一个业务问题，在冷启动的时候要求稳定进入首页（MainActivity）再去检测跳转，首页之前可能还有 SplashActivity 等开屏页面需要展示，
于是改为如下方式，在 check 方法增加参数 checkWhenColdStart 指定是在冷启动还是热启动检测：

```kotlin
import android.app.Application
import android.content.ClipData
import android.content.ClipboardManager
import android.content.Context
import android.os.Build
import android.util.Log
import android.util.Patterns
import androidx.lifecycle.DefaultLifecycleObserver
import androidx.lifecycle.LifecycleOwner
import androidx.lifecycle.ProcessLifecycleOwner
import java.util.regex.Pattern
import kotlin.properties.Delegates

object ClipboardChecker : DefaultLifecycleObserver {

    private const val TAG = "ClipboardChecker"

    // 应用当前是否处于后台
    private var inBackground = false
    // 本次启动是否是冷启动
    private var isColdStart = true
    // 本次启动是否已经检测过剪切板
    private var hasCurrentStartChecked = false

    private var application: Application by Delegates.notNull()

    fun init(application: Application) {
        this.application = application
        ProcessLifecycleOwner.get().lifecycle.addObserver(this)
    }

    /**
     * 检查剪切板内容
     *
     * @param checkWhenColdStart true-只在冷启动检测，false-只在热启动检测
     * @param pattern 匹配规则
     * @param onMatch 匹配命中的结果（列表），返回值表示是否需要清空剪切板
     */
    fun check(
        checkWhenColdStart: Boolean = false,
        pattern: Pattern = Patterns.WEB_URL,
        onMatch: (List<String>) -> Boolean = { false },
    ) {
        if (checkWhenColdStart != isColdStart) {
            if (checkWhenColdStart) {
                Log.d(TAG, "Cold start check on hot start, ignore!")
            } else {
                Log.d(TAG, "Hot start check on cold start, ignore!")
            }
            return
        }
        if (hasCurrentStartChecked) {
            Log.d(TAG, "${if (isColdStart) "Cold start" else "Hot start"} has checked, ignore!")
            return
        }
        hasCurrentStartChecked = true
        val clipboard = application.getSystemService(Context.CLIPBOARD_SERVICE) as? ClipboardManager
        if (clipboard == null) {
            Log.d(TAG, "clipboard is null, ignore!")
            return
        }
        val clipData = clipboard.primaryClip
        if (clipData == null) {
            Log.d(TAG, "ClipData is null, ignore!")
            return
        }
        if (clipData.itemCount == 0) {
            Log.d(TAG, "ClipData is empty, ignore!")
            return
        }
        val text = (0 until clipData.itemCount).firstNotNullOfOrNull { i ->
            clipData.getItemAt(i).coerceToText(application)?.takeIf { it.isNotEmpty() }
        }
        if (text == null) {
            Log.d(TAG, "No valid clipData item, ignore!")
            return
        }
        Log.d(TAG, "clipboardText:\n$text")
        if (onMatch(match(text, pattern))) {
            Log.d(TAG, "clear clipboard")
            // 不要使用 Android P+ 的系统方法 clipboard.clearPrimaryClip()，部分手机上会 NullPointerException 
            val emptyClip = ClipData.newPlainText("", "")
            clipboard.setPrimaryClip(emptyClip)
        }
    }

    override fun onStart(owner: LifecycleOwner) {
        super.onStart(owner)
        hasCurrentStartChecked = false
        if (inBackground) {
            inBackground = false
            isColdStart = false
        }
    }

    override fun onStop(owner: LifecycleOwner) {
        super.onStop(owner)
        inBackground = true
    }

    private fun match(text: CharSequence, pattern: Pattern): List<String> {
        val result = mutableListOf<String>()
        val matcher = pattern.matcher(text)
        while (matcher.find()) {
            result.add(matcher.group())
        }
        return result
    }
}
```

然后在 MainActivity 里调用冷启动检测：

```kotlin
    override fun onWindowFocusChanged(hasFocus: Boolean) {
        // 基类调用不可少，首页也需要在热启动场景检测，咱们需要保证的是冷启动场景只在首页检测，而不是首页只检测冷启动
        super.onWindowFocusChanged(hasFocus) 
        Log.d("MainActivity", "hasFocus:$hasFocus")
        if (hasFocus) {
            ClipboardChecker.check(checkWhenColdStart = true) { list ->
                list.firstOrNull { it.isNotEmpty() }?.apply {
                    Log.d("MainActivity", "found clipboard url: $this")
                    // do your stuff
                }
                true
            }
        }
    }
```

BaseActivity 调用 check 方法依然保持不变，只是变成了在热启动场景才检测（checkWhenColdStart 默认值为 false）。

如果想要某些 Activity 页面不检测，比如大多数国产流氓 app 在热启动的时候弹个 5 秒广告页面，希望用户在观赏完广告后再检测跳转，
那么只需要重写广告页面的 onWindowFocusChanged 方法，不调用 super.onWindowFocusChanged(hasFocus) 即可。

如果基类 BaseActivity onWindowFocusChanged 方法还有其他业务，就不能通过屏蔽 super.onWindowFocusChanged(hasFocus) 来解决了，
这个时候需要在 BaseActivity 添加一个 Boolean 参数控制是否检测剪切板，然后在不需要检测的广告 Activity 页面将其置为 false 不检测即可。

如果并不是所有 Activity 都继承自 BaseActivity 怎么办？<br>
还能怎么办，要么继承 BaseActivity，要么一个页面一个页面 onWindowFocusChanged 加呗。

如果···<br>
打住，就你喵的事儿多！