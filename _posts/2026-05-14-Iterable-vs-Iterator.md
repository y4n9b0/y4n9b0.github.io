---
layout: post
title: Iterable vs Iterator
date: 2026-05-14 11:30:00 +0800
categories: kotlin
tags: for-in
published: true
---

* content
{:toc}

**Iterable 定义**

```kotlin
public actual interface Iterable<out T> {
    /**
     * Returns an iterator over the elements of this object.
     */
    public actual operator fun iterator(): Iterator<T>
}
```

**Iterator 定义**

```kotlin
public actual interface Iterator<out T> {
    /**
     * Returns the next element in the iteration.
     *
     * @throws NoSuchElementException if the iteration has no next element.
     */
    public actual operator fun next(): T

    /**
     * Returns `true` if the iteration has more elements.
     */
    public actual operator fun hasNext(): Boolean
}
```

在 [kotlin 扩展函数奇技淫巧](https://y4n9b0.github.io/2026/03/14/kotlin-awesome-extension-function/){:target="_blank"} 中提到了 Iterable 和 Iterator，我们知道 Iterable 只是提供了一个 `iterator()` 方法并返回一个 Iterator，有没有想过为什么要多此一举包装一个 Iterable？
毕竟两者都是 interface，干嘛不直接 implement Iterator？

事实上并不是多此一举，Iterable 和 Iterator 分离的核心原因是：一个表示“可遍历的数据源”，一个表示“正在遍历的游标”。

Iterator 是有状态的、一次性的：

```kotlin
val it = listOf(1, 2, 3).iterator()

it.next() // 1
it.next() // 2
```

它内部有“当前走到哪了”的状态。走过了就走过了。

而 Iterable 本身不负责保存遍历进度，它只负责提供 Iterator。每次需要遍历时，创建一个新的 Iterator（依赖具体的实现，但绝大多数情况创建新的 Iterator 才合理）。所以：

```kotlin
val list = listOf(1, 2, 3)

for (x in list) {
    println(x)
}

for (x in list) {
    println(x)
}
```

这两次 for 都能从头开始，因为每次 for 都会调用一次 list.iterator() 拿到新的 Iterator。<br>
如果集合本身直接就是 Iterator，问题会很多。比如嵌套遍历：

```kotlin
val list = listOf(1, 2, 3)

for (a in list) {
    for (b in list) {
        println("$a $b")
    }
}
```

这里需要两个互不干扰的遍历状态：外层一个，内层一个。<br>
如果 list 自己就是 Iterator，内层循环会把外层的游标状态也消费掉，逻辑就乱了。

所以设计上通常是：
* Iterable<T>：我这个对象可以被遍历，可以提供 iterator
* Iterator<T>：我正在遍历，知道当前走到哪里了

可以把它理解成：<br>
Iterable = 可重复取票的入口<br>
Iterator = 一张正在使用的票

Kotlin 的 for (x in something) 本质上关心的是 something.iterator()，所以 Iterable 是集合、容器、数据源这类对象的抽象；
Iterator 则是一次具体遍历过程的抽象。

这也是为什么 List、Set 这类集合实现的是 Iterable，而不是直接把自己做成 Iterator。

另外，Iterable 和 Iterator 的几个方法都是 operator fun，对应的是同一个语言约定：for-in 遍历协议。<br>

```kotlin
for (x in iterable) {
    println(x)
}
```

会被编译器大致展开成：

```kotlin
val iterator = iterable.iterator()
while (iterator.hasNext()) {
    val x = iterator.next()
    println(x)
}
```

for-in 里的 in 只是一个语法约定，不同于 contains 重载操作符对应的 in。

普通判断：

```kotlin
if (x in collection) { ... }
```

等价于：

```kotlin
if (collection.contains(x)) { ... }
```
  
这里重载的是：

```kotlin
operator fun contains(x: T): Boolean
```
