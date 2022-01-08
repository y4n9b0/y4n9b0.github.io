---
layout: post
title:  Kotlin deserialization
date:   2022-01-08 14:30:00 +0800
categories: kotlin
tags: gson
published: true
---

* content
{:toc}

## 一、data class

```kotlin
data class Somebody(
    val name: String,
    val age: Int,
    val girlFriends: List<String> = listOf("Jane", "Lisa")
)

fun main() {
    val json = """
        {
            "name":"Bob",
            "age":30
        }
    """
    val gson = GsonBuilder().setLenient().create()

    val sb = gson.fromJson(json, Somebody::class.java)
    println("sb=$sb")   // sb=Somebody(name=Bob, age=30, girlFriends=null)
}
```

尽管我们在 data class 里对 girlFriends 设置了默认值，但是当 json 字符串缺少 girlFriends 字段时，解析出来的结果 girlFriends 依然为 null。
为什么 data class 赋予的默认值没有生效？

追踪 gson 构造对象源码：

```java
/*
 * ConstructorConstructor.get
 */
public <T> ObjectConstructor<T> get(TypeToken<T> typeToken) {
    final Type type = typeToken.getType();
    final Class<? super T> rawType = typeToken.getRawType();
	
    // ··· 省略一些缓存容器相关代码

    ObjectConstructor<T> defaultConstructor = newDefaultConstructor(rawType);
    if (defaultConstructor != null) {
      return defaultConstructor;
    }

    ObjectConstructor<T> defaultImplementation = newDefaultImplementationConstructor(type, rawType);
    if (defaultImplementation != null) {
      return defaultImplementation;
    }

    // finally try unsafe
    return newUnsafeAllocator(type, rawType);
}
```

gson 在构造对象时有三个流程：

1. newDefaultConstructor
2. newDefaultImplementationConstructor
3. newUnsafeAllocator

第一步，尝试获取了无参的构造函数，如果能够找到，则通过 newInstance 反射的方式构建对象。<br>
没有找到无参构造函数则返回 null，然后走第二步 newDefaultImplementationConstructor，这个方法里面都是一些集合类相关对象的逻辑，直接跳过。<br>
最后走第三步 newUnsafeAllocator，通过 Unsafe 的方法，绕过了构造方法，直接构建了一个对象。
<!-- https://juejin.cn/post/6844904183691214862 -->

## 二、data class with default parameterless constructor

```kotlin
data class Somebody(
    val name: String,
    val age: Int,
    val girlFriends: List<String>
) {
    @SuppressWarnings("unused") // Used by GSON
    constructor() : this("Bob", 30, listOf("Jane", "Lisa"))
}

fun main() {
    val json = """
        {
            "name":"Bob",
            "age":30
        }
    """
    val gson = GsonBuilder().setLenient().create()

    val sb = gson.fromJson(json, Somebody::class.java)
    println("sb=$sb")   // sb=Somebody(name=Bob, age=30, girlFriends=[Jane, Lisa])
}
```

Android Studio 这种 IDE 会提示这里的无参次构造函数从未使用，一定不要听从提示的建议而删掉。

## 三、data class defines default values for all fields

```kotlin
data class Somebody(
    val name: String = "Bob",
    val age: Int = 30,
    val girlFriends: List<String> = listOf("Jane", "Lisa")
)

fun main() {
    val json = """
        {
            "name":"Bob",
            "age":30
        }
    """
    val gson = GsonBuilder().setLenient().create()

    val sb = gson.fromJson(json, Somebody::class.java)
    println("sb=$sb")   // sb=Somebody(name=Bob, age=30, girlFriends=[Jane, Lisa])
}
```

data class 所有 fields 都设置默认值后，kotlin compiler 会生成默认的无参构造函数。
<!-- https://proandroiddev.com/safe-parsing-kotlin-data-classes-with-gson-4d560fe3cdd2 -->


如果所有 fields 都设置了默认值，同时又显式声明了无参构造函数，则以无参构造函数中的默认值为准（显然的，毕竟无参构造函数最终调用主构造函数，会覆盖掉主构造函数里的默认值）：
```kotlin
data class Somebody(
    val name: String = "Bob",
    val age: Int = 30,
    val girlFriends: List<String> = listOf("Jane", "Lisa")
) {
    constructor() : this("Bob", 30, listOf("Jane", "Daddario"))
}

fun main() {
    val json = """
        {
            "name":"Bob",
            "age":30
        }
    """
    val gson = GsonBuilder().setLenient().create()

    val sb = gson.fromJson(json, Somebody::class.java)
    println("sb=$sb")   // sb=Somebody(name=Bob, age=30, girlFriends=[Jane, Daddario])
}
```

## 四、normal class

```kotlin
/**
 * 也可以加上注解 @JvmOverloads 重载构造函数（关键依然是必须给构造函数里所有的 field 赋默认值）：
 *
 * class Somebody @JvmOverloads constructor(
 *     val name: String = "Bob",
 *     val age: Int = 30,
 *     val girlFriends: List<String> = listOf("Jane", "Lisa")
 * ) {
 *     override fun toString() = "Somebody(name=$name, age=$age, girlFriends=$girlFriends)"
 * }
 */
class Somebody(
    val name: String = "Bob",
    val age: Int = 30,
    val girlFriends: List<String> = listOf("Jane", "Lisa")
) {
    override fun toString() = "Somebody(name=$name, age=$age, girlFriends=$girlFriends)"
}

fun main() {
    val json = """
        {
            "name":"Bob",
            "age":30
        }
    """
    val gson = GsonBuilder().setLenient().create()

    val sb = gson.fromJson(json, Somebody::class.java)
    println("sb=$sb")   // sb=Somebody(name=Bob, age=30, girlFriends=[Jane, Lisa])
}
```

or

```kotlin
class Somebody {
    val name: String = "Bob"
    val age: Int = 30
    val girlFriends: List<String> = listOf("Jane", "Lisa")

    override fun toString() = "Somebody(name=$name, age=$age, girlFriends=$girlFriends)"
}

fun main() {
    val json = """
        {
            "name":"Bob",
            "age":30
        }
    """
    val gson = GsonBuilder().setLenient().create()

    val sb = gson.fromJson(json, Somebody::class.java)
    println("sb=$sb")   // sb=Somebody(name=Bob, age=30, girlFriends=[Jane, Lisa])
}
```

这两种普通类的写法，kotlin compile 都会自动生成默认的无参构造函数。<br>
其实跟 data class 类似，只要保证有参构造函数的 filed 都赋有默认值即可（只不过 data class 主构造函数至少要有一个参数）。

## 五、deserialization post-processing

* 完整的 json 字符串：`{"name":"Bob","age":30,"girlFriends":["Jane","Lisa"]}`
* 缺少 girlFriends 字段的 json：`{"name":"Bob","age":30}`
* 不缺字段但字段 girlFriends 对应值为 null 的 json：`{"name":"Bob","age":30,"girlFriends":null}`

上文测试代码的 json 串都缺少 girlFriends 这个字段，现在让我们做一点小小的改动，添加上 girlFriends 字段，但是其值设置为 null。<br>
如此一来，以上三种修复方式(带无参构造函数的数据类、所有 fields 都设置默认值的数据类、普通类)又失效了。

接下来，用另外一种方式来解决这两种异常 json 字符串的反序列化空安全问题：

```kotlin
interface IAfterDeserializeAction {
    fun doAfterDeserialize()
}

class DeserializeActionAdapterFactory : TypeAdapterFactory {
    override fun <T : Any?> create(gson: Gson, type: TypeToken<T>): TypeAdapter<T> {
        // 获取其他低优先级Factory创建的DelegateAdapter
        val delegate = gson.getDelegateAdapter(this, type)
        return if (IAfterDeserializeAction::class.java.isAssignableFrom(type.rawType)) {
            // 如果type实现了DeserializeAction，则返回包裹后的TypeAdapter
            object : TypeAdapter<T>() {
                @Throws(IOException::class)
                override fun write(out: JsonWriter?, value: T) = delegate.write(out, value)

                @Throws(IOException::class)
                override fun read(`in`: JsonReader?): T? = delegate.read(`in`).also {
                    (it as? IAfterDeserializeAction)?.doAfterDeserialize()
                }
            }
        } else delegate
    }
}


data class Somebody(
    var name: String,
    var age: Int,
    var girlFriends: List<String>
) : IAfterDeserializeAction {

    override fun doAfterDeserialize() {
        name = name ?: "Bob"
        age = if (age != 0) age else 30
        girlFriends = girlFriends ?: listOf("Jane", "Lisa")
    }
}

fun main() {
    // val json = """
    //     {
    //         "name":"Bob",
    //         "age":30
    //     }
    // """

    val json = """
        {
            "name":"Bob",
            "age":30,
            "girlFriends":null
        }
    """
    val gson = GsonBuilder().setLenient()
        .registerTypeAdapterFactory(DeserializeActionAdapterFactory())
        .create()

    val sb = gson.fromJson(json, Somebody::class.java)
    println("sb=$sb")   // sb=Somebody(name=Bob, age=30, girlFriends=[Jane, Lisa])
}
```

* 这种方式的 data class fields 必须声明为 var 可变的，以便于 doAfterDeserialize 中赋值。
  <!-- https://medium.com/mobile-app-development-publication/post-processing-on-gson-deserialization-26ce5790137d -->
  <!-- https://blog.simplypatrick.com/tils/2016/2016-03-02-post-processing-gson-deserialization/ -->
  <!-- https://stackoverflow.com/questions/10596667/how-to-invoke-default-deserialize-with-gson -->
* 需要注意，对于诸如 Int 这样的基本数据类型(Int、Long、Float、Double、Boolean)，json缺失字段或者字段值为null 解析出来默认为0（Boolean类型默认为false）。故应该是 `age = if (age != 0) age else 30` 而不是 `age = age ?: 30`

<br>

对于 Android 开发中常使用的 Retrofit，可参照如下方式接入：
```kotlin
val gson = GsonBuilder().setLenient().registerTypeAdapterFactory(DeserializeActionAdapterFactory()).create()
val retrofit = Retrofit.Builder().addConverterFactory(GsonConverterFactory.create(gson)).build()
```

## 六、SerializedName annotation and back field

利用 gson @SerializedName 注解和 kotlin back field 的方式：
<!-- https://proandroiddev.com/most-elegant-way-of-using-gson-kotlin-with-default-values-and-null-safety-b6216ac5328c -->

```kotlin
data class Somebody(
    @field:SerializedName("name") private val _name: String?,
    @field:SerializedName("age") private val _age: Int,
    @field:SerializedName("girlFriends") private val _girlFriends: List<String>?
) {
    val name: String
        get() = _name ?: "Bob"

    val age: Int
        get() = if (_age != 0) _age else 30

    val girlFriends: List<String>
        get() = _girlFriends ?: listOf("Jane", "Lisa")

    override fun toString() = "Somebody(name=$name, age=$age, girlFriends=$girlFriends)"
}

fun main() {
    val json = """
        {
            "name":"Bob",
            "age":30,
            "girlFriends":null
        }
    """
    val gson = GsonBuilder().setLenient().create()

    val sb = gson.fromJson(json, Somebody::class.java)
    println("sb=$sb")   // sb=Somebody(name=Bob, age=30, girlFriends=[Jane, Lisa])
}
```

## 七、gson vs moshi vs kotlinx.serialization

// todo
<!-- https://engineering.kitchenstories.io/data-classes-and-parsing-json-a-story-about-converting-models-to-kotlin-caf8a599df9e -->
<!-- https://blog.csdn.net/qq_20330595/article/details/89205377 -->


<!-- 
你真的会用Gson吗?Gson使用指南 - 怪盗kidou
https://www.jianshu.com/p/e740196225a4
https://www.jianshu.com/p/c88260adaf5e
https://www.jianshu.com/p/0e40a52c0063
https://www.jianshu.com/p/3108f1e44155
https://www.jianshu.com/p/d62c2be60617

你真的会用Retrofit2吗?Retrofit2完全教程 - 怪盗kidou
https://www.jianshu.com/p/308f3c54abdd
-->

<!-- 
Gson全解析 - 程序员Anthony
https://www.jianshu.com/p/fc5c9cdf3aab
https://www.jianshu.com/p/8cc857583ff4
https://www.jianshu.com/p/17a68d4fffbe
-->
