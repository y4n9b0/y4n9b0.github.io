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
 * fixed capacity & stable sorting & thread unsafe
 */
class PriorityDoublyLinkedList<T>(
    private val capacity: Int = 9527,
    private val comparator: Comparator<T>
) : Iterable<T> {

    private var head: Node<T>? = null
    private var tail: Node<T>? = null

    // object pool: a stack-based recycled Node chain
    private var recycled: Node<T>? = null

    // max recycled size: capacity + 1
    private var recycledSize = 0

    var size: Int = 0
        private set

    fun add(element: T) {
        val insert = obtainNode(element)
        var node = head
        while (node != null && comparator.compare(node.element, insert.element) <= 0) {
            node = node.next
        }
        when (node) {
            head -> {
                insert.next = head
                head?.prev = insert
                head = insert
                if (tail == null) tail = insert
            }

            null -> {
                tail?.next = insert
                insert.prev = tail
                tail = insert
            }

            else -> {
                node.prev?.next = insert
                insert.prev = node.prev
                insert.next = node
                node.prev = insert
            }
        }
        size++
        if (size > capacity) removeTail()
    }

    fun addAll(elements: Collection<T>) = elements.forEach(::add)

    operator fun plusAssign(element: T) = add(element)

    operator fun plusAssign(elements: Collection<T>) = addAll(elements)

    fun headOrNull(): T? = head?.element

    fun tailOrNull(): T? = tail?.element

    fun firstOrNull(predicate: (T) -> Boolean): T? {
        var node = head
        while (node != null && !predicate(node.element)) node = node.next
        return node?.element
    }

    fun lastOrNull(predicate: (T) -> Boolean): T? {
        var node = tail
        while (node != null && !predicate(node.element)) node = node.prev
        return node?.element
    }

    fun removeHead(): T? {
        val node = head ?: return null
        head = node.next?.apply { prev = null }
        if (tail == node) tail = null
        val element = node.element
        recycleNode(node)
        size--
        return element
    }

    fun removeTail(): T? {
        val node = tail ?: return null
        tail = node.prev?.apply { next = null }
        if (head == node) head = null
        val element = node.element
        recycleNode(node)
        size--
        return element
    }

    fun removeFirst(predicate: (T) -> Boolean): T? {
        var node = head
        while (node != null) {
            val next = node.next
            if (predicate(node.element)) {
                node.prev?.next = node.next
                node.next?.prev = node.prev
                if (head == node) head = node.next
                if (tail == node) tail = node.prev
                val element = node.element
                recycleNode(node)
                size--
                return element
            }
            node = next
        }
        return null
    }

    fun removeLast(predicate: (T) -> Boolean): T? {
        var node = tail
        while (node != null) {
            val prev = node.prev
            if (predicate(node.element)) {
                node.prev?.next = node.next
                node.next?.prev = node.prev
                if (head == node) head = node.next
                if (tail == node) tail = node.prev
                val element = node.element
                recycleNode(node)
                size--
                return element
            }
            node = prev
        }
        return null
    }

    fun removeAt(index: Int): T {
        val node = getNodeAt(index)
        node.prev?.next = node.next
        node.next?.prev = node.prev
        if (head == node) head = node.next
        if (tail == node) tail = node.prev
        val element = node.element
        recycleNode(node)
        size--
        return element
    }

    fun removeAll(predicate: (T) -> Boolean): List<T> {
        val list = mutableListOf<T>()
        var node = head
        while (node != null) {
            val next = node.next
            if (predicate(node.element)) {
                node.prev?.next = node.next
                node.next?.prev = node.prev
                if (head == node) head = node.next
                if (tail == node) tail = node.prev
                list += node.element
                recycleNode(node)
                size--
            }
            node = next
        }
        return list
    }

    fun clear() {
        var node = head
        while (node != null) {
            val next = node.next
            recycleNode(node)
            node = next
        }
        head = null
        tail = null
        size = 0
    }

    fun isEmpty(): Boolean = size == 0

    fun isNotEmpty(): Boolean = size != 0

    // region PriorityDoublyLinkedList 实现了 Iterable，标准库中已经对 Iterable 扩展了如下方法
    // fun indexOf(element: T): Int {
    //    var node = head
    //    var index = 0
    //    while (node != null) {
    //        if (node.element == element) return index
    //        node = node.next
    //        index++
    //    }
    //    return -1
    // }
    //
    // fun contains(element: T): Boolean = any { it == element }
    //
    // fun any(predicate: (T) -> Boolean): Boolean {
    //    for (element in this) if (predicate(element)) return true
    //    return false
    // }
    //
    // fun all(predicate: (T) -> Boolean): Boolean {
    //    for (element in this) if (!predicate(element)) return false
    //    return true
    // }
    //
    // fun forEach(action: (T) -> Unit) {
    //    for (element in this) action(element)
    // }
    //
    // fun toList(): List<T> {
    //    val list = mutableListOf<T>()
    //    forEach(list::add)
    //    return list
    // }
    // endregion

    override operator fun iterator(): Iterator<T> = object : Iterator<T> {
        private var node = head
        override fun hasNext(): Boolean = node != null
        override fun next(): T {
            val value = node?.element ?: throw NoSuchElementException()
            node = node?.next
            return value
        }
    }

    operator fun get(index: Int): T = getNodeAt(index).element

    fun getOrNull(index: Int): T? {
        if (index < 0 || index >= size) return null
        var node: Node<T>?
        if (index < size / 2) {
            node = head
            repeat(index) { node = node?.next }
        } else {
            node = tail
            repeat(size - 1 - index) { node = node?.prev }
        }
        return node?.element
    }

    private fun getNodeAt(index: Int): Node<T> {
        if (index < 0 || index >= size) {
            throw IndexOutOfBoundsException("Index: $index, Size: $size")
        }
        var node: Node<T>?
        if (index < size / 2) {
            node = head
            repeat(index) { node = node?.next }
        } else {
            node = tail
            repeat(size - 1 - index) { node = node?.prev }
        }
        return node ?: error("Index: $index, Size: $size")
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
```
