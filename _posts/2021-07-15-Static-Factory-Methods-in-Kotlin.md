---
layout: post
title: Static Factory Methods in Kotlin
date: 2021-07-15 17:10:00 +0800
categories: Kotlin
tags: design-pattern
published: true
---

* content
{:toc}

优雅高效的Kotlin静态工厂方法：

```kotlin
interface Car {
    val price: Int

    companion object {
        operator fun invoke(brand: Brand): Car {
            return when (brand) {
                is Brand.Audi -> Audi()
                is Brand.Bmw -> Bmw()
                is Brand.Benz -> Benz()
            }
        }
    }
}

sealed class Brand {
    object Audi : Brand()
    object Bmw : Brand()
    object Benz : Brand()
}

class Audi : Car {
    override val price = 300000
}

class Bmw : Car {
    override val price = 400000
}

class Benz : Car {
    override val price = 500000
}

fun main() {
    val car = Car(Brand.Audi)
    println(car.price)
}
```
