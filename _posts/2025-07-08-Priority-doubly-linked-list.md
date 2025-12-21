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
class PriorityDoublyLinkedList<E>(
    private val capacity: Int = 9527,
    private val comparator: Comparator<E>
) : Iterable<E> {

    private var head: Node<E>? = null
    private var tail: Node<E>? = null

    // object pool: a stack-based recycled Node chain
    private var recycled: Node<E>? = null

    // max recycled size: capacity + 1
    private var recycledSize = 0

    var size: Int = 0
        private set

    fun add(element: E) {
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

    fun addAll(elements: Collection<E>) = elements.forEach(::add)

    operator fun plusAssign(element: E) = add(element)

    operator fun plusAssign(elements: Collection<E>) = addAll(elements)

    fun headOrNull(): E? = head?.element

    fun tailOrNull(): E? = tail?.element

    fun firstOrNull(predicate: (E) -> Boolean): E? {
        var node = head
        while (node != null && !predicate(node.element)) node = node.next
        return node?.element
    }

    fun lastOrNull(predicate: (E) -> Boolean): E? {
        var node = tail
        while (node != null && !predicate(node.element)) node = node.prev
        return node?.element
    }

    fun removeHead(): E? {
        val node = head ?: return null
        head = node.next?.apply { prev = null }
        if (tail == node) tail = null
        val element = node.element
        recycleNode(node)
        size--
        return element
    }

    fun removeTail(): E? {
        val node = tail ?: return null
        tail = node.prev?.apply { next = null }
        if (head == node) head = null
        val element = node.element
        recycleNode(node)
        size--
        return element
    }

    fun removeFirst(predicate: (E) -> Boolean): E? {
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

    fun removeLast(predicate: (E) -> Boolean): E? {
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

    fun removeAt(index: Int): E {
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

    fun removeAll(predicate: (E) -> Boolean): List<E> {
        val list = mutableListOf<E>()
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

    fun clear(clearRecycled: Boolean = false) {
        if (clearRecycled) {
            recycled = null
            recycledSize = 0
        } else {
            var node = head
            while (node != null) {
                val next = node.next
                recycleNode(node)
                node = next
            }
        }
        head = null
        tail = null
        size = 0
    }

    fun isEmpty(): Boolean = size == 0

    fun isNotEmpty(): Boolean = size != 0

    // region PriorityDoublyLinkedList 实现了 Iterable，标准库中已经对 Iterable 扩展了如下方法
    // fun indexOf(element: E): Int {
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
    // fun contains(element: E): Boolean = any { it == element }
    //
    // fun any(predicate: (E) -> Boolean): Boolean {
    //    for (element in this) if (predicate(element)) return true
    //    return false
    // }
    //
    // fun all(predicate: (E) -> Boolean): Boolean {
    //    for (element in this) if (!predicate(element)) return false
    //    return true
    // }
    //
    // fun forEach(action: (E) -> Unit) {
    //    for (element in this) action(element)
    // }
    //
    // fun toList(): List<E> {
    //    val list = mutableListOf<E>()
    //    forEach(list::add)
    //    return list
    // }
    // endregion

    override operator fun iterator(): Iterator<E> = object : Iterator<E> {
        private var node = head
        override fun hasNext(): Boolean = node != null
        override fun next(): E {
            val element = node?.element ?: throw NoSuchElementException()
            node = node?.next
            return element
        }
    }

    operator fun get(index: Int): E = getNodeAt(index).element

    fun getOrNull(index: Int): E? {
        if (index < 0 || index >= size) return null
        var node: Node<E>?
        if (index < size / 2) {
            node = head
            repeat(index) { node = node?.next }
        } else {
            node = tail
            repeat(size - 1 - index) { node = node?.prev }
        }
        return node?.element
    }

    fun printStructure() {
        println("---- PriorityDoublyLinkedList ----")
        println("Size = $size")
        var node = head
        var count = 0
        while (node != null) {
            println("    [$count] $node")
            node = node.next
            count++
        }
        println("Recycled Size = $recycledSize")
        var recycledNode = recycled
        count = 0
        while (recycledNode != null) {
            println("  ♻️[$count] $recycledNode")
            recycledNode = recycledNode.next
            count++
        }
        println("----------------------------------")
    }

    private fun getNodeAt(index: Int): Node<E> {
        if (index < 0 || index >= size) {
            throw IndexOutOfBoundsException("Index: $index, Size: $size")
        }
        var node: Node<E>?
        if (index < size / 2) {
            node = head
            repeat(index) { node = node?.next }
        } else {
            node = tail
            repeat(size - 1 - index) { node = node?.prev }
        }
        return node ?: error("Index: $index, Size: $size")
    }

    private fun obtainNode(element: E): Node<E> {
        val node = recycled
        return if (node != null) {
            recycled = node.next
            recycledSize--
            node.element = element
            node.next = null
            node.prev = null
            node
        } else {
            Node(_element = element)
        }
    }

    private fun recycleNode(node: Node<E>) {
        if (recycledSize > capacity) return
        node._element = null
        node.next = recycled
        node.prev = null
        recycled = node
        recycledSize++
    }

    @Suppress("PropertyName")
    private data class Node<E>(var _element: E?) {
        var prev: Node<E>? = null
        var next: Node<E>? = null

        var element: E
            get() = _element ?: error("Element is null or has been recycled!")
            set(value) {
                _element = value
            }

        override fun toString(): String {
            return "Node@${System.identityHashCode(this).toString(16)}(element=${_element})"
        }
    }
}
```
