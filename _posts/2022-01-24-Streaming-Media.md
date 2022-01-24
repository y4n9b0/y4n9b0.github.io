---
layout: post
title:  Stream Media
date:   2022-01-24 17:00:00 +0800
categories: media
tags: ffmpeg
published: false
---

* content
{:toc}

## 一、音视频

    AV = audio & video

## 二、FFmpeg

    Fast Forward Moving Picture Experts Group

    codec = coder & decoder

    HEVC = High Efficiency Video Coding

    流媒体传输协议(rtp/rtcp/rtsp/rtmfp/rtmp/rmps/mms/hls)
    H.264 H.265 vp8 vp9

## 三、

    MediaCodec MediaProjection

## 四、

雷霄骅 github
https://github.com/leixiaohua1020

雷霄骅 csdn
https://blog.csdn.net/leixiaohua1020/article/details/15811977
https://blog.csdn.net/leixiaohua1020/article/details/18893769
https://blog.csdn.net/leixiaohua1020/article/details/47068015
https://blog.csdn.net/leixiaohua1020/article/details/15814587

FFmpeg github
https://github.com/FFmpeg/FFmpeg

数字图像处理标准图
http://www.lenna.org/full/l_hires.jpg

ExoPlayer
https://github.com/google/ExoPlayer

FFmpeg从入门到精通——进阶篇，SEI那些事儿
https://zhuanlan.zhihu.com/p/33720871

在直播应用的开发过程中，如果把主播端消息事件传递到观众端，一般会以Instant Messaging（即时通讯）的方式传递过去，但因为消息分发通道和直播通道是分开的，因此消息与直播音视频数据的同步性就会出现很多问题。那么有没有在音视频内部传递消息的方法呢？答案是SEI。
流媒体是采用流式传输方式在网络上播放的媒体格式，视频网站内容、短视频、在线直播这些视频形态，均属于流媒体的不同分支。流媒体大致包含三个层级：码流、封装和协议。从音视频编码器输出的码流，经过某种封装格式后，经过特定的协议传输、保存，构成了流媒体世界的基础功能。
SEI即补充增强信息（Supplemental Enhancement Information），属于码流范畴，它提供了向视频码流中加入额外信息的方法，是H.264/H.265这些视频压缩标准的特性之一。

探索移动端音视频与GSYVideoPlayer之旅 ｜ Agora Talk
https://mp.weixin.qq.com/s/4s2T2B9VxoNydie8tgQCcg

术语 terms
DRM： DigitalRightsManagement 数字版权管理 https://www.zhihu.com/question/37577132
Extractor：提取器 https://www.cnblogs.com/guanxinjing/p/11378133.html

DASH：Dynamic Adaptive Streaming over HTTP 基于HTTP的动态自适应流 
https://www.jianshu.com/p/8a4733a27793
https://zh.wikipedia.org/wiki/%E5%9F%BA%E4%BA%8EHTTP%E7%9A%84%E5%8A%A8%E6%80%81%E8%87%AA%E9%80%82%E5%BA%94%E6%B5%81

HLS：HTTP Live Streaming

MQTT: Mosquitto
SEI: Supplemental Enhancement Information 补充增强信息


GSYVideoPlayer
IJKPlayer
MediaPlayer
ExoPlayer