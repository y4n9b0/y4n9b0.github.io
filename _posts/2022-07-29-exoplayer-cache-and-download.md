---
layout: post
title:  ExoPlayer cache & download
date:   2022-07-29 10:20:00 +0800
categories: publish
tags: ExoPlayer
published: true
---

* content
{:toc}

## Cache

### Evictor 应用启动从磁盘加载缓存到内存会调用 onSpanAdded

### 缓存重新播放会调用 onSpanTouched，即便设置 CacheDataSource.Factory.setCacheWriteDataSinkFactory(null) 也只是不会写缓存，但是依然会调用 onSpanTouched，因为
```java
// SimpleCache.startReadWriteNonBlocking()
if (span.isCached) {
  // Read case.
  return touchSpan(key, span);
}
```
touchSpan 会改变 CacheSpan.lastTouchTimestamp，这样设计的目的是方便 LruCache 重排序。

### 缓存和下载都是使用 Cache 、CacheWriter（CacheDataSource、CacheDataSink）、DatabaseProvider 这一套机制
如果使用了相同的 Cache（存放缓存、下载路径和内容） 和 DatabaseProvider（记录缓存、下载的内容） 实例，则缓存和下载是会相互使用的（没有测试过不同实例的情况，但理论上应该不会相互使用，而且官方也不推荐这样，官方把缓存、下载设计成同一套机制就是方便复用，节省时间、流量，提高效率）。也就是说下载过的视频再次播放是不会缓存的（即便没设置 CacheDataSource.Factory.setCacheWriteDataSinkFactory(null)），但是在此播放会改变 CacheSpan.lastTouchTimestamp（会调用 Evictor.onSpanTouched）；而缓存过的视频也不会再次下载（依然会走下载整套流程，但是查询 Cache 里存在，直接返回下载进度100并结束下载），有所不同的是下载已缓存过的视频不会改变 CacheSpan.lastTouchTimestamp。

### CacheSpan fragment size 导致同一个视频分隔为多个 CacheSpan，缓存个数需要合并同一个key的多个CacheSpan进一个 TreeSet

### 一个 m3u8 会有多个 ts，不同的key，导致被识别为多个视频（视频链接为缓存default key），可能的方法是自定义缓存 key factory

## Download

## Adaptive Stream


https://exoplayer.dev/customization.html#caching-data-loaded-from-the-network
https://exoplayer.dev/downloading-media.html
https://medium.com/google-exoplayer/downloading-streams-6d259eec7f95
https://medium.com/google-exoplayer/downloading-adaptive-streams-37191f9776e
https://www.ietf.org/rfc/rfc8216.html
https://developer.apple.com/documentation/http_live_streaming
