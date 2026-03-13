---
layout: post
title: 鸿蒙装饰器
date: 2026-03-13 15:30:00 +0800
categories: extension-function
tags: kotlin、jvm
published: true
---

* content
{:toc}

鸿蒙的 ArkUI 是 声明式 UI 框架，使用装饰器改变变量行为。

## 状态管理

### @State

组件内部状态，用于定义组件私有状态，状态变化会自动触发 UI 刷新。<br>
只属于当前组件，子组件不能直接修改。<br>
@State 装饰的属性必须要设置初始值。

```ArkTS
@Component
export struct Foo {
  @State count: number = 0

  build() {
    Column() {
      Button("Add")
        .onClick(() => {
          this.count++
        })
      Text(this.count.toString())
    }
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```

### @Link

父子双向绑定，子组件引用父组件状态，并且可以修改父组件状态。

```ArkTS
@Component
export struct Parent {
  @State count: number = 0

  build() {
    Column() {
      Button("parent add")
        .onClick(() => {
          this.count++
        })
      Text("parent count:" + this.count)
      Child({ count: this.count })
    }
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}

@Component
export struct Child {
  @Link count: number

  build() {
    Column() {
      Button("child Add")
        .onClick(() => {
          this.count++
        })
      Text("child count:" + this.count)
    }
  }
}
```

### @Prop

父组件传入只读数据，子组件修改不影响父组件状态（更加正确的做法是子组件不要去修改，修改 @Prop 数据有点违反设计初衷）。

```ArkTS
@Component
export struct Parent {
  @State count: number = 0

  build() {
    Column() {
      Button("parent add")
        .onClick(() => {
          this.count++
        })
      Text("parent count:" + this.count)
      Child({ count: this.count })
    }
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}

@Component
export struct Child {
  @Prop count: number

  build() {
    Column() {
      Button("child Add")
        .onClick(() => {
          this.count++
        })
      Text("child count:" + this.count)
    }
  }
}
```

### @Provide @Consume

@Provide 向任意层级的子孙组件提供数据，需要子孙组件搭配 @Consume 来消费 Provide 数据（构造子孙组件不再需要传递该数据）。

`@State + @Link` 仅用于父子组件双向绑定数据，`@Provide + @Consume` 可以用于父组件与任意层级子孙组件之间传递数据。

```ArkTS
@Component
export struct Parent {
  @Provide count: number = 0

  build() {
    Column() {
      Button("parent add")
        .onClick(() => {
          this.count++
        })
      Text("parent count:" + this.count)
      Child()
    }
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}

@Component
export struct Child {
  @Consume count: number

  build() {
    Column() {
      Button("child Add")
        .onClick(() => {
          this.count++
        })
      Text("child count:" + this.count)
    }
  }
}
```

### @Watch 

监听状态变化

```ArkTS
@Component
export struct Foo {
  // 当 count 改变时，会调用 onCountChange
  @State @Watch("onCountChange") count: number = 0

  build() {
    Column() {
      Button("Add")
        .onClick(() => {
          this.count++
        })
      Text(this.count.toString())
    }
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }

  onCountChange() {
    console.log("count 变化了", this.count)
  }
}
```

## 数据对象

### @Observed 

让对象内部字段也可观察

```ArkTS
@Observed
class User {
  name: string = ""
  age: number = 0
}
```

添加 @Observed，User 对象内部字段变化 → 自动触发 UI 更新。<br>
不加 @Observed，User 对象内部字段变化 → UI 不会刷新。ArkUI 默认只追踪变量引用变化，不会追踪对象内部字段。

如果是 API 11+，推荐使用 `@ObservedV2`。

### @ObjectLink 

用于绑定 @Observed 对象

@Link 不会监听对象内部字段变化，所以 @Link 大多只用于基本类型，对象类型必须用 @ObjectLink 才能保证响应式同步。

## UI 构建

### @Builder @BuilderParam

@Builder 声明一个可复用的 UI 构建函数。需要注意每个组件（用 @Component struct 定义）都有固定签名的 build 方法，这个 build 方法不是普通函数，它是组件生命周期的一部分，框架内部会自动当作 Builder 来执行，不需要再加 @Builder。

@BuilderParam 把一个 @Builder UI 构建函数作为参数传给组件，它允许父组件决定子组件内部某一块 UI 怎么画。

```ArkTS
@Component
export struct Parent {
  build() {
    Column() {
      this.buildItem("haha")
      Child({
        itemBuilder: this.buildItem
      })
    }
    .backgroundColor(Color.White)
    .height("100%")
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }

  @Builder
  buildItem(text: string) {
    Text(text)
  }
}

@Component
export struct Child {
  @BuilderParam itemBuilder: (text: string) => void

  build() {
    Column() {
      this.itemBuilder("Hello")
    }
  }
}
```

## 组件相关

### @Component

声明一个结构体为 UI 组件，可以被其他组件调用。

### @Entry 

声明应用 UI 入口组件，应用启动后：

```txt
Application start
      ↓
加载 EntryAbility
      ↓
渲染 @Entry 标记的组件
```

### @Reusable 

可复用组件，告诉框架：这个组件可以被复用，而不是每次都重新创建。主要目的是减少组件创建销毁，提高列表等场景的性能。常用于列表滚动的 item 组件

```ArkTS
@Builder
List() {
  LazyForEach(this.data, (item: string) => {
    ListItem() {
      ItemView({ text: item })
    }
  })
}

@Component
@Reusable
struct ItemView {
  @Prop text: string

  build() {
    Text(this.text)
  }
}
```

### @Preview 

在 DevEco Studio Preview 面板中显示预览

## LocalStorage

LocalStorage 是 ArkTS 为构建页面级别状态变量提供存储的内存内“数据库”。在页面中创建实例，组件中使用 @LocalStorageLink 和 @LocalStorageProp 装饰器修饰对应的状态变量，绑定对应的组件使用比状态属性更灵活。

* 应用程序可以创建多个 LocalStorage 实例，LocalStorage 实例可以在页面内共享，也可以通过 GetShared 接口，实现跨页面、UIAbility 实例内共享。
* 组件树的根节点，即被 @Entry 装饰的 @Component，可以被分配一个 LocalStorage 实例，此组件的所有子组件实例将自动获得对该 LocalStorage 实例的访问权限。
* 被 @Component 装饰的组件最多可以访问一个 LocalStorage 实例和 AppStorage，未被 @Entry 装饰的组件不可被独立分配 LocalStorage 实例，只能接受父组件通过 @Entry 传递来的 LocalStorage 实例。一个 LocalStorage 实例在组件树上可以被分配给多个组件。
* LocalStorage 中的所有属性都是可变的。

应用程序决定 LocalStorage 对象的生命周期。当应用释放最后一个指向 LocalStorage 的引用时，比如销毁最后一个自定义组件，LocalStorage 将被 JS Engine 垃圾回收。

### @LocalStorageLink

用来把变量和 LocalStorage 双向绑定。常用于轻量级、跨页面或组件间共享的状态，例如主题色、用户设置。key 必须唯一。
* 变量改变时自动写入本地存储
* 页面初始化时自动从本地存储读取值
* 简化了手动调用 LocalStorage 的过程

```ArkTS
@Component
export struct ThemeSwitcher {
  // 绑定到 LocalStorage 的 theme 字段，默认是 "light"
  @LocalStorageLink("theme") theme: string = "light"

  build() {
    Column() {
      Text("当前主题：" + this.theme)
      Button("切换主题")
        .onClick(() => {
          this.theme = this.theme === "light" ? "dark" : "light"
        })
    }
  }
}
```

### @LocalStorageProp

@LocalStorageProp 装饰的变量和与 LocalStorage 中给定属性建立单向同步关系。

## AppStorage

AppStorage 是在应用启动的时候会被创建的单例。它的目的是为了提供应用状态数据的中心存储，这些状态数据在应用级别都是可访问的。AppStorage 将在应用运行过程保留其属性。属性通过唯一的键字符串值访问。<br>
AppStorage 可以和 UI 组件同步，且可以在应用业务逻辑中被访问。<br>
AppStorage 支持应用的主线程内多个 UIAbility 实例间的状态共享。

在各个页面组件中使用 @StorageProp 和 @StorageLink 装饰器修饰对应的状态变量。

### @StorageProp

### @StorageLink

## PersistentStorage

PersistentStorage 将选定的 AppStorage 属性保留在设备磁盘上。应用程序通过 API，以决定哪些 AppStorage 属性应借助 PersistentStorage 持久化。UI 和业务逻辑不直接访问 PersistentStorage 中的属性，所有属性访问都是对 AppStorage 的访问，AppStorage 中的更改会自动同步到 PersistentStorage。

PersistentStorage 和 AppStorage 中的属性建立双向同步。应用开发通常通过 AppStorage 访问 PersistentStorage，另外还有一些接口可以用于管理持久化属性，但是业务逻辑始终是通过 AppStorage 获取和设置属性的。
