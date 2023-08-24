---
layout: post
title:  Download youtube video
date:   2022-07-08 20:30:00 +0800
categories: publish
tags: youtube
published: true
---

* content
{:toc}

## 在线下载 youtube 视频

无论是大名鼎鼎的 [youtube-dl](https://ytdl-org.github.io/youtube-dl/index.html){:target="_blank"} 还是 [Gihosoft TubeGet](https://www.gihosoft.com/free-youtube-downloader.html){:target="_blank"}，这一类软件都需要安装，而我只是想找个在线网站快速下载而已。

这里推荐 [y2mate](https://www.y2mate.com/){:target="_blank"} 或者 [savefrom](https://en.savefrom.net/){:target="_blank"} 在线下载 youtube 视频，
[DownSub](https://downsub.com/){:target="_blank"} 在线下载字幕，
再使用 FFmpeg 合成视频和字幕（虽然 FFmpeg 也需要安装，但恰好做多媒体相关的东西，早已安装 FFmpeg）。

**Note**
建议优先试用 y2mate，可以下载大多数 1080p，而 safefrom 很多时候不支持 1080p 的音频流。

## 在线下载 bilibili 视频

[哔哩哔哩视频解析下载](https://bilibili.iiilab.com/){:target="_blank"}

## FFmpeg

### 合成视频字幕

在 Linux/macOS 中，可以使用 ffmpeg 命令合成视频和字幕：
```bash
ffmpeg -i infile.mp4 -i infile.srt -c copy -c:s mov_text outfile.mp4
```

注意， -c copy -c:s mov_text 的顺序是非常重要的，因为这是简写。或者可以使用如下选项 -c:v copy -c:a copy -c:s mov_text  ，在这组选项里，顺序就不重要了。

其中， -i 选项用于指定需要读取的文件，在这里是视频文件与字幕文件（字幕文件 srt 与 ass 格式皆可）。

但是，对于格式为 Matroska 的视频文件（扩展名为 mkv），上述命令会提示错误信息：Subtitle codec 94213 is not supported，应当使用如下命令：
```bash
ffmpeg -i infile.mkv -i infile.srt -c copy -c:s srt outfile.mkv
```

### 截取视频片段

```bash
ffmpeg -i input.mp4 -ss 00:01:00.000 -to 00:02:00.000 -c:v libx264 output.mp4
```

* 指定开始时间 `-ss` start second，时间格式 HH:MM:SS.MILLISECONDS
* 指定结束范围
    * `-t` 指定所需剪辑的持续时间。例如，-ss 40 -t 10 指示 FFmpeg 从第 40 秒开始提取 10 秒的视频。
    * `-to` 指定结束时间。例如，-ss 40 -to 70 指示 FFmpeg 从第 40 秒到第 70 秒提取 30 秒的视频。
    注意：如果同时使用 -t 和 -to，那么只有 -t 将被使用。
* `-c:v libx264` 重新编码，确保 i 帧精确到指定起始时间对应帧

### 裁剪视频尺寸

```bash
ffmpeg -i input.mp4 -vf crop='480:360:60:0' output.mp4
```

其中 480 和 360 代表裁剪后的宽和高，60 和 0 分别代表裁剪的 x 坐标和 y 坐标（基于原始图像）

<!-- https://zhuanlan.zhihu.com/p/349506890 -->
<!-- http://www.caotama.com/1982370.html -->
<!-- https://zhuanlan.zhihu.com/p/567582170 -->
<!-- https://blog.csdn.net/tusong86/article/details/122627789 -->