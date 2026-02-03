---
layout: post
title: 苹果监管机
date: 2026-02-03 19:30:00 +0800
categories: Apple
tags: DEP、MDM
published: true
---

* content
{:toc}

## DEP

苹果 DEP 一般指的是 Device Enrollment Program（设备注册计划），现在它已经并入并更名为 Apple Business Manager（ABM） / Apple School Manager（ASM） 的一部分了。
是苹果提供的一套「企业设备自动管理 & 自动入管控」机制，设备一开机，就自动被企业/学校管理，用户无法绕过。

* 开机第一次激活
* 强制登录公司 MDM
* 不允许跳过
* 重装系统也无法绕过

## MDM

MDM 一般指的是 Mobile Device Management（设备管理工具/远程控制系统），公司 IT 用它来：

* 限制你能不能装软件
* 强制加密
* 禁止拷贝 U 盘
* 远程锁机 / 抹机
* 下发证书、VPN、Wi-Fi
* 安装公司配置

有管理描述文件（Profile），可以使用如下命令查看：
```bash
$ sudo profiles show -type enrollment

Error fetching Device Enrollment configuration: Client is not DEP enabled.
```

DEP 决定“这台设备属不属于某个组织”，MDM 决定“这台设备现在被不被管”。<br>
MDM 可以被移除，自己无法彻底解绑（需要管理员），移除 MDM ≠ 设备自由。

## 查看是否是监管机

在终端输入如下命令：

```bash
$ sudo profiles status -type enrollment

Enrolled via DEP: No
MDM enrollment: No
```

如果 DEP 和 MDM 显示都为 No，代表不是监管机。还有一种查看方式就是重装系统:-)

查看当前机子上所有的配置描述文件：
```bash
$ sudo profiles list

There are no configuration profiles installed in the system domain
```

## 绕过监管锁

* [skipmdm.com](https://github.com/y4n9b0/skipmdm.com){:target="_blank"}
* [Mac Os ventura sonama，12 13 14 监管锁跳过](https://www.bilibili.com/read/cv25781723/){:target="_blank"}
* [MacOS 13/14/15(Ventura/Sonoma/Sequoia)跳过监管锁(配置锁)](https://zhuanlan.zhihu.com/p/659458982){:target="_blank"}