---
layout: post
title: Staged Builder
date: 2025-11-22 10:00:00 +0800
categories: Kotlin
tags: design-pattern
published: true
---

* content
{:toc}

## 阶段式 Builder

阶段式 Builder 是一种通过类型系统在编译期强制使用者按步骤填写必需字段的模式。相比传统 Builder 在 build() 时抛出 requireNotNull 错误，阶段式 Builder 在编译期就阻止缺少必需字段的代码通过，提升 API 安全性和可用性——尤其适合 library 的公共 API。

优点：
* 编译期保证必填字段已设置（更早发现错误）。
* 能强制字段设置顺序（如果业务需要）。
* API 更自文档化（类型名即步骤名）。
* 适合对外库：降低用户配置错误、减少 runtime 检查。

缺点：
* 增加类型/接口数量，代码复杂度上升。
* 对可读性/简洁性有影响（阶段过多有啰嗦感）。

## 简单的接口链

实现：通过定义一系列接口，每个接口只暴露下一步的方法，直到最后返回 build()。<br>
典型场景：http 网络请求必须先设置 method 再设置 url，或必须设置 host 和 port 两个必填字段才可构造对象。

```kotlin
class HttpRequest private constructor(
    val method: String,
    val url: String,
    val headers: Map<String, String>,
    val body: String?,
) {

    override fun toString(): String = "HttpRequest(method=$method, url=$url, headers=$headers, body=$body)"

    companion object {
        @JvmStatic
        fun builder(): MethodStage = Builder()
    }

    interface MethodStage {
        fun method(method: String): UrlStage
    }

    interface UrlStage {
        fun url(url: String): OptionalStage
    }

    interface OptionalStage {
        fun header(name: String, value: String): OptionalStage
        fun body(body: String): OptionalStage
        fun build(): HttpRequest
    }

    private class Builder : MethodStage, UrlStage, OptionalStage {
        private var method: String = ""
        private var url: String = ""
        private val headers = mutableMapOf<String, String>()
        private var body: String? = null

        override fun method(method: String): UrlStage = apply {
            this.method = method
        }

        override fun url(url: String): OptionalStage = apply {
            this.url = url
        }

        override fun header(name: String, value: String): OptionalStage = apply {
            headers[name] = value
        }

        override fun body(body: String): OptionalStage = apply {
            this.body = body
        }

        override fun build(): HttpRequest {
            require(method.isNotBlank()) { "method required" }
            require(url.isNotBlank()) { "url required" }
            return HttpRequest(method, url, headers.toMap(), body)
        }
    }
}

fun main() {
    val req = HttpRequest.builder()
        .method("POST")
        .url("https://y4n9b0.github.io/")
        .header("Content-Type", "application/json")
        .body("{}")
        .build()
    println("req=$req")
}
```

builder 只能调用 method，调用 method 之后只能调用 url，调用 url 之后才可以调用 build 方法。

## 多个必填字段但顺序不重要

如果有多个必填字段但不需要固定顺序，可以用类型级别的“标志位”来记录哪些字段已经设置（采用泛型参数或类型标记）。<br>
下面给出一种更通用但复杂的模式：用泛型表示 set/not-set，每次调用返回一个新的 Builder 类型，直到全部标志为已设置。

```kotlin
class HttpRequest private constructor(
    val method: String,
    val url: String,
    val headers: Map<String, String>,
    val body: String?,
) {
    fun copy(block: Builder<MethodSet, UrlSet>.() -> Unit = {}): HttpRequest {
        val builder = Builder<MethodSet, UrlSet>().apply {
            method = this@HttpRequest.method
            url = this@HttpRequest.url
            headers.putAll(this@HttpRequest.headers)
            body = this@HttpRequest.body
        }
        builder.block()
        return builder.build()
    }

    override fun toString(): String = "HttpRequest(method=$method, url=$url, headers=$headers, body=$body)"

    companion object {
        @JvmStatic
        fun builder(): Builder<Unset, Unset> = Builder()

        @JvmStatic
        fun Builder<MethodSet, UrlSet>.build(): HttpRequest {
            // runtime check
            require(method.isNotBlank()) { "method required" }
            require(url.isNotBlank()) { "url required" }
            return HttpRequest(method, url, headers.toMap(), body)
        }
    }

    // Marker types
    sealed interface Marker
    object Unset : Marker
    object MethodSet : Marker
    object UrlSet : Marker

    @Suppress("UNCHECKED_CAST")
    class Builder<M : Marker, U : Marker> internal constructor() {
        internal var method: String = ""
        internal var url: String = ""
        internal val headers: MutableMap<String, String> = mutableMapOf()
        internal var body: String? = null

        fun method(method: String): Builder<MethodSet, U> = apply { this.method = method } as Builder<MethodSet, U>
        fun url(url: String): Builder<M, UrlSet> = apply { this.url = url } as Builder<M, UrlSet>
        fun header(name: String, value: String): Builder<M, U> = apply { headers[name] = value }
        fun body(body: String?): Builder<M, U> = apply { this.body = body }
        fun clearHeaders(): Builder<M, U> = apply { headers.clear() }
    }
}

fun main() {
    val req = HttpRequest.builder()
        .method("POST")
        .url("https://y4n9b0.github.io/")
        .header("Content-Type", "application/json")
        .body("{}")
        .build()
    println("req=$req")

    val request = req.copy {
        method("GET")
        body(null)
    }
    println("request=$request")
}
```

只有在同时设置了 method 和 url 之后才可以调用 build 方法。

java 调用如下：
```java
public void main() {
    HttpRequest.Builder<HttpRequest.MethodSet, HttpRequest.UrlSet> builder = HttpRequest.builder()
            .method("POST")
            .url("https://y4n9b0.github.io/")
            .header("Content-Type", "application/json")
            .clearHeaders()
            .body("{}");
    HttpRequest.build(builder);
}
```