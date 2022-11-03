---
layout: post
title:  Logcat tag
date:   2022-11-03 10:00:00 +0800
categories: publish
tags: logcat
published: true
---

* content
{:toc}

## filter scope: tag or message

过滤指定 tag（不包含 message）：

```bash
adb logcat -s "TAGNAME1","TAGNAME2"
```

logcat 自带正则表达式过滤(只过滤 message 内容，不包含 tag)：

```bash
adb logcat -e <keyword>
```

使用 grep 过滤整行日志（包含 tag 和 message），关键字大小写敏感：

```bash
adb logcat | grep --line-buffered <keyword> 
```

`grep -v` or `grep --invert-match` 反向查找，输出不包含关键字的日志：

```bash
adb logcat | grep -v <keyword> 
```

## grep buffer

grep 过滤日志重定向到文件或者管道的时候偶尔会出现日志不完整，而过滤日志输出到终端却没有问题。这个怪异的现象跟 grep 内部的 buffer 有关。grep 输出到 terminal 的时候，默认使用 line-buffered。重定向到文件或者管道的时候，使用 fully buffered（大小通常是 4k），只有当 buffer 充满时才会输出，最后 buffer 不满的那部分无法输出便会造成不完整的现象。添加参数 `grep --line-buffered` 强制使用 line-buffered 即可解决。参考 [https://stackoverflow.com/a/70311742](https://stackoverflow.com/a/70311742){:target="_blank"}

## 过滤掉正则表达式匹配的 tag（不展示）

go to **Android Studio > Logcat console > Edit Filter Configuration > Log Tag(regex)** and put this instead
`^(?!(EXCLUDE_TAG1|EXCLUDE_TAG2))`

or

```bash
adb logcat | grep --invert-match 'EXCLUDE_TAG1\|EXCLUDE_TAG2'
```

## 过滤指定进程

```bash
adb logcat --pid=$(adb shell pidof -s com.example.app)
```

## 更多用法见帮助手册

```bash
adb logcat --help
```

<!-- https://stackoverflow.com/questions/6173985/filter-output-in-logcat-by-tagname -->
<!-- https://stackoverflow.com/questions/29619376/how-to-exclude-log-tag-in-logcat-android-studio -->
<!-- https://stackoverflow.com/questions/5511433/how-to-exclude-certain-messages-by-tag-name-using-android-adb-logcat/ -->
<!-- https://developer.android.com/studio/command-line/logcat -->
<!-- https://stackoverflow.com/a/70311742 -->