---
layout: post
title:  win10 installation
date:   2020-08-24 10:30:00 +0800
categories: system
tags: windows
published: true
---

* content
{:toc}

## 前言

这些年几乎一直在 mac 下开发，很久没有接触过 windows 了。最近在闲鱼捡了个 nuc8i5bek，打算装个 win10 先测试一下机子。记录一下捣鼓的经历。

## 一、茴字的四种写法

### 安装 windows 的几种方式

1. 光盘，由于现在的电脑几乎都不再自带光驱，所以该方式几乎不再使用
2. 虚拟光驱，解决了电脑没有光驱的问题，但是需要有操作系统，而且受旧系统限制 不能随意切换32/64位系统
3. 硬盘安装
4. u盘启动盘，目前主流的方式

### 创建 u盘启动盘的两种方式

1. 通过 UltraISO/Nero 等软件把 windows 镜像刻录在 u盘，这种方式存在两个弊端：
    * 硬盘需要提前格式化并分好区，虽然现在的镜像在安装前也能做一些简单分区操作，但大多数情况下都不能满足我们的需求。仅适合那些已经格式化并分好区的安装
    * u盘无法再存储其他数据
2. WinPE

## 二、WinPE

### 什么是 PE

PE：Preinstall Environment，预安装环境，能够提供丰富的各种工具、可以加载操作系统镜像。

简单地说就是把 u盘分为两个区（实际不一定是这样，这里只是方便描述），一个区通过刻录安装 WinPE 系统，另一个区用来存放 windows 镜像文件。u盘启动的时候进入 PE 系统，然后通过 PE 去加载 windows 镜像 再执行安装。在不作为启动盘的时候，存放镜像的这个区也可以存放其他数据，跟平常的 u盘功能无异。

### 如何选择 PE

微软对 WinPE 的本意：企图让用户将 WinPE 系统仅仅用于维护，并设置了种种限制。这导致原生的 WinPE 不是很友好，在广大电脑爱好者的努力下将这些限制不断被突破，衍生出了各种更好用的版本。

但是，有人的地方就有江湖，PE 强大的功能使其可以在安装的时候动各种手脚，比如通过强行预装流氓软件来牟利。如何选择一个好的 PE 便成了重中之重。

一圈晒选下来，无责任推荐 [WePE](http://www.wepe.com.cn/)：

* 根据作者的[情怀](http://www.wepe.com.cn/about.html)自述来看，应该没有后门。
* 无广告
* 工具齐全
* 支持直接复制超过 4G 的镜像文件（FAT32 格式仅支持小于 4G 的单个文件，很多其他 PE 必须借助 fastcopy 工具）
* 足够精简

### 安装 WePE

* 进入官网，[下载](http://www.wepe.com.cn/download.html)最新版本的 WePE 安装包
* 运行安装包，根据提示一路无脑点下去。唯一需要注意的是选择安装介质的时候一定要选 u盘（界面会把硬盘也一同显示出来，不要选错了）

## 三、下载 windows 系统镜像

u盘启动盘装好 PE 后还缺 windows 系统镜像文件，一般获取正规的镜像文件有两种方式：

* [微软官网](https://www.microsoft.com/zh-cn/software-download/windows10ISO)
* [MSDN itellyou](https://msdn.itellyou.cn/)

**Note**

* 微软官方提供了一个安装工具 Media Creation Tool，可以自行下载并刻录 u盘作为启动盘。同时该工具还提供下载 ISO 的功能，建议使用该方式下载官网镜像。
* WePE 官网也提供了镜像下载，链接来自官网或者 MSDN itellyou。
* 镜像文件下载成功后，拷贝进 u盘启动盘（可以放多个镜像），就算真正完成了准备工作。

## 四、WePE 安装 win10

1. bios 设置 u盘启动、关闭 Secure boot。具体进 bios 的方式视主板而异，自行查阅
2. 启动 u盘，进入 WePE，格式化硬盘（NTFS）、分区：
    * UEFI + GPT(GUID分区表），支持 UEFI 引导的新款电脑需要使用 GUID 引导分区
    * Legacy + MBR，不支持 UEFI 的老款电脑选择 MBR 引导分区
3. 运行 Windows安装器（本质还是挂载虚拟光驱的方式），选择对应的 ISO 镜像文件，选择对应的安装盘符、引导盘(UEFI 模式一般已默认选中 ESP 分区，无须再选择)，一路傻瓜式安装，可参考 WePE 官方[文档](http://www.wepe.com.cn/ubook/)
4. 系统安装好后检查、下载系统更新
5. 激活系统，推荐战斗民族的[数字激活工具](https://www.lanzous.com/i1u5m9c)
6. 检查硬件管理器是否驱动齐全、更新驱动

## 后记

至此，win10 的安装就结束了。当然，除了 win10 u盘启动盘还可以安装其他系统，只需要拷贝进去对应的镜像文件即可。</br>
下回咱们聊聊 nuc8 Hackintosh

**缩写**

* EFI  - Extensible Firmware Interface，可扩展固件接口
* UEFI - Unified EFI，统一的可扩展固件接口
* ESP  - EFI EFI system partition，EFI系统分区
* GUID - Globally Unique Identifier，全局唯一标识符
* GPT  - GUID Partition Table，GUID分区表
* BIOS - Basic Input Output System，基本输入输出系统
* MBR  - Master Boot Record，主引导记录
* GRUB - GRand Unified Bootloader

<!-- todo：上传数字激活工具至github，替换下载链接 -->
<!-- http://bbs.wuyou.net/forum.php -->
<!-- https://www.ventoy.net/cn/index.html -->
<!-- https://www.zhihu.com/question/27036799 -->