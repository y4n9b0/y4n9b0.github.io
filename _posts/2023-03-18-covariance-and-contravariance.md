---
layout: post
title: 协变与逆变
date: 2023-03-18 23:30:00 +0800
categories: Java
tags: Java
published: true
---

* content
{:toc}

## 定义

* 当 A ≦ B 时,如果有 f(A) ≦ f(B)，那么 f 叫做协变(covariance)；
* 当 A ≦ B 时,如果有 f(B) ≦ f(A)，那么 f 叫做逆变(contravariance)；
* 如果上面两种关系都不成立则叫做不变(invariance)

## 栗子

定义一个父类 Fruit 和两个子类 Apple、Banana，而 Fruit 又继承自 Food

```java
public class Food {}

public class Fruit extends Food {}

public class Apple extends Fruit {
    public void killPersonWithToxicSeed() {}
}

public class Banana extends Fruit {}
```

### 对象

```java
    public void foo() {
        Fruit fruit = new Fruit();
        Apple apple = new Apple();
        fruit = apple;
        apple = fruit; // 编译错误: 不兼容的类型: Fruit无法转换为Apple
    }
```

Java 中对象是默认支持协变、不支持逆变的。
这很好理解，子类除了继承父类所有公开的属性和方法外，还可以定义父类没有的属性和方法，也就是说子类的功能 ≥ 父类的功能，
那么我们把一个子类实例对象赋值给一个父类变量是完全合理的，因为子类能实现所有父类的调用，这就是协变。
就像公司招人一样，我们都倾向于招一个 overqualified 而不是能力不足的。

反之，如果把一个父类实例对象赋值给一个子类变量则称之为逆变，对象是不支持逆变的。
就拿示例中的父类 Fruit 和子类 Apple 来说，我们都知道苹果籽有毒，吃多了会致人死亡，但不是所有的水果籽都有毒，
把一个 Fruit 实例赋值给一个 Apple 变量，无法保证该实例能够调用方法 `killPersonWithToxicSeed`。
更有甚者，我们结合上面的协变，先把子类 Banana 的一个实例对象赋值给 Fruit 变量，然后再把 Fruit 变量赋值给 Apple 变量，
如果对象支持逆变的话，我们会发现香蕉等于苹果了，这显然是不正确的。

### 数组

```java
    public void foo() {
        Fruit[] fruits = new Fruit[9527];
        Apple[] apples = new Apple[9527];
        fruits = apples;
        apples = fruits; // 编译错误: 不兼容的类型: Fruit[]无法转换为Apple[]
    }
```

与对象一样，Java 中数组也是默认支持协变而不支持逆变的。
但数组协变其实是有些问题的：

```java
    public void foo() {
        Fruit[] fruits = new Apple[9527];
        fruits[0] = new Banana();
    }
```

如上所示，我们分配了一个 Apple 数组，并通过数组协变将其赋值给一个 Fruit 数组，再通过对象协变将一个 Banana 实例对象赋值给 Fruit 数组的其中一个元素。
这段代码是可以成功编译的，但会运行出错 ArrayStoreException。
事实上足够智能的编译器都会提前警示：
Storing element of type 'com.y4n9b0.playground.Banana' to array of 'com.y4n9b0.playground.Apple' elements will produce 'ArrayStoreException'

出错的原因也不难理解，分配的数组实例对象是固定为 Apple 的，插入一个非 Apple 及其子类的对象肯定会类型不匹配。
其实取消数组协变可以在编译期就避免该问题，至于为什么要将数组设计为支持协变，应该是 Java 早期没有泛型(since Java 1.5)的妥协，可以参考知乎这个问答：
[java中，数组为什么要设计为协变？](https://www.zhihu.com/question/21394322){:target="_blank"}

很多设计缺陷，都是对历史的妥协，大家在工作中应该都有不少的体会。有时候不是我们不愿意做正确的事儿，实不能也:)

### 集合

```java
    public void foo() {
        List<Fruit> fruits = new ArrayList<>();
        List<Apple> apples = new ArrayList<>();
        fruits = apples;        // 编译错误: 不兼容的类型: List<Apple>无法转换为List<Fruit>
        apples = fruits;        // 编译错误: 不兼容的类型: List<Fruit>无法转换为List<Apple>
        fruits.addAll(apples);  // 可以添加，这里实际上使用了对象的协变，与集合协变/逆变无关
    }
```

Java 中集合是默认不支持协变和逆变的。
但是我们可以通过通配符(Wildcards)手动支持协变和逆变：

```java
    public void foo() {
        List<? extends Fruit> fruits = new ArrayList<>(); // 上界通配符，支持协变
        List<Apple> apples = new ArrayList<>();
        fruits = apples; // 协变成功
        apples = fruits; // 编译错误: 不兼容的类型: List<Fruit>无法转换为List<Apple>

        Fruit fruit = fruits.get(0);    // 可以读取
        fruits.add(new Apple());        // 编译错误: fruits 只能读，不能修改
        fruits.add(new Food());         // 编译错误: fruits 只能读，不能修改
    }

    public void bar() {
        List<Fruit> fruits = new ArrayList<>();
        List<? super Apple> apples = new ArrayList<>(); // 下界通配符，支持逆变
        fruits = apples; // 编译错误: 不兼容的类型: List<Apple>无法转换为List<Fruit>
        apples = fruits; // 逆变成功

        apples.add(new Apple());        // 可以添加，add/set Apple 及其子类 
        apples.add(new Food());         // 编译错误： apples 只能 add/set Apple 及其子类
        apples.add(new Banana());       // 编译错误： apples 只能 add/set Apple 及其子类
        fruits.add(new Banana());       // 可以添加。这里挺有意思，因为 Banana 不是 Apple 的子类，apples 不能直接添加 Banana；但是 fruits 可以添加 Banana，又因为 fruits 赋值给了 apples，所以通过 fruits 添加 Banana 可以达到 apples 添加 Banana 的目的。Surprise mother fxxker!
        Apple apple = apples.get(0);    // 编译错误： apples 只能修改，不能读取赋值给 Apple 变量
        Object object = apples.get(0);  // 可以读取赋值给 Object 变量
    }
```

* <? extends T> 上界通配符，不能修改，只能读取。集合元素类型是 T 及其任意子类(派生类)，所以读取出来的元素可以赋值给 T 及其父类。若想成功添加元素，必须保证被添加元素是当前集合元素类型或其子类，才能协变成功；上界通配符这里的元素类型是指 T 及其任意子类，如果我们把 T 及其任意子类看成一个类型的集合，由于子类可以无限派生，这个类型集合理论上是无限大，没有办法保证被添加的元素能协变成这个类型集合里的任意元素(类型)，所以不能修改。不能修改也有很多好处，诸如避免多线程加锁。
* <? super T> 下界通配符，可以修改，仅能读取赋值给 Object。集合元素类型是 T 及其任意父类，所以可以写入 T 及其子类(对象协变)；而读取出来的元素仅可以赋值给 Object 是因为 Object 是所有类的基类，把任何东西赋值给 Object 都是 ok 的，apples.get(0) 一定是某个东西对吧？

## PECS 原则

PECS（Producer Extends Consumer Super）原则：

* 生产者，频繁往外读取内容的，适合用上界 extends。
* 消费者，经常往里插入的，适合用下界 super。

## kotlin

kotlin 基于 Java，所以协变/逆变是一样的，只不过语法、关键字不一样，记住 out(协变)、in(逆变)就行了。

<!-- https://www.zhihu.com/question/21394322 -->
<!-- https://www.zhihu.com/question/20400700/answer/117464182 -->