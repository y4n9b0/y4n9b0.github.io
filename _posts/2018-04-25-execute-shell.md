---
layout: post
title:  excute shell
date:   2018-04-25 15:30:00 +0800
categories: linux
tags: shell
published: true
---

* content
{:toc}


# Prologue
在 linux 里，source、sh、bash、./ 都可以执行 shell script 文件，简单记录下各自的区别。  

# source
```
source a.sh
```
在当前 shell 内去读取、执行 a.sh，而 a.sh 不需要有"执行权限"。  
source命令可以简写为"."

```
. a.sh

```
**Note**：中间是有空格的。

# sh/bash
```
sh a.sh
bash a.sh
```
都是打开一个 subshell 去读取、执行 a.sh，而 a.sh 不需要有"执行权限"。  
通常在 subshell 里运行的脚本里设置变量，不会影响到父 shell 的。

**Note** sh/bash 加上 -x 参数可以调试脚本:
```
sh -x a.sh
bash -x a.sh
```

# ./
```
./a.sh
#bash: ./a.sh: Permission denied
chmod +x a.sh
./a.sh
```
打开一个 subshell 去读取、执行 a.sh，但 a.sh 需要有"执行权限"。  
可以用 chmod +x 添加执行权限

# versus
设置脚本 a.sh 的内容如下：
```
#!/bin/bash
cd /tmp
var="var test"
```
然后分别用以上命令运行 a.sh，再运行 `pwd` 和 `echo $var`，观察对应结果：

* `source a.sh` or `. a.sh`
  ```
  bob@ubuntu:~$ source a.sh
  bob@ubuntu:/tmp$ pwd
  /tmp
  bob@ubuntu:/tmp$ echo $var
  var test
  ```
* `bash a.sh`  
  **Note**: 请新开 terminal 执行，防止前一次试验影响 `$var`
  ```
  bob@ubuntu:~$ bash a.sh
  bob@ubuntu:~$ pwd
  /home/bob
  bob@ubuntu:~$ echo $var

  bob@ubuntu:~$
  ```
* `./a.sh`
  ```
  bob@ubuntu:~$ ./a.sh
  bash: ./a.sh: Permission denied
  bob@ubuntu:~$ chmod +x a.sh
  bob@ubuntu:~$ ./a.sh
  bob@ubuntu:~$ pwd
  /home/bob
  bob@ubuntu:~$ echo $var

  bob@ubuntu:~$

  ```

# fork、source、exec
使用fork方式运行script时， 就是让shell(parent process)产生一个child process去执行该script，当child process结束后，会返回parent process，但parent process的环境是不会因child process的改变而改变的。

使用source方式运行script时， 就是让script在当前process内执行， 而不是产生一个child process来执行。由于所有执行结果均于当前process内完成，若script的环境有所改变， 当然也会改变当前process环境了。

使用exec方式运行script时， 它和source一样，也是让script在当前process内执行，但是process内的原代码剩下部分将被终止。同样，process内的环境随script改变而改变。
通常如果我们执行时，都是默认为fork的。

为了实践下，我们可以先建立2个sh文件：

1.sh
```
#!/bin/bash
A=B
echo "PID for 1.sh before exec/source/fork:$$"
export A
echo "1.sh: \$A is $A"
case $1 in
    exec)
        echo "using exec..."
        exec ./2.sh ;;
    source)
        echo "using source..."
        . ./2.sh ;;
    *)
        echo "using fork by default..."
        ./2.sh ;;
esac
echo "PID for 1.sh after exec/source/fork:$$"
echo "1.sh: \$A is $A"
```

2.sh
```
#!/bin/bash
echo "PID for 2.sh: $$"
echo "2.sh get \$A=$A from 1.sh"
A=C
export A
echo "2.sh: \$A is $A"
```

运行观看结果：
```
chmod +x 1.sh
chmod +x 2.sh
./1.sh fork
./1.sh source
./1.sh exec
```
