---
layout: post
title:  file descriptor and IO redirections
date:   2019-04-09 16:00:00 +0800
categories: Linux
tags: shell fd redirections
published: true
---

* content
{:toc}

# Prologue

UNIX 的设计哲学是 Everything is a file，不仅将数据抽象成了文件，也将一切操作和资源抽象成了文件，比如说硬件设备，socket，磁盘，进程，线程等。  
当然，Linus 认为更贴切地说法是 [Everything is a stream of bytes](https://yarchive.net/comp/linux/everything_is_file.html)。  

|   文件类型                   |   标记符  |
|   :-                        |   :-:    |
|   普通文件                   |    -     |
|   目录文件（directory）       |    d     |
|   字符设备文件（character)    |    c     |
|   块设备文件（block）         |    b     |
|   套接字文件（socket）        |    s     |
|   管道文件（pipe）            |    p     |
|   链接文件（link）            |    l     |

# File Descriptor

Linux 操作文件时，会将其分配一个对应的索引，该索引就是文件描述符（简称 fd）。fd 在系统里面是一个非负的整数，每打开或创建一个文件，内核就会向进程返回一个当前进程最小可用的文件描述符号码 fd。

Linux 启动后，会默认打开 3 个文件描述符:

|   FileDescriptor  |   名称       |   POSIX常量标识(unistd.h)  |   标准IO常量标识(stdio.h) |
|   :-:             |   :-:       |   :-:                     |   :-:                   |
|   0               |   标准输入流  |   STDIN_FILENO            |   stdin                 |
|   1               |   标准输出流  |   STDOUT_FILENO           |   stdout                |
|   2               |   标准错误流  |   STDERR_FILENO           |   stderr                |

一条 shell 指令执行，都会继承父进程的文件描述符。因此， 所有运行的 shell 指令，都会有 Linux 默认启动的 3 个文件描述符。使用 `ls -la /dev | grep std` 可以看到 /dev/stdin、/dev/stdout、/dev/stderr 分别链接到 fd 对应的 0、1、2。

用户可以自定义文件描述符范围是：3 ～ num，这个最大数字 num 是定义的单一进程允许打开的最大文件数，可使用指令 `ulimit -n` 查看。

[文件描述符更多的内部细节](https://www.jianshu.com/p/504a53c30c17)

# IO redirections

IO redirections 指的是重定向标准 IO 流（上面提及的 3 个默认文件描述符对应的标准 IO 流）。输入流向程序提供输入，这些输入通常来自键盘。输出流打印文本字符，通常打印到终端 tty。

|   命令                |    说明                                          |
|   :-                 |    :-                                           |
|   command < file     |    将输入重定向到 file。                           |
|   command > file     |    将输出重定向到 file。                           |
|   command >> file    |    将输出以追加的方式重定向到 file。                 |
|   n > file           |    将文件描述符为 n 的文件重定向到 file。            |
|   n >> file          |    将文件描述符为 n 的文件以追加的方式重定向到 file。  |
|   n >& m             |    将输出文件 m 和 n 合并。                        |
|   n <& m             |    将输入文件 m 和 n 合并。                        |
|   <<tag              |    将开始标记 tag 和结束标记 tag 之间的内容作为输入。  |

* `>` 和 `>>` 的区别在于前者是覆盖，后者是追加；符号前面不带文件描述符则默认指的是标准输出流。
* 将标准输出和标准错误都重定向到一个文件中：

    ```bash
    command &>output.txt
    ```

    或者

    ```bash
    command >output.txt 2>&1
    ```

    注意重定向输出的顺序很重要，不能使用`command 2>&1 >output.txt`，这种方式只重定向了标准输出，标准错误依然显示在终端，最终效果等同于`command >output.txt`。
* 使用 `/dev/null` 忽略输出

    ```bash
    command >output.txt 2>/dev/null
    ```

    /dev/null 是空设备文件，它就像一个无底洞，会丢弃一切写入其中的数据，而且没有任何可以读取的内容。  
    示例中的 command 使用 `ls` 可以很方便的进行实践，当列举的路径没有对应文件时就会输出标准错误 `No such file or directory`。

# tee

输出到终端的同时重定向到文件

```bash
command | tee output.txt
```