---
layout: post
title: 什么是协程
date: 2023-07-15 09:30:00 +0800
categories: coroutine
tags: coroutine
published: false
---

* content
{:toc}

## 什么是协程

coroutine 实际上是两个单词的组合 “co” (cooperative) 和 “routine” (function)。<br>
顾名思义，协程就是协作式的程序。


出去面试，但凡简历提到熟悉协程或者使用过协程，几乎都会被问到这个问题。

官方的 slogan 说协程是轻量级的线程。

没问题但好像又哪里不对劲。
说它没问题，是因为协程确实跟线程很像，诸如 join、await 之类异曲同工的设计；同时协程又不同于线程，它没有线程的切换[^1]，省掉了 cpu 切换线程的状态暂停和恢复[^2]，所以协程比线程更轻。

但是，如果我作为一个面试官，这显然是一个不希望听到的回答，没有听到任何有助于深入理解协程的信息。<br>
一般我都把这类统称为薛定谔式定义，说了又好像什么都没说。体制内经常听政府报告或者领导发言的同学应该不陌生。

## 中断

要理解协程就得先理解协程的核心，协程的核心是什么？<br>
对我来说，协程的核心是中断（官方的说法是挂起，我更愿意称之为中断，whatever 听得懂就行）。
知道了协程是如何实现函数的中断，也就理解了协程。

## 状态机

协程内部实现不是使用普通回调的形式，而是使用状态机来处理不同的中断点，大致的 CPS(Continuation Passing Style) 代码为


## Continuation 代理 

## 协程代码可读性

1. 多平台 kmm
2. 设计迭代

**Footnote**

[^1]: 有同学可能会反驳，kotlin 协程也可以切换线程呀。<br>
      嗯，你说得很对，但那不是协程的核心，那只是方便日常使用额外添加的细枝末叶

[^2]: 相对于 cpu 执行其他普通代码来说，线程切换是个很重的操作

<!-- https://www.educative.io/answers/what-is-a-coroutine -->
<!-- https://amitshekhar.me/blog/callback-to-coroutines-in-kotlin -->
<!-- https://stackoverflow.com/a/48562175 -->
<!-- https://stackoverflow.com/a/62536933 -->
