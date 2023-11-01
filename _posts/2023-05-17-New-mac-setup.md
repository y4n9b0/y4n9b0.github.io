---
layout: post
title: Mac 新电脑 Android 开发环境配置
date: 2023-05-17 17:30:00 +0800
categories: setup
tags: setup
published: true
---

* content
{:toc}

## Homebrew

Homebrew 官网 https://brew.sh/<br>
臭名昭著 GFW 的原因，国内不建议直接使用官网推荐的方式安装，可参考如下换源安装

1. 下载 install shell 脚本
    ```bash
    curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh > install.sh
    ```
    如果下载不成功，可能是 GitHub 的 raw.githubusercontent.com 域名解析被污染了，在 /etc/hosts 中配置 199.232.28.133 raw.githubusercontent.com 试试。
2. 修改 install.sh 脚本，替换安装源
    ```bash
    HOMEBREW_BREW_DEFAULT_GIT_REMOTE="https://github.com/Homebrew/brew"
    HOMEBREW_CORE_DEFAULT_GIT_REMOTE="https://github.com/Homebrew/homebrew-core"
    ```
    替换为中科大源（当然也可以使用清华的）：
    ```bash
    HOMEBREW_BREW_DEFAULT_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
    HOMEBREW_CORE_DEFAULT_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
    ```
3. 添加 install.sh 脚本执行权限
    ```bash
    sudo chmod +x install.sh
    ```
4. 执行 install.sh 脚本安装
    ```bash
    /bin/bash -c ./install.sh
    ```
    安装过程可能会静默升级，持续时间较长，只要没出错耐心等待即可。安装成功后按 next step 提示添加环境变量。

5. 替换 api 源
    ```bash
    echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
    echo 'export HOMEBREW_API_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/api' >> ~/.zprofile
    ```
之所以把安装 Homebrew 放在第一位，是因为 Homebrew 可以简便地安装很多其他软件，大家都懂的。当然也不排除有人跟我一样不喜欢用 Homebrew 安装的，Anyway 排第一总是好的。

## pkg-config

```bash
brew install pkg-config
```

pkg-config 是后续很多软件源码安装 configure 阶段需要用到的重要工具，对于喜欢源码安装的人来说，需要提前安装。

## Git

Mac 下载 Git 官网链接 https://git-scm.com/download/mac，Git 安装方式有三种（直接无视官网提到的 MacPorts 和二进制文件安装）：

1. 命令行开发者工具 Command line tools
```bash
xcode-select --install
```

2. Homebrew
```bash
brew install git
```

3. 源码安装
https://mirrors.edge.kernel.org/pub/software/scm/git/ 找到自己想要安装的版本点击下载，下载解压后 terminal 进入该文件夹
```bash
sudo ./configure
sudo make
sudo make install
```

第一种命令行开发者工具可能受网络限制会下载很久，不太推荐（网络没问题的话当我没说）。个人比较推荐第三种源码安装，可以装任意版本，网上很多人跟风说源码安装麻烦，对于懒惰的人来说那确实麻烦～

## gitconfig

Home 目录下新建 .gitconfig 文件：

```
 [user]
     name = ᵇᵒ
     email = 40080007@qq.com
 [alias]
     co = checkout
     br = branch
     ci = commit
     st = status
     rb = rebase
     af = !git add -A && git commit --amend --no-edit && git push --force && :
     lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
 [push]
     default = simple
 [gui]
     encoding = utf-8
 [core]
     quotepath = false
 [color]
     ui = auto
 [pull]
     rebase = true
 [rebase]
     autoStash = true
```

## Chrome

两种方式：
1. 官网链接 https://www.google.cn/intl/zh-CN/chrome/，点击下载即可，下载成功后安装
2. wget 下载 chrome dmg 文件
    ```bash
    wget https://dl.google.com/chrome/mac/universal/stable/GGRO/googlechrome.dmg
    ```
两种方式理论上是一回事，都是同一个下载链接，方法 2 还需要安装 wget，所以用方法 1 就好。但方法 2 的好处是，命令行更加方便于脚本自动化安装。

### 插件：[OneTab](https://chrome.google.com/webstore/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall){:target="_blank"}<br>
如果打不开 google 商店，也可以百度搜索相关插件文件 直接下载本地安装，比如 [极简插件 OneTab](https://chrome.zzzmh.cn/info/chphlpgkkbolifaimnlloiipkdnihall){:target="_blank"}

## Android Studio

官网 https://developer.android.google.cn/studio 点击下载安装即可。<br>
需要注意的是经常最新版的 Android Studio 都是深坑，如果需要下载历史版本去归档链接：<br>
https://developer.android.google.cn/studio/archive?hl=zh-cn<br>
语言设置为中文的话可能无法看到近期一些归档版本，把语言从中文改成英文即可看到：<br>
[https://developer.android.google.cn/studio/archive?hl=en](https://developer.android.google.cn/studio/archive?hl=en){:target="_blank"}

* 安装 SDK 和 sdk tools（settings -> Languages & Frameworks -> SDK Tools -> 取消勾选 Hide Obsolete Packages）
* 安装插件 dart、flutter

## JDK

下载 JDK 还需要注册登陆的专利流氓 Oracle 首先排除，考虑直接使用 openJDk<br>
openJDK 官网安装链接 https://openjdk.org/install/<br>
openJDK 官网安装归档文件链接 https://jdk.java.net/archive/

### JDK 11

在归档文件里找到 JDK 11 对应的操作系统版本点击下载即可。 [JDK 11.0.2 Mac 64-bit](https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_osx-x64_bin.tar.gz){:target="_blank"}

### JDK 8

* azul<br>
https://www.azul.com/downloads/?version=java-8-lts&os=macos&architecture=arm-64-bit&package=jdk#zulu
筛选出自己需要的版本（上面的链接已经筛选出 Mac arm-64 jdk 8），点击底部下载 tar.gz 版本解压，取其中 zulu-8.jdk 文件夹即可

* Amazon corretto
暂不介绍

另外，JDK 还可以通过 Android Studio -> Gradle Setting -> Gradle JDK -> Download JDK 来安装想要的版本。

## Gradle

官方下载链接 https://services.gradle.org/distributions/

## Rosetta

```bash
sudo softwareupdate --install-rosetta --agree-to-license
```

## VS Code

官网下载链接 https://code.visualstudio.com/Download<br>
获取下载链接后，替换host haz764295.vo.msecnd.net 为国内镜像 vscode.cdn.azure.cn，可解决国内下载速度慢的问题

## wget

两种安装方式：

1. Homebrew
```bash
brew install wget
```

2. 手动安装
https://www.gnu.org/software/wget/ 下载需要安装版本的 tar.gz 文件，然后解压：
```bash
tar zxvf xxx.tar.gz
cd xxx # 对应的解压目录
sudo ./configure
sudo make
sudo make install
```

建议使用 Homebrew 安装，源码安装 configure 阶段需要相当多的依赖，否则会 configure 失败。

## 解压软件

推荐 The Unarchiver 官网链接 https://theunarchiver.com/，直接点击下载 dmg 文件安装即可

## 环境变量

高版本的 macOS 已经默认使用 zsh，所以只需要在 Home 目录下新建 .zshrc 配置文件即可，
Homebrew、jdk、gradle、Android Studio 等配置必须，其他随意。

## ssh key

生成 ssh key
```bash
ssh-keygen -t ed25519 -C "comment" -P "password"
```
* -t 指定加密方式，除 rsa 外，还有 dsa、ecdsa、rsa、rsa1 等方式，最好使用 ed25519 (Github 有要求)。
* -C 指定注释，说明这个 KEY 的用途等，大多数使用邮箱主要是方便管理，说明这个 KEY 是谁的。
* -P 指定密码，可以设置为空密码""，在这里设置密码就可以省略掉后续输密码按 Enter 键的次数。
* 还有一些其他参数，可以通过man ssh-keygen 查看使用。

查看 ssh publish key
```bash
cat ~/.ssh/id_ed25519.pub 
```

将显示的 ssh publish key 字符串整个复制，添加到代码托管平台比如 github。

## “无法打开应用，因为Apple无法检查其是否包含恶意软件“解决方法

* 方法一，在系统偏好设置-> 安全性与隐私-> 通用中启用该应用
* 方法二，执行下面命令，禁用新安全检查：
    ```bash
    sudo spctl --master-disable
    ```
* 方法三，xattr 移除 com.apple.quarantinevia 属性：
    ```bash
    sudo xattr -r -d com.apple.quarantine ${file}
    ```
    <!-- https://www.cnblogs.com/Flat-White/p/17153264.html -->
    <!-- https://blog.csdn.net/qq_35624642/article/details/125014122 -->

## Flutter

1. 下载
    ```bash
    git clone https://github.com/flutter/flutter.git -b stable
    ```
2. 添加环境变量
    ```bash
    echo 'export PATH="$PATH:'$PWD'/flutter/bin"' >> ~/.zshrc
    ```
3. 检查
    ```bash
    flutter doctor
    ```

    可能出现的缺失：
    * Android toolchain - cmdline-tools component is missing，直接在 Android Studio SDKmanager 页面 SDK Tools 标签下面点击安装即可，也可以终端运行命令（需要切到java8）：
    ```bash
    sdkmanager --install "cmdline-tools;latest"
    ```
    * Android toolchain - Android license status unknown，终端运行命令（需要切到java11）：
    ```bash
    flutter doctor --android-licenses
    ```

## smartctl

查看 mac 硬盘读写工具

```bash
brew install smartmontools
```

```bash
smartctl -a disk0
```

## 修改电脑名字

```bash
# terminal 显示名字
sudo scutil --set HostName xxx
# 共享显示名字，比如隔空传送
sudo scutil --set LocalHostName xxx
```

## cloc 统计代码行数

```bash
brew install cloc
```

## motd

锦上添花的小玩意儿，不懂的可以无视。

<!-- https://zhuanlan.zhihu.com/p/610820273 -->
<!-- https://blog.csdn.net/weixin_41984936/article/details/130809661 -->