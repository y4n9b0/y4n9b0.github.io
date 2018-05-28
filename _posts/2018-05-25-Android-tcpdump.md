---
layout: post
title:  Android tcpdump
date:   2018-05-25 10:00:00 +0800
categories: Android
tags: tcpdump
published: true
---

* content
{:toc}

# Prologue
最近在研究万能钥匙，分析它的行为，需要进行tcp抓包，简单记录下流程。

# Requirements
* root的Android设备一台
* tcpdump 二进制文件
* wireshark 分析工具

# Android tcpdump
[Android tcpdump](http://www.androidtcpdump.com/)

# wireshark
[wireshark](https://www.wireshark.org/)

# Capture
* 安装tcpdump，并修改运行权限
```
adb push tcpdump /data/local/tcpdump
adb root
adb shell
cd /data/local
chmod 6755 tcpdump
```

* 抓包  
`adb shell`并进入到tcpdump所在目录`/data/local`
```
./tcpdump -i any -p -s 0 -w /sdcard/capture.pcap
```
使用 `Ctrl + C`中断抓包

* 拉取抓获的tcp/udp包
```
adb pull /sdcard/capture.pcap
```

* 用wireshark打开capture.pcap分析log
![/styles/images/tcpdump/capturelog.png]({{'/styles/images/tcpdump/capturelog.png'|prepend:site.baseurl}})

可以看到万能钥匙使用的url是`51y5.net`，我要WiFi的谐音。

**Note**
```
tcpdump 参数说明
    # "-i any": listen on any network interface
    # "-p": disable promiscuous mode (doesn't work anyway)
    # "-s 0": capture the entire packet
    # "-w": write packets to a file (rather than printing to stdout)
    ... do whatever you want to capture, then ^C to stop it ...
```

# PIE错误
在5.0之前的系统上可以正常抓包，但是在Android5.0及以后的系统用tcpdump抓包可能失败：
```
error: only position independent executables (PIE) are supported.
```

这是由于PIE安全机制所引起的，PIE机制它会随机分配程序的内存地址从而令攻击者更难发现程序的溢出漏洞。  
[PIE机制详细介绍](https://en.wikipedia.org/wiki/Position-independent_code)

PIE安全机制从Android4.1开始引入，Android L之前的系统版本并不会去检验可执行文件是否基于PIE编译出的，因此低于Android L 以前不会报错。但是Android L已经开启验证，如果调用的可执行文件不是基于PIE方式编译的，则无法运行。

有两个解决办法：
* 下载支持的新版本tcpdump
* 自行编译tcpdump源码，编译的时候加上如下的flag就行：
  ```
  LOCAL_CFLAGS += -pie -fPIE
  LOCAL_LDFLAGS += -pie -fPIE
  ```
