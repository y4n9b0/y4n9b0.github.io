---
layout: post
title: 优先级双向链表
date: 2025-07-08 11:55:00 +0800
categories: algorithm
tags: algorithm
published: true
---

* content
{:toc}

```kotlin
/**
 * stable sorting & thread unsafe
 */
class PriorityDoublyLinkedList<T>(
    private val capacity: Int = 100,
    private val comparator: Comparator<T>
) {

    private var head: Node<T>? = null
    private var tail: Node<T>? = null

    // object pool: a stack-based recycled Node chain
    private var recycled: Node<T>? = null

    // max recycled size: capacity + 1
    private var recycledSize = 0

    var size: Int = 0
        private set

    fun add(value: T) {
        val node = obtainNode(value)
        var curr = head
        while (curr != null && comparator.compare(curr.value, node.value) <= 0) {
            curr = curr.next
        }
        when (curr) {
            head -> {
                node.next = head
                head?.prev = node
                head = node
                if (tail == null) tail = node
            }

            null -> {
                tail?.next = node
                node.prev = tail
                tail = node
            }

            else -> {
                curr.prev?.next = node
                node.prev = curr.prev
                node.next = curr
                curr.prev = node
            }
        }
        size++
        if (size > capacity) pollTail()
    }

    fun addAll(list: List<T>) = list.forEach(::add)

    operator fun plus(value: T): PriorityDoublyLinkedList<T> = apply {
        add(value)
    }

    operator fun plus(values: Collection<T>): PriorityDoublyLinkedList<T> = apply {
        addAll(values.toList())
    }

    fun peekHead(): T? = head?.value

    fun peekTail(): T? = tail?.value

    fun pollHead(): T? {
        val node = head ?: return null
        head = node.next?.apply { prev = null }
        if (tail == node) tail = null
        val result = node.value
        recycleNode(node)
        size--
        return result
    }

    fun pollTail(): T? {
        val node = tail ?: return null
        tail = node.prev?.apply { next = null }
        if (head == node) head = null
        val result = node.value
        recycleNode(node)
        size--
        return result
    }

    fun peekFirst(predicate: (T) -> Boolean): T? {
        var curr = head
        while (curr != null && !predicate(curr.value)) curr = curr.next
        return curr?.value
    }

    fun peekLast(predicate: (T) -> Boolean): T? {
        var curr = tail
        while (curr != null && !predicate(curr.value)) curr = curr.prev
        return curr?.value
    }

    fun pollFirst(predicate: (T) -> Boolean): T? {
        var curr = head
        while (curr != null) {
            val next = curr.next
            if (predicate(curr.value)) {
                curr.prev?.next = curr.next
                curr.next?.prev = curr.prev
                if (head == curr) head = curr.next
                if (tail == curr) tail = curr.prev
                val result = curr.value
                recycleNode(curr)
                size--
                return result
            }
            curr = next
        }
        return null
    }

    fun pollLast(predicate: (T) -> Boolean): T? {
        var curr = tail
        while (curr != null) {
            val prev = curr.prev
            if (predicate(curr.value)) {
                curr.prev?.next = curr.next
                curr.next?.prev = curr.prev
                if (head == curr) head = curr.next
                if (tail == curr) tail = curr.prev
                val result = curr.value
                recycleNode(curr)
                size--
                return result
            }
            curr = prev
        }
        return null
    }

    fun removeAll(predicate: (T) -> Boolean): List<T> {
        val list = mutableListOf<T>()
        var curr = head
        while (curr != null) {
            val next = curr.next
            if (predicate(curr.value)) {
                curr.prev?.next = curr.next
                curr.next?.prev = curr.prev
                if (head == curr) head = curr.next
                if (tail == curr) tail = curr.prev
                list += curr.value
                recycleNode(curr)
                size--
            }
            curr = next
        }
        return list
    }

    fun removeAt(index: Int): T {
        if (index < 0 || index >= size) {
            throw IndexOutOfBoundsException("Index: $index, Size: $size")
        }
        var curr = head
        repeat(index) { curr = curr?.next }
        val node = curr ?: error("Index: $index, Size: $size")
        node.prev?.next = node.next
        node.next?.prev = node.prev
        if (head == node) head = node.next
        if (tail == node) tail = node.prev
        val result = node.value
        recycleNode(node)
        size--
        return result
    }

    fun clear() {
        var curr = head
        while (curr != null) {
            val next = curr.next
            recycleNode(curr)
            curr = next
        }
        head = null
        tail = null
        size = 0
    }

    operator fun get(index: Int): T {
        if (index < 0 || index >= size) {
            throw IndexOutOfBoundsException("Index: $index, Size: $size")
        }
        var i = 0
        for (item in this) if (i++ == index) return item
        error("Index: $index, Size: $size")
    }

    operator fun iterator(): Iterator<T> = object : Iterator<T> {
        private var curr = head
        override fun hasNext(): Boolean = curr != null
        override fun next(): T {
            val value = curr?.value ?: throw NoSuchElementException()
            curr = curr?.next
            return value
        }
    }

    fun forEach(action: (T) -> Unit) {
        for (item in this) action(item)
    }

    fun toList(): List<T> {
        val list = mutableListOf<T>()
        forEach(list::add)
        return list
    }

    fun isEmpty(): Boolean = size == 0

    fun isNotEmpty(): Boolean = size != 0

    fun contains(value: T): Boolean {
        return any { it == value }
    }

    fun indexOf(value: T): Int {
        var index = 0
        for (item in this) {
            if (item == value) return index
            index++
        }
        return -1
    }

    fun any(predicate: (T) -> Boolean): Boolean {
        for (item in this) if (predicate(item)) return true
        return false
    }

    fun all(predicate: (T) -> Boolean): Boolean {
        for (item in this) if (!predicate(item)) return false
        return true
    }

    private fun obtainNode(value: T): Node<T> {
        val node = recycled
        return if (node != null) {
            recycled = node.next
            recycledSize--
            node.value = value
            node.next = null
            node.prev = null
            node
        } else {
            Node(value)
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
        var value: T,
        var next: Node<T>? = null,
        var prev: Node<T>? = null
    )
}
```
