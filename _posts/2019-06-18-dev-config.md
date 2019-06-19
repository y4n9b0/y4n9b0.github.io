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

## chrome 插件

* [OneTab](https://chrome.google.com/webstore/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall)
* [Octotree](https://chrome.google.com/webstore/detail/octotree/bkhaagjahfmjljalopjnoealnfndnagc/related)
* [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop)
* [Tampermonkey](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo?utm_campaign=en)

## .vimrc

```vim
set nocompatible            " 关闭vi兼容模式
set shiftwidth=4            " 设置缩进为4个空格
set tabstop=4               " 设置tab键4个空格长度
set expandtab               " 转换tab为空格
set smartindent             " 开启smart缩进
set number                  " 显示行号
syntax on                   " 开启颜色高亮
colorscheme desert          " 设置配色方案
set cursorline              " 突出显示当前行
highlight CursorLine cterm=NONE ctermbg=black ctermfg=green guibg=black guifg=green
set ruler                   " 打开状态栏标尺
set incsearch               " 输入搜索内容时就显示搜索结果
set hlsearch                " 搜索时高亮显示被找到的文本
```

## .tmux.conf

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
