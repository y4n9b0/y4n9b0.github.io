---
layout: post
title:  Hello Rust
date:   2019-06-10 21:00:00 +0800
categories: Rust
tags: Rust
published: true
---

* content
{:toc}

## Prologue

Rust 是一门系统编程语言，专注于安全，尤其是并发安全，支持函数式和命令式以及泛型等编程范式的多范式语言。Rust 在语法上和 C++ 类似，但是设计者想要在保证性能的同时提供更好的内存安全。Rust 最初是由 Mozilla 研究院的 Graydon Hoare 设计创造，然后在 Dave Herman, Brendan Eich 以及很多其他人的贡献下逐步完善的。

## WebSite

* [Rust 官网](https://www.rust-lang.org/)
* [Rust 中文](https://rustlang-cn.org/)

## Docs

The Rust Programming Language:

* [英文](https://doc.rust-lang.org/book/)
* [中文](https://rustlang-cn.org/office/rust/book/)
* [trpl-zh-cn](https://kaisery.github.io/trpl-zh-cn/foreword.html)

## Installation

* 官方推荐的 [rustup 安装](https://www.rust-lang.org/tools/install)

    ```bash
    curl https://sh.rustup.rs -sSf | sh
    ```

* [其他安装方式](https://forge.rust-lang.org/other-installation-methods.html)

强烈建议使用官方推荐的 rustup 安装，但是对于墙内用户需要做一点改变，参考中科大 [Rust Toolchain 反向代理使用帮助](https://mirrors.ustc.edu.cn/help/rust-static.html)：

1. 设置环境变量

    ```bash
    # RUSTUP_DIST_SERVER （用于更新 toolchain）
    export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static

    # RUSTUP_UPDATE_ROOT （用于更新 rustup）
    export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
    ```

2. 通过 [jsdelivr](https://cdn.jsdelivr.net/gh/rust-lang-nursery/rustup.rs/rustup-init.sh) 下载 `rustup-init.sh`
3. 替换脚本中的 `RUSTUP_UPDATE_ROOT` 变量值为 `https://mirrors.ustc.edu.cn/rust-static/rustup`
4. 执行该 `rustup-init.sh` 脚本。

Rust 开发环境，所有的工具都安装在 `~/.cargo/bin` 目录，包括 rustc、 cargo 和 rustup。安装完成后需要将该目录加入到环境变量中去：

```bash
export PATH="$HOME/.cargo/bin:$PATH"
```

环境变量生效后就可以通过相应的命令查看版本：

```bash
rustc --version
cargo --version
rustup --version
```

## Hello world

首先创建一个存放 Rust 代码的目录 RustProjects。Rust 并不关心代码的存放位置，但是建议将所有的 Rust 项目都存放在同一个目录下。
然后在 RustProjects 目录下创建一个 hello_world，需要注意的是这里的工程目录名建议使用蛇形命名，否则会被 cargo 告警：

```bash
mkdir ~/RustProjects
cd ~/RustProjects

mkdir hello_world
cd hello_world
```

接下来，新建一个源文件，命名为 main.rs。Rust 源文件总是以 .rs 扩展名结尾。如果文件名包含多个单词，使用下划线分隔它们（snake case name）。

```bash
touch main.rs
```

打开刚创建的 main.rs 文件，输入示例代码。

```rust
fn main() {
    // println! 调用了一个 Rust 宏（macro）。如果是调用函数，则没有!
    println!("Hello, world!");
}
```

保存文件，编译并运行文件：

```bash
$ rustc main.rs
$ ./main
Hello, world!
```

Rust 是一种 **预编译静态类型**（ahead-of-time compiled）语言，将编译和运行分为两个单独的步骤。

## Hello Cargo

仅仅使用 rustc 编译简单程序是没问题的，不过随着项目的增长，就需要用到 Cargo 管理项目的方方面面。

Cargo 是 Rust 的构建系统和包管理器。大多数 Rustacean 们使用 Cargo 来管理他们的 Rust 项目，因为它可以处理很多任务，比如构建代码、下载依赖库并编译这些库。

1. 使用 Cargo 创建项目

    ```bash
    cargo new hello_cargo
    cd hello_cargo
    ```

    进入 hello_cargo 目录并列出文件。将会看到 Cargo 生成了两个文件和一个目录：一个 Cargo.toml 文件，一个 src 目录，以及位于 src 目录中的 main.rs 文件。它也在 hello_cargo 目录初始化了一个 git 仓库，以及一个 .gitignore 文件。
    Cargo.toml 是该项目 [TOML](https://github.com/toml-lang/toml) (Tom's Obvious, Minimal Language) 格式的配置文件，src 目录存放的是源代码文件。

    **Note**: 可以通过 --vcs 参数使 cargo new 切换到其它版本控制系统（VCS），或者不使用 VCS。运行 cargo new --help 参看可用的选项。

2. 构建并运行 Cargo 项目

    ```bash
    $ cargo build
        Compiling hello_cargo v0.1.0 (/Users/bob/RustProjects/hello_cargo)
         Finished dev [unoptimized + debuginfo] target(s) in 0.46s
    ```

    这个命令会创建一个可执行文件 target/debug/hello_cargo，执行该文件

    ```bash
    $ ./target/debug/hello_cargo
    Hello, world!
    ```

    使用 `cargo run` 可以同时编译并运行生成的可执行文件：

    ```bash
    $ cargo run
       Compiling hello_cargo v0.1.0 (/Users/bob/RustProjects/hello_cargo)
        Finished dev [unoptimized + debuginfo] target(s) in 0.46s
         Running `target/debug/HelloCargo`
    ```

    如果项目已经编译过且源文件并没有被改变，Cargo 会直接运行二进制文件，不会重新编译。
    若需要重新编译，可以通过 `cargo clean` 清除相关缓存。

    Cargo 还提供了一个叫 `cargo check` 的命令。该命令快速检查代码确保其可以编译，但并不产生可执行文件：

    ```bash
    $ cargo check
        Checking hello_cargo v0.1.0 (/Users/bob/RustProjects/hello_cargo)
         Finished dev [unoptimized + debuginfo] target(s) in 0.15s
    ```

    为什么会不需要可执行文件呢？通常 cargo check 要比 cargo build 快得多，因为它省略了生成可执行文件的步骤。如果在编写代码时持续的进行检查，cargo check 会加速开发！为此很多 Rustaceans 编写代码时定期运行 cargo check 确保它们可以编译。当准备好使用可执行文件时才运行 cargo build。

3. 发布 release 构建

    当项目最终准备好发布时，可以使用 cargo build --release 来优化编译项目。这会在 target/release 而不是 target/debug 下生成可执行文件。这些优化可以让 Rust 代码运行的更快，不过启用这些优化也需要消耗更长的编译时间。这也就是为什么会有两种不同的配置：一种是为了开发，你需要经常快速重新构建；另一种是为用户构建最终程序，它们不会经常重新构建，并且希望程序运行得越快越好。如果在测试代码的运行时间，请确保运行 cargo build --release 并使用 target/release 下的可执行文件进行测试。
