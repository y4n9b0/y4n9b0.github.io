---
layout: post
title: Print in Order
date: 2021-11-03 13:30:00 +0800
categories: algorithm
tags: leetcode
published: true
---

* content
{:toc}

## LeetCode 1114

[Print in Order](https://leetcode.com/problems/print-in-order/){:target="_blank"}

```java
// java
import java.util.concurrent.*;
class Foo {
    Semaphore run2, run3;

    public Foo() {
        run2 = new Semaphore(0);
        run3 = new Semaphore(0);
    }

    public void first(Runnable printFirst) throws InterruptedException {
        printFirst.run();
        run2.release();
    }

    public void second(Runnable printSecond) throws InterruptedException {
        run2.acquire();
        printSecond.run();
        run3.release();
    }

    public void third(Runnable printThird) throws InterruptedException {
        run3.acquire(); 
        printThird.run();
    }
}
```

另外，还有使用 [ReentrantLock Condition](https://leetcode.com/problems/print-in-order/discuss/893827/Java-SynchronizedLockSemaphoreCondition-Variable){:target="_blank"} 以及 [AtomicInteger](https://leetcode.com/problems/print-in-order/discuss/332890/Java-Basic-semaphore-solution-8ms-36MB/318872){:target="_blank"} 的方式。

## 阿里面试题

四个线程，串行执行依次打印数字1到1000

线程1打印：1  5  9 ...<br>
线程2打印：2  6  10 ...<br>
线程3打印：3  7  11 ...<br>
线程4打印：4  8  12 ...<br>

最后打印的结果：1 2 3 4 5 6 7 8 9 ...

```java
// java
public static void main(String[] args) {
    final int NUMBER_SIZE = 1000;
    final int THREAD_SIZE = 4;

    List<Semaphore> semaphores = new ArrayList<>();
    for (int i = 0; i < THREAD_SIZE; i++) {
        semaphores.add(new Semaphore(i == 0 ? 1 : 0));
    }

    List<Thread> threads = new ArrayList<>();
    for (int i = 0; i < THREAD_SIZE; i++) {
        final int index = i;
        Thread t = new Thread(() -> {
            int n = 0;
            while (n < NUMBER_SIZE / THREAD_SIZE) {
                try {
                    semaphores.get(index).acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(index + n * THREAD_SIZE + 1);
                n++;
                semaphores.get((index + 1) % THREAD_SIZE).release();
            }
            if (NUMBER_SIZE % THREAD_SIZE != 0 && index + n * THREAD_SIZE < NUMBER_SIZE) {
                try {
                    semaphores.get(index).acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(index + n * THREAD_SIZE + 1);
                semaphores.get((index + 1) % THREAD_SIZE).release();
            }
        });
        threads.add(t);
    }

    for (int i = 0; i < THREAD_SIZE; i++) {
        threads.get(i).start();
    }
}
```

```kotlin
// kotlin
fun main() {
    print(1000, 4)
}

fun print(numberSize: Int, threadSize: Int) {
    val semaphores: MutableList<Semaphore> = MutableList(threadSize) {
        Semaphore(if (it == 0) 1 else 0)
    }

    val threads: MutableList<Thread> = MutableList(threadSize) {
        Thread {
            var n = 0
            while (n < numberSize / threadSize) {
                try {
                    semaphores[it].acquire()
                } catch (e: InterruptedException) {
                    e.printStackTrace()
                }
                println(it + n * threadSize + 1)
                n++
                semaphores[(it + 1) % threadSize].release()
            }
            if (numberSize % threadSize != 0 && it + n * threadSize < numberSize) {
                try {
                    semaphores[it].acquire()
                } catch (e: InterruptedException) {
                    e.printStackTrace()
                }
                println(it + n * threadSize + 1)
                semaphores[(it + 1) % threadSize].release()
            }
        }
    }

    for (i in 0 until threadSize) {
        threads[i].start()
    }
}
```

<!-- https://www.cnblogs.com/jyx140521/p/6747750.html -->
