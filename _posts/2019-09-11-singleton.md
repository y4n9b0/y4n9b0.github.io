---
layout: post
title:  单例模式
date:   2019-09-11 09:30:00 +0800
categories: Java
tags: singleton
published: true
---

* content
{:toc}

## 一、饿汉模式

```java
public class Singleton {  
    private static Singleton instance = new Singleton();

    private Singleton (){}  

    public static Singleton getInstance() {  
        return instance;  
    }  
}

// 饿汉模式变种，将 instance 放在 static 代码块
// 静态代码块必须放在 instance 定义之后，否则静态代码块赋值无效（可以正常编译）
// kotlin 的 object 实现原理便是这种
public class Singleton {
    private static Singleton instance = null;

    static {
        instance = new Singleton();
    }

    private Singleton (){}

    public static Singleton getInstance() {
        return instance;
    }
}

```

类加载时就完成了初始化，线程安全，但不是 lazy loading

## 二、懒汉模式（线程不安全）

```java
public class Singleton {  
    private static Singleton instance;

    private Singleton (){}  
  
    public static Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
    }  
}
```

lazy loading，解决了饿汉模式资源浪费的缺点，但是多线程下不安全

## 三、懒汉模式（线程安全）

```java
public class Singleton {  
    private static Singleton instance;

    private Singleton (){}  

    public static synchronized Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
    }  
}  
```

lazy loading，线程安全，但是同步带来额外开销会导致效率降低（绝大部分情况下不会并发）

## 四、双重校验锁

```java
public class Singleton {
    private volatile static Singleton instance;

    private Singleton (){}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

双重校验锁：Double Check Lock，缩写 DCL，懒汉模式（线程安全）的升级版，同样 lazy loading，线程安全。

getInstance 方法中对 instance 进行了两次判空，第二次判空很好理解，在 instance 等于 null 的情况下才创建实例。  
为什么需要第一次的判空呢？是为了不必要的同步。对比懒汉模式（线程安全），getInstance 方法其实只需要在第一次调用的时候加锁，第一次调用后 instance 已经实例化，不再需要加锁；于是进化成双重校验锁的模式，第一次调用会进入到 synchronized 代码块，第一次调用后 instance 已经实例化，后面调用可以通过判空使其不再进入 synchronized 代码块（减少同步开销），直接返回 instance。

instance 必须配合关键字 volatile 保证可见性，具体原因参考 [有序性](https://y4n9b0.github.io/2019/09/10/Java-concurrent/#%E6%9C%89%E5%BA%8F%E6%80%A7){:target="_blank"}。

**Note** volatile 在 JDK 1.5 之前只保证可见行，并不禁止重排序，所以理论上 DCL 在 JDK 1.5 之前可能会失效。
参考 [JSR 133](https://jcp.org/en/jsr/detail?id=133)。

## 五、静态内部类

```java
public class Singleton {  
    private Singleton (){}

    public static final Singleton getInstance() {  
        return Holder.INSTANCE;  
    }

    private static class Holder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
}
```

lazy loading，线程安全。

第一次加载 Singleton 类时并不会初始化 INSTANCE，只有在第一次调用 getInstance 方法时虚拟机加载 Holder 并初始化 INSTANCE。

<br>

到目前位置，我们已经知道了五种单例模式，但是这五种单例模式都有两个问题：**反射**、**序列化**。

### 反射

虽然我们将默认构造函数设置为 private，但是依然可以通过反射来调用该私有构造函数，从而生成一个新的实例。

### 序列化

将一个单例的实例对象写到磁盘再读回来，从而获得了一个新的实例（读回来的时候会调用 readObject，任何一个readObject方法，不管是显式的还是默认的，它都会返回一个新建的实例，这个新建的实例不同于该类初始化时创建的实例）。

反序列化操作提供了 readResolve 方法，这个方法可以让开发人员控制对象的反序列化。在上面的五个单例中如果要杜绝单例对象被反序列化时重新生成对象，就必须加入如下方法：

```java
private Object readResolve() throws ObjectStreamException{
    return instance;
}
```

## 六、枚举

```java
public enum Singleton {  
    INSTANCE;

    public void whateverMethod() {  
    }  
}
```

枚举实例的创建默认是线程安全的，在任何情况下都是单例，而且没有反射、序列化问题。枚举的反射会抛 IllegalArgumentException 异常（参考 Constructor 类的 newInstance 方法）；枚举的序列化读取回来依然是同一个实例。

强烈推荐该方式，Joshua Bloch 在《effective java》中亦说到“单元素的枚举类型已经成为实现 Singleton 的最佳方法“。

**Note** 枚举也是 JDK 1.5 才有的。
