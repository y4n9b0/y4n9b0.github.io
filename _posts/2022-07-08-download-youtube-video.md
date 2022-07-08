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

这里推荐 [savefrom](https://en.savefrom.net/){:target="_blank"} 在线下载 youtube 视频，
[DownSub](https://downsub.com/){:target="_blank"} 在线下载字幕，
再使用 FFmpeg 合成视频和字幕（虽然 FFmpeg 也需要安装，但恰好做多媒体相关的东西，早已安装 FFmpeg）。

## FFmpeg 合成视频、字幕

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

<!-- https://zhuanlan.zhihu.com/p/349506890 -->
<!-- http://www.caotama.com/1982370.html -->