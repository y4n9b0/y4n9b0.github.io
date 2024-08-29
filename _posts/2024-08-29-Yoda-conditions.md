---
layout: post
title: Yoda conditions
date: 2024-08-29 10:30:00 +0800
categories: program
tags: condition
published: true
---

* content
{:toc}

尤达条件式是一种计算机编程中的编程风格，在此风格中表达式的两个部分与条件语句中的顺序将会对调, 并且表达式的常量部分将会放在条件语句的左侧。这种风格的名称来自于星际大战的绝地大师尤达，他说着非标准语法的英语。

通常计算机编程中的条件语句会写成：

```
if (xx == 9527) { /* ... */ }
// Reads like: "If the xx is equal to 9527..."
```

在尤达条件式中对于相同的条件语句会反转过来：

```
if (9527 == xx) { /* ... */ }
// Reads like: "If 9527 equals the xx..."
```

常量会放在比较运算符的左侧，而右侧会写入测试常量的变量。这个次序和尤达的非标准口语风格非常相似，类似于宾主动语序。

## 优点

**1. 可以防止条件判断误写为赋值操作**

```
if (xx = 9527) { /* ... */ }
// This assigns 9527 to xx instead of evaluating the desired condition
```

如上所示，在试图进行大小比较时漏写了一个等号，就会演变成赋值操作。而很多编程语言都支持自动推导赋值语句布尔值，当赋值语句推导的布尔值为 true 时(绝大多数语言只要赋值不为 0 即会判为 true)依旧会继续执行条件满足语句。

如果使用尤达条件式，漏写等号就会变成将一个变量赋值给一个常量，这会触发编译器报错，可以及时提醒等号遗漏：

```
if (9527 = xx) { /* ... */ }
// This is a syntax error and will not compile
```

**2. 避免空指针异常**

```
String xx = null;
if (xx.equals("foobar")) { /* ... */ }
// This causes a NullPointerException in Java
```

尤达条件式：
```
String xx = null;
if ("foobar".equals(xx)) { /* ... */ }
// This is false, as expected
```

## 缺点

1. 尤达条件式主宾倒置，缺乏可读性。
2. 有相当多的语言不支持条件表达式赋值，在这些语言场景下尤达条件式失去了优点 1。严格来说，这算不上缺点。
3. 尤达条件式的优点 2 避免的空指针异常，仍然可能在后续代码中触发，违背了 fail fast。

目前的现状是很多老外都比较反对尤达条件式，就我自己来说还是非常赞同尤达条件式的。
特别是尤达条件式的缺点 1 对于 native language 非英语的人来说，问题不大；而且一个条件判断表达式，往往都比较简短，根本不存在阅读障碍。

<!-- https://zh.wikipedia.org/wiki/%E5%B0%A4%E9%81%94%E6%A2%9D%E4%BB%B6%E5%BC%8F -->
<!-- https://knowthecode.io/yoda-conditions-yoda-not-yoda -->
<!-- https://developer.woocommerce.com/2022/08/11/goodbye-yoda-conditions/ -->
<!-- https://dev.to/greg0ire/why-using-yoda-conditions-you-should-probably-not -->