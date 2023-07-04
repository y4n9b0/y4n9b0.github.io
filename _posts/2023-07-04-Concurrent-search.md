---
layout: post
title: Concurrent Search
date: 2023-07-04 14:30:00 +0800
categories: concurrent
tags: concurrent
published: true
---

* content
{:toc}

## BFS

单线程层级优先搜索：

```kotlin
    // coroutine 版本
    fun bfs(
        root: File = Environment.getExternalStorageDirectory(),
        onComplete: (List<File>) -> Unit = {}
    ): Job {
        return CoroutineScope(Dispatchers.IO).launch {
            val result: MutableList<File> = mutableListOf()
            val queue: Queue<File> = LinkedList()
            queue.offer(root)
            while (queue.isNotEmpty()) {
                var size = queue.size
                while (size-- > 0) {
                    val file = queue.poll() ?: continue
                    Log.d("Bob", "thread=${Thread.currentThread().name} file=${file}")
                    if (file.isDirectory) {
                        file.listFiles()?.forEach { queue.offer(it) }
                    } else {
                        result.add(file)
                    }
                }
            }
            withContext(Dispatchers.Main) {
                onComplete(result)
            }
        }
    }
```

```kotlin
    // thread 版本
    fun bfs(
        root: File = Environment.getExternalStorageDirectory(),
        onComplete: (List<File>) -> Unit = {}
    ) {
        Thread(Runnable {
            val result: MutableList<File> = mutableListOf()
            val queue: Queue<File> = LinkedList()
            queue.offer(root)
            while (queue.isNotEmpty()) {
                var size = queue.size
                while (size-- > 0) {
                    val file = queue.poll() ?: continue
                    if (file.isDirectory) {
                        file.listFiles()?.forEach { queue.offer(it) }
                    } else {
                        result.add(file)
                    }
                }
            }
            Handler(Looper.getMainLooper()).post { onComplete(result) }
        }).start()
    }
```

## 并发搜索

并发需要几个元素：

* 一个线程池，执行 task
* 一个队列，保存 task

```kotlin       
    fun concurrentBfs(
        threadCount: Int = Runtime.getRuntime().availableProcessors(),
        root: File = Environment.getExternalStorageDirectory(),
        onComplete: (List<File>) -> Unit
    ) {
        val result: ConcurrentLinkedQueue<File> = ConcurrentLinkedQueue()
        val queue: LinkedBlockingQueue<File> = LinkedBlockingQueue()
        queue.offer(root)

        CoroutineScope(Dispatchers.Main).launch {
            val threadPool: ExecutorService = Executors.newFixedThreadPool(threadCount)
            while (queue.isNotEmpty()) {
                val list = queue.toList()
                Log.d("Bob", "dispatch new task size=${list.size}")
                queue.clear()
                list.map { file ->
                    async(threadPool.asCoroutineDispatcher()) {
                        Log.d("Bob", "thread=${Thread.currentThread().name} file=${file}")
                        if (file.isDirectory) {
                            file.listFiles()?.forEach { queue.offer(it) }
                        } else {
                            result.offer(file)
                        }
                    }
                }.awaitAll()
            }
            threadPool.shutdown()
            onComplete(result.toList())
        }
    }
```

```kotlin
    // Warning: 反面教材，千万不要写成这样。虽然看起来使用了线程池，但是单个 Deferred await 阻塞了 task 分配，导致同一时间只有一个 task 在运行（每次随机运行在某个线程上）
    fun concurrentSearch(
        threadCount: Int = Runtime.getRuntime().availableProcessors(),
        root: File = Environment.getExternalStorageDirectory(),
        onComplete: (List<File>) -> Unit = {}
    ) {
        CoroutineScope(Dispatchers.Main).launch {
           val threadPool: ExecutorService = Executors.newFixedThreadPool(threadCount)
            val dispatcher = threadPool.asCoroutineDispatcher()
            val result: ConcurrentLinkedQueue<File> = ConcurrentLinkedQueue()
            val queue: ConcurrentLinkedQueue<File> = ConcurrentLinkedQueue()
            queue.offer(root)
            while (queue.isNotEmpty()) {
                Log.d("Bob", "dispatch new task")
                async(dispatcher) {
                    val file = queue.poll() ?: return@async
                    Log.d("Bob",  "thread=${Thread.currentThread().name} file=${file}")
                    if (file.isDirectory) {
                        file.listFiles()?.forEach { queue.offer(it) }
                    } else {
                        result.offer(file)
                    }
                }.await()
            }
            threadPool.shutdown()
            onComplete(result.toList())
        }
    }
```

怎么样分割 task 很关键，这里我们将每一个文件节点看作一个 task，task 里对文件进行识别，如果是文件直接加入结果列表，如果是文件夹则将其所有子文件加入任务队列。这个方法很完美，实现也很优雅，但是在测试手机上实际运行结果很意外，耗时比上面的 BFS 高。

分析了下原因，大概如下几个：
1. 线程数设置不太合适（默认使用处理器核心数量）。
2. 文件读写属于系统 IO 操作，最终都会汇聚到同一个 IO 工作线程，系统的 IO 线程成了瓶颈。
3. 测试手机上文件数量过少，多线程大量的耗时都花在了入队列、取队列、切换 task 上了。

#### 针对原因一

尝试过设置更少的线程数，耗时相差不大，所以只要线程数设置不太离谱，基本可以排除。

#### 针对原因二

如果瓶颈是卡在系统文件 IO 上的话，那理论上多线程和单线程耗时应该相差不大才对。系统文件 IO 瓶颈会是个问题，但应该不是这里耗时剧增的原因。

#### 针对原因三

为减少队列操作，尝试优化并发 task，计划按 root 子文件来划分 task，每一个 task 再做 BFS 搜索。
由于 root 各子文件下的文件数量不一致，可能造成 task 严重不均匀，我们需要多划分一些 task（并不是越多越好，原因有二：一划分是单线程bfs搜索进行的，二是task越多线程task切换越频繁），尽可能减少每个线程 task 不均匀的情况。
这里改写 bfs，增加层级 level 参数（当 level = -1 时代表搜索所有层级），并返回 pair，一个代表已搜索到的文件列表，
一个代表还未搜索的文件列表：

```kotlin
    fun concurrentSearch(
        threadCount: Int = Runtime.getRuntime().availableProcessors(),
        root: File = Environment.getExternalStorageDirectory(),
        onComplete: (List<File>) -> Unit = {}
    ) {
        val result: ConcurrentLinkedQueue<File> = ConcurrentLinkedQueue()
        // bfs 搜索两个层级，然后按未搜索文件划分 task，每个 task 各自做完整的 bfs 搜索
        val (founded, toBeFound) = bfs(root, 2)
        result.addAll(founded)
        CoroutineScope(Dispatchers.Main).launch {
            val threadPool: ExecutorService = Executors.newFixedThreadPool(threadCount)
            toBeFound.map { file ->
                async(threadPool.asCoroutineDispatcher()) {
                    result.addAll(bfs(file).first)
                }
            }.awaitAll()
            threadPool.shutdown()
            onComplete(result.toList())
        }
    }

    fun bfs(
        root: File = Environment.getExternalStorageDirectory(),
        level: Int = -1 // -1: search all level
    ): Pair<List<File>, List<File>> {
        val founded: MutableList<File> = mutableListOf()
        val toBeFound: Queue<File> = LinkedList()
        toBeFound.offer(root)
        var count = 0
        while (toBeFound.isNotEmpty() && (level == -1 || (level != -1 && count < level))) {
            var size = toBeFound.size
            while (size-- > 0) {
                val file = toBeFound.poll() ?: continue
                if (file.isDirectory) {
                    file.listFiles()?.forEach { toBeFound.offer(it) }
                } else {
                    founded.add(file)
                }
            }
            count++
        }
        return Pair(founded, toBeFound.toList())
    }
```

尝试过几种不同的 level 划分 task，比之前版本的并发搜索耗时有减少。虽然不是很多，但至少能证明入队列、取队列、task 切换频繁确实会对耗时有影响。怎么样划分 task，让 task 工作尽量均匀、数量尽量吻合线程数，是个棘手的问题。

**Note** 比对过协程切换线程和手动切换线程，开销几乎相同。当然在文件数量不多时，切换线程这部分耗时开销可能是搜索本身的许多倍。这也导致文件数量少时某些优化可能不明显

## 后记

虽然多线程在文件搜索没有太多的帮助，但不代表多线程并发没有用，只能说明不适合这个场景。对于一些任务均衡且耗时的多 task 场景，并发处理提升是显而易见的。

<!-- https://stackoverflow.com/a/15194025 -->
