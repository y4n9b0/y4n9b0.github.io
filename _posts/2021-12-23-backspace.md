---
layout: post
title:  backspace
date:   2021-12-23 16:00:00 +0800
categories: backspace
tags: terminal
published: true
---

* content
{:toc}

## 一、个困扰很久的现象

大约半年前，在 terminal 里查阅某个 git 指令的时候遇到了一个奇怪现象(年纪大了 记不太清具体啥指令，这里就以 git push 为例吧)。

在 terminal 里 `git push --help` 或者 `man git-push` 看起来一切都是正常的：

![/styles/images/backspace/terminal.png]({{ '/styles/images/backspace/terminal.png' | prepend: site.baseurl }}){:width="570" height="371"} 

<br>
但是当我 `git push --help > ~/push.txt` 把 manual 文档输出重定向到本地文件，然后再用 macOS 的文本编辑打开 push.txt 文件时，所有 terminal 里加黑加粗的字体在 txt 文本里都是重复且未加粗的：

![/styles/images/backspace/txt.png]({{ '/styles/images/backspace/txt.png' | prepend: site.baseurl }}){:width="571" height="373"} 

## 二、拨开云雾见天日

通过比对 terminal 和 txt 文本编辑，不难猜测应该是 manual 手册在终端输出的加粗格式无法被 macOS 的文本编辑识别，导致错误显示为重复的字体。接下来我们分别用 vim 和 VScode 试试。

vim 打开 push.txt：<br>
![/styles/images/backspace/vim.png]({{ '/styles/images/backspace/vim.png' | prepend: site.baseurl }}){:width="570" height="371"}  

<br>
VScode 打开 push.txt：<br>
![/styles/images/backspace/vscode.png]({{ '/styles/images/backspace/vscode.png' | prepend: site.baseurl }}){:width="570" height="395"}  

<br>
可以看到，在vim 里是 `^H`，在 vscode 里是很小且背景标红的 `BS`，这两都不是普通的字体内容（hard-to-see/type），但都代表了同一个意思 backspace，也就是转义字符 `\b`。

backspace 即退格键，manual 里使用 `<character>\b<same-character>` 来显示粗黑的字体。仔细想想，其实挺有意思，相当于我先在屏幕上显示了一个字母，然后把光标后退一格，再在同样的位置又显示一遍该字母，从而达到加黑加粗的效果。双胞胎 爱了爱了，借用王自如的话来说 wow～amazing awesome！

macOS 的文本编辑无法识别 backspace，于是就把 `<character>\b<same-character>` 显示成了 `<character><same-character>`，也就是说重定向 manual 手册保存的文件没有问题，关键是看解析是否支持。
然而，很不幸，backspace 加粗显示的这种技术过于古老，已经不怎么使用了，现在比较流行的是 tput。

```bash
# backspace
echo "\n u\b_n\b_d\b_e\b_r\b_l\b_i\b_n\b_e\b_ B\bBO\bOL\bLD\bD \n" | ul
# tput
echo "\n $(tput smul)underline$(tput rmul) $(tput bold)BOLD$(tput sgr0) \n"
```

如上两种方式显示同样的下划线以及加粗效果(echo 也可以换成 print)，tput 的写法明显比 backspace 优雅很多，可读性也更强。<br>
![/styles/images/backspace/tput-vs-backspace.png]({{ '/styles/images/backspace/tput-vs-backspace.png' | prepend: site.baseurl }}){:width="570" height="371"}  

## 三、tput

// todo

<!-- https://stackoverflow.com/a/8657733/7368406 -->
<!-- https://stackoverflow.com/questions/18041720/h-in-bash-variable -->
<!-- https://blog.csdn.net/weixin_33464488/article/details/116625501 -->
