---
layout: post
title:  dev config
date:   2019-06-18 11:30:00 +0800
categories: tool
tags: config
published: true
---

* content
{:toc}

## chrome

1. 技巧

    * 把 chrome 当作临时记事本，在标签输入 `data:text/html,<html contenteditable>`

2. 插件

    * [OneTab](https://chrome.google.com/webstore/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall){:target="_blank"}
    * [Octotree](https://chrome.google.com/webstore/detail/octotree/bkhaagjahfmjljalopjnoealnfndnagc/related){:target="_blank"}
    * [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop){:target="_blank"}
    * [Tampermonkey](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo?utm_campaign=en){:target="_blank"}

    如果打不开 google 商店，也可以百度搜索相关插件文件 直接下载本地安装，比如 [极简插件 OneTab](https://chrome.zzzmh.cn/info?token=chphlpgkkbolifaimnlloiipkdnihall){:target="_blank"}

## vim

1. 基础

    * 学习 vim（终端里输入 vimtutor）
    * 学习 vim script 基本语法（vim 里输入 :help usr_41.txt）

2. .vimrc

    ```vim
    set nocompatible                        " 关闭vi兼容模式
    set shiftwidth=4                        " 设置缩进为4个空格
    set tabstop=4                           " 设置tab键4个空格长度
    set expandtab                           " 转换tab为空格
    set smartindent                         " 开启smart缩进
    set number                              " 显示行号
    syntax on                               " 开启颜色高亮
    colorscheme desert                      " 设置配色方案
    set cursorline                          " 突出显示当前行
    highlight CursorLine cterm=NONE ctermbg=black ctermfg=green guibg=black guifg=green
    set ruler                               " 打开状态栏标尺
    set incsearch                           " 输入搜索内容时就显示搜索结果
    set hlsearch                            " 搜索时高亮显示被找到的文本
    au FileType gitcommit setlocal cc=72    " 设置git commit 标尺，ColorColumn
    au FileType gitcommit setlocal tw=72    " 设置git commit 行宽，TextWidth
    ```

3. 插件

    * [vim-plug](https://github.com/junegunn/vim-plug) 插件管理器
    * [nerdtree](https://github.com/scrooloose/nerdtree) 树状文件列表
    * [defx](https://github.com/Shougo/defx.nvim) 异步树状文件列表
    * [TagList](https://www.vim.org/scripts/script.php?script_id=273) 文件函数列表

## tmux

1. .tmux.conf

    ```tmux
    # 设置 r 键重新加载配置文件
    bind r source-file ~/.tmux.conf \; display "Config reloaded.."

    # 开启鼠标支持，-g 表示 global
    set-option -g mouse on

    # 绑定 hjkl 键（vim 风格）为面板切换的方向键，-r 表示可重复案件，大约 500ms 之内，重复的 hjkl 按键都将有效
    bind -r h select-pane -L        # 绑定 h 为 ←
    bind -r j select-pane -D        # 绑定 j 为 ↓
    bind -r k select-pane -U        # 绑定 k 为 ↑
    bind -r l select-pane -R        # 绑定 l 为 →
    ```

## git

1. .gitconfig

    ```git
    [user]
        name = ᴮᵒᵇ
        email = step2hell@qq.com
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

## bash

1. ~~.bash_profile~~

    ```bash
    # tree
    alias tree="find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'"

    # Android Studio
    ANDROID_STUDIO=/Applications/Android\ Studio.app/Contents
    export PATH=$PATH:$ANDROID_STUDIO/MacOS:$ANDROID_STUDIO/bin

    # Gradle
    GRADLE_6=${HOME}/.gradle/wrapper/dists/gradle-6.5-all/2oz4ud9k3tuxjg84bbf55q0tn/gradle-6.5
    GRADLE_7=${HOME}/.gradle/wrapper/dists/gradle-7.0.2-all/7era6s5ay7zsbhuvl0oc9g94s/gradle-7.0.2
    export PATH=$PATH:${GRADLE_7}/bin

    # openJDK
    JDK8=/Users/bob/Library/Java/JavaVirtualMachines/corretto-1.8.0_312/Contents/Home
    JDK11=$ANDROID_STUDIO/jre/Contents/Home
    export JAVA_HOME=$JDK11
    export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$JAVA_HOME/jre/lib
    export CLASSPATH=$JAVA_HOME/lib:$JAVA_HOME/lib
    alias java8="export JAVA_HOME=$JDK8"
    alias java11="export JAVA_HOME='$JDK11'"

    # Android SDK
    export ANDROID_HOME=/Users/bob/Library/Android/sdk
    export PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin

    # java keystore
    export ANDROID_STORE_PWD=xxx
    export ANDROID_KEY_PWD=xxx

    # rust
    export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
    export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
    # cargo
    export PATH="$HOME/.cargo/bin:$PATH"

    # MySQL
    MySQL_HOME=/usr/local/mysql
    export PATH=$PATH:$MySQL_HOME/bin:$MySQL_HOME/support-files

    # gerrit
    alias gerritpush='function _(){ git push origin HEAD:refs/for/$1; };_'

    # bash http://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html
    # PS1="\[\e[1;32m\]\u\[\e[1;33m\]@\[\e[1;34m\]\h\[\e[1;36m\] \W\[\e[0m\]\$ "
    ```

2. .zshrc
    
    macOS 替换默认的 bash 为 zsh 后，原有的 ~/.bash_profile 设置不再生效。<br>
    可以在 ~/.zshrc 文件开始添加如下设置：

    ```zsh
    if [ -f ~/.bash_profile ]; then
        . ~/.bash_profile;
    fi

    # Android Studio
    ANDROID_STUDIO=/Applications/Android\ Studio.app/Contents
    export PATH=$PATH:$ANDROID_STUDIO/MacOS:$ANDROID_STUDIO/bin

    # Gradle
    GRADLE_7=${HOME}/.gradle/wrapper/dists/gradle-7.3.3-bin/6a41zxkdtcxs8rphpq6y0069z/gradle-7.3.3
    export PATH=$PATH:${GRADLE_7}/bin

    # openJDK
    JDK8=/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
    JDK11=/Library/Java/JavaVirtualMachines/openjdk-11.0.2.jdk/Contents/Home
    export JAVA_HOME=$JDK11
    export PATH=$PATH:$JAVA_HOME/bin
    export CLASSPATH=$JAVA_HOME/lib:$JAVA_HOME/lib
    alias java8="export JAVA_HOME=$JDK8"
    alias java11="export JAVA_HOME='$JDK11'"

    # Android SDK
    export ANDROID_HOME=/Users/bob/Library/Android/sdk
    export PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin

    # java keystore
    export ANDROID_STORE_PWD=xxx
    export ANDROID_KEY_PWD=yyy

    # rust
    export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
    export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
    # cargo
    export PATH="$HOME/.cargo/bin:$PATH"

    # MySQL
    MySQL_HOME=/usr/local/mysql
    export PATH=$PATH:$MySQL_HOME/bin:$MySQL_HOME/support-files

    # http://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html
    # PS1="\[\e[1;32m\]\u\[\e[1;33m\]@\[\e[1;34m\]\h\[\e[1;36m\] \W\[\e[0m\]\$ "
    # zsh
    autoload -U colors && colors
    PS1="%{$fg[green]%}%n%{$reset_color%}@%{$fg[blue]%}%m %{$fg[cyan]%}%~ %{$reset_color%}%% "

    alias tree="find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'"
    export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles

    alias screencap="adb exec-out screencap -p > ~/screencap.png"

    alias gerritpush='function _(){ git push origin HEAD:refs/for/$1; };_'

    alias uploadBuglyMapping="${JDK8}/bin/java -jar ~/Downloads/buglyqq-upload-symbol/buglyqq-upload-symbol.jar -appid bec105b4fe -appkey 8dfd6c4c-bf37-40c9-9192-794caff65884 -bundleid com.step2hell.newsmth -version 1.0.0 -platform Android -inputSymbol ~/AndroidProjects/sm-kotlin/app/build/outputs/mapping/release -inputMapping ~/AndroidProjects/sm-kotlin/app/build/outputs/mapping/release"

    # Jekyll ruby
    export SDKROOT=$(xcrun --show-sdk-path)
    # ruby
    export PATH="/usr/local/lib/ruby/gems/3.0.0/bin:$PATH"
    export PATH="/usr/local/opt/ruby/bin:$PATH"

    # python 2.7.18
    export PATH=$(pyenv root)/shims:$PATH

    # Flutter
    export PATH="$PATH:${HOME}/Applications/flutter/bin"
    ```

<!-- https://www.cnblogs.com/lazyfang/p/7643621.html -->
<!-- https://blog.csdn.net/gausszhch/article/details/5628009 -->
<!-- https://forum.ubuntu.com.cn/viewtopic.php?t=466064#p3115352 -->
<!-- https://www.jianshu.com/p/2a699726dfad -->
<!-- https://stackoverflow.com/questions/7131670/make-a-bash-alias-that-takes-a-parameter -->
<!-- https://stackoverflow.com/a/25915221 -->
