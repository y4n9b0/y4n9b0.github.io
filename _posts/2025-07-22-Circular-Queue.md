---
layout: post
title: 循环队列
date: 2025-07-22 14:00:00 +0800
categories: algorithm
tags: algorithm
published: true
---

* content
{:toc}


## Design Circular Queue

[LeetCode 622](https://leetcode.com/problems/design-circular-queue/description/){:target="_blank"}

Design your implementation of the circular queue. The circular queue is a linear data structure in which the operations are performed based on FIFO (First In First Out) principle, and the last position is connected back to the first position to make a circle. It is also called "Ring Buffer".

One of the benefits of the circular queue is that we can make use of the spaces in front of the queue. In a normal queue, once the queue becomes full, we cannot insert the next element even if there is a space in front of the queue. But using the circular queue, we can use the space to store new values.

Implement the `MyCircularQueue` class:

* `MyCircularQueue(k)` Initializes the object with the size of the queue to be `k`.
* `int Front()` Gets the front item from the queue. If the queue is empty, return `-1`.
* `int Rear()` Gets the last item from the queue. If the queue is empty, return `-1`.
* `boolean enQueue(int value)` Inserts an element into the circular queue. Return `true` if the operation is successful.
* `boolean deQueue()` Deletes an element from the circular queue. Return `true` if the operation is successful.
* `boolean isEmpty()` Checks whether the circular queue is empty or not.
* `boolean isFull()` Checks whether the circular queue is full or not.

You must solve the problem without using the built-in queue data structure in your programming language. 

**Example 1:**

```
Input: 
["MyCircularQueue", "enQueue", "enQueue", "enQueue", "enQueue", "Rear", "isFull", "deQueue", "enQueue", "Rear"]
[[3], [1], [2], [3], [4], [], [], [], [4], []]

Output: 
[null, true, true, true, false, 3, true, true, true, 4]

Explanation:
MyCircularQueue myCircularQueue = new MyCircularQueue(3);
myCircularQueue.enQueue(1); // return True
myCircularQueue.enQueue(2); // return True
myCircularQueue.enQueue(3); // return True
myCircularQueue.enQueue(4); // return False
myCircularQueue.Rear();     // return 3
myCircularQueue.isFull();   // return True
myCircularQueue.deQueue();  // return True
myCircularQueue.enQueue(4); // return True
myCircularQueue.Rear();     // return 4
```

**Constraints:**

* 1 <= k <= 1000
* 0 <= value <= 1000
* At most 3000 calls will be made to `enQueue`, `deQueue`, `Front`, `Rear`, `isEmpty`, and `isFull`.

## Approach 1: 链表

```kotlin
/**
 * fixed capacity & thread unsafe
 */
open class CircularQueue<T>(private val capacity: Int = 9527) : Iterable<T> {

    private var head: Node<T>? = null
    private var tail: Node<T>? = null

    // object pool: a stack-based recycled Node chain
    private var recycled: Node<T>? = null

    // max recycled size: capacity + 1
    private var recycledSize = 0

    var size: Int = 0
        private set

    fun enqueue(element: T): Boolean {
        if (size == capacity) return false
        val node = obtainNode(element)
        if (head == null) {
            node.next = node
            node.prev = node
            head = node
            tail = node
        } else {
            node.next = head
            node.prev = tail
            head?.prev = node
            tail?.next = node
            tail = node
        }
        size++
        return true
    }

    fun dequeue(): T? {
        val node = head ?: return null
        val element = node.element
        if (size == 1) {
            head = null
            tail = null
        } else {
            head?.next?.prev = tail
            tail?.next = head?.next
            head = head?.next
        }
        recycleNode(node)
        size--
        return element
    }

    fun head(): T? = head?.element

    fun tail(): T? = tail?.element

    fun isEmpty(): Boolean = size == 0

    fun isFull(): Boolean = size == capacity

    fun clear() {
        var node = head
        repeat(size) {
            val next = node?.next
            node?.apply(::recycleNode)
            node = next
        }
        head = null
        tail = null
        size = 0
    }

    override operator fun iterator(): Iterator<T> = object : Iterator<T> {
        private var node = head
        private var count = 0
        override fun hasNext(): Boolean = count < size
        override fun next(): T {
            val element = node?.element ?: throw NoSuchElementException()
            node = node?.next
            count++
            return element
        }
    }

    private fun obtainNode(element: T): Node<T> {
        val node = recycled
        return if (node != null) {
            recycled = node.next
            recycledSize--
            node.element = element
            node.next = null
            node.prev = null
            node
        } else {
            Node(element)
        }
    }

    private fun recycleNode(node: Node<T>) {
        if (recycledSize > capacity) return
        node.next = recycled
        node.prev = null
        recycled = node
        recycledSize++
    }

    private class Node<T>(
        var element: T,
        var next: Node<T>? = null,
        var prev: Node<T>? = null
    )
}

class MyCircularQueue(k: Int) : CircularQueue<Int>(capacity = k) {
    fun enQueue(value: Int): Boolean = super.enqueue(value)
    fun deQueue(): Boolean = super.dequeue() != null
    fun Front(): Int = super.head() ?: -1
    fun Rear(): Int = super.tail() ?: -1
}
```

## Approach 2: 数组

```kotlin
/**
 * fixed capacity & thread unsafe
 */
open class CircularQueue<T>(private val capacity: Int = 9527) : Iterable<T> {

    @Suppress("UNCHECKED_CAST")
    private val array = arrayOfNulls<Any?>(capacity) as Array<T?>

    private var head: Int = 0
    private var tail: Int = -1

    var size: Int = 0
        private set

    fun enqueue(element: T): Boolean {
        if (size == capacity) return false
        tail = (tail + 1) % capacity
        array[tail] = element
        size++
        return true
    }

    fun dequeue(): T? {
        if (size == 0) return null
        val element = array[head]
        array[head] = null
        head = (head + 1) % capacity
        size--
        return element
    }

    fun head(): T? = if (size == 0) null else array[head]

    fun tail(): T? = if (size == 0) null else array[tail]

    fun isEmpty(): Boolean = size == 0

    fun isFull(): Boolean = size == capacity

    fun clear() {
        array.fill(null)
        head = 0
        tail = -1
        size = 0
    }

    override operator fun iterator(): Iterator<T> = object : Iterator<T> {
        private var index = head
        private var count = 0
        override fun hasNext(): Boolean = count < size
        override fun next(): T {
            val element = array[index] ?: throw NoSuchElementException()
            index = (index + 1) % capacity
            count++
            return element
        }
    }
}

class MyCircularQueue(k: Int) : CircularQueue<Int>(capacity = k) {
    fun enQueue(value: Int): Boolean = super.enqueue(value)
    fun deQueue(): Boolean = super.dequeue() != null
    fun Front(): Int = super.head() ?: -1
    fun Rear(): Int = super.tail() ?: -1
}
```
