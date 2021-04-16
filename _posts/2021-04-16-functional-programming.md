---
layout: post
title:  functional programming
date:   2021-04-16 11:00:00 +0800
categories: programming
tags: paradigm
published: true
---

* content
{:toc}

## 一、programming paradigm

* 命令式编程 imperative programming
* 声明式编程 declarative programming
* 函数式编程 functional programming

## 二、functional programming

[什么是函数式编程思维？ - nameoverflow](https://www.zhihu.com/question/28292740/answer/100284611){:target="_blank"}

[什么是函数式编程思维？ - 用心阁](https://www.zhihu.com/question/28292740/answer/40336090){:target="_blank"}

## 三、referential transparancy

函数式编程的基本原则是“引用透明性”（referential transparency，简称 RT）。

RT 经常被理解为 "no side-effects"，事实上 RT 有一个精确的定义：<br>
An expression e is referentially transparent if for all programs p every occurrence of e in p can be replaced with the result of evaluating e, without affecting the observable result of p.

说人话就是：一个表达式，对任何包含该表达的程序，都可以使用表达式推导出来的结果进行替换，而不对程序产生可观察的影响，则该表达式就是引用透明的。

比如定义一个字符串常量 `val fuckyou: String = "艹尼玛"`，显而易见 fuckyou 和 "艹尼玛" 在任何程序里都是可以等价替换，并且无副作用，所以是引用透明的。

再来看看 `val currentTime: Long = System.currentTimeMillis()`，currentTime 在每一次调用都是固定不变的，而 System.currentTimeMillis() 每次调用都会产生不同的值，显然跟 currentTime 无法等效替代，所以不是引用透明的。

明白什么是 RT，就入门了。如果能在写程序中坚持 RT，那就是真正的 functional programmer。但是，其实要坚持 RT，很多时候并不容易，可以说会让人很头疼，那么为什么还要坚持 RT 呢？

遵守 RT 往往会使得代码整体更加模块化。每一个模块相互独立，可以单独被测试。我们可以把相对简单的小模块，组织起来解决更复杂的问题，这样更容易确保程序的正确性。总结 RT 约束带来的好处：

* 更快速地找到合理的代码实现方案。
* 更安全，方便地重构代码。
* 模块化，Local reasoning。

## 四、side effects

“等号右边的表达式及左边的值可以等效替换而不使程序的行为发生改变”，而不符合这一特点的代码就称其具有副作用(side effects)。

常见 side effecting 的代码有

* logging
* println
* 抛异常
* 可变状态（mutable state）
* 等等

## 五、functor、applicative、monad

[functors, applicatives, and monads in pictures](https://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html){:target="_blank"}
