---
layout: post
title: Rust 学习笔记
date: 2023-06-26 10:30:00 +0800
categories: rust
tags: rust
published: false
---

* content
{:toc}

## 为什么学习 Rust

内存管理三种类型

## Rust playground

[Rust playground](https://play.rust-lang.org/){:target="_blank"}

## Rust 基础类型

[Rust 基础类型](https://blog.fudenglong.site/2022/04/10/%E3%80%90Rust%E3%80%91%E5%9F%BA%E7%A1%80%E7%B1%BB%E5%9E%8B/){:target="_blank"}

## Rust string

```rust
fn main() {
    let mut s = String::from("hey");
    s += " girl";
    println!("{}", s) // 编译成功
}
```

```rust
fn main() {
    let mut s = "hey";
    s += " girl";
    println!("{}", s) // 编译错误
}
```

```rust
fn main() {
    let s1 = String::from("hey");
    let s2 = "hey";
    print_type_of(&s1); // alloc::string::String
    print_type_of(&s2); // &str
}

fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>())
}
```

## Rust 变量

## Rust 引用

```rust
fn main() {
    let s1 = "hey";
    let s2 = &s1;
    print_type_of(&s1); // &str
    print_type_of(&s2); // &&str
}

fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>())
}
```

```rust
fn main() {
    let s1 = String::from("hey");
    let s2 = &s1;
    print_type_of(&s1); // alloc::string::String
    print_type_of(&s2); // &alloc::string::String
}

fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>())
}
```

```rust
fn main() {
    let s1 = 1;
    let s2 = &s1;
    print_type_of(&s1); // i32
    print_type_of(&s2); // &i32
}

fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>())
}
```

```rust
// failure
fn main() {
    assert!(is_five(&5));
    assert!(!is_five(&6));
    println!("Success!");
}

fn is_five(x: &i32) -> bool {
    let five = 5;
    x == five
}
```

```rust
// success
fn main() {
    assert!(is_five(&5));
    assert!(!is_five(&6));
    println!("Success!");
}

fn is_five(x: &i32) -> bool {
    let five = 5;
    x == &five
}
```

```rust
fn main() {
    let s1 = 1;
    let s2 = &s1;
    let s3 = *s2;
    print_type_of(&s1); // i32
    print_type_of(&s2); // &i32
    print_type_of(&s3); // i32
}

fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>())
}
```

```rust
// success
fn main() {
    assert!(is_five(&5));
    assert!(!is_five(&6));
    println!("Success!");
}

fn is_five(x: &i32) -> bool {
    let five = 5;
    *x == five
}
```

## Rust 所有权(ownership)和借用(borrow)

所有权有以下三条规则：

* Rust 中的每个值都有一个变量，称为其所有者。
* 一次只能有一个所有者。
* 当所有者不在程序运行范围时，该值将被删除。

每个女人都有一个老公，同一时刻只能有一个老公，如果现任老公挂了 老婆也会跟着一起陪葬

## copy clone move


borrow move
```rust
fn main() {
    let s1 = String::from("hey");
    let s2 = s1;
    println!("{}, {}", s1, s2); // 编译错误
}
```

copy 只是存储在栈中的数据，比如整数类型，浮点类型，布尔类型，字符类型。这些是rust直接进行隐式拷贝，不需要移动操作。
```rust
fn main() {
    let s1 = "hey";
    let s2 = s1;
    println!("{}, {}", s1, s2); // 编译成功
}
```

```rust
fn main() {
    let s1 = "hey";
    let s2 = s1;
    println!("{}, {}", s1, s2); // 编译成功
    print_type_of(&s1); // &str
    print_type_of(&s2); // &str
}

fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>())
}
```


### 插件：[OneTab](https://chrome.google.com/webstore/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall){:target="_blank"}<br>