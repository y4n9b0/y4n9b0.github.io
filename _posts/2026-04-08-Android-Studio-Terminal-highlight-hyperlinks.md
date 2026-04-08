---
layout: post
title: Android Studio Terminal 超链接高亮
date: 2026-04-08 19:45:00 +0800
categories: Android-Studio
tags: Terminal
published: true
---

* content
{:toc}

正常情况下在 Android Studio 自带的 Terminal 里使用 git status 查看文件变动的时候，changed 的文件会显示红色，added 会显示绿色。

![/styles/images/android-studio/git-normal.jpeg]({{'/styles/images/android-studio/git-normal.jpeg'|prepend:site.baseurl}}){:height="151" width="576"}

不知从几何时起，Android Studio 自带的 Terminal 支持超链接高亮，当 git 项目有文件 changed/added 的时候，文件颜色不再是默认的红/绿色，而是超链接的蓝色。

以往只需要扫一眼就知道哪些文件是 changed，哪些是 added，现在需要很吃力的先去看前面的文案颜色，再看具体的文件。对于 git 命令重度患者来说及其不适应。

![/styles/images/android-studio/git-highlight-hyperlink.jpeg]({{'/styles/images/android-studio/git-highlight-hyperlink.jpeg'|prepend:site.baseurl}}){:height="151" width="576"}

在 android studio -> settings -> tools -> terminal 里禁用 Highlight hyperlinks 即可。

![/styles/images/android-studio/android-studio-settings-tools-terminal.jpeg]({{'/styles/images/android-studio/android-studio-settings-tools-terminal.jpeg'|prepend:site.baseurl}}){:height="424" width="576"}

如果修改了 Highlight hyperlinks 设置也可能当时不生效（重启 IDE 也可能没用），诡异的低级 bug。

哦 对了，如果启用了 Terminal 超链接，changed/added 的文件有概率是普通颜色，只有鼠标放上去才显示超链接蓝色，bug+1

![/styles/images/android-studio/git-highlight-hyperlink-bug.jpeg]({{'/styles/images/android-studio/git-highlight-hyperlink-bug.jpeg'|prepend:site.baseurl}}){:height="151" width="576"}