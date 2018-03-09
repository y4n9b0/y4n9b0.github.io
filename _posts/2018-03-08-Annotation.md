---
layout: post
title:  Annotation
date:   2018-03-08 10:30:00 +0800
categories: Java
tags: annotation
published: false
---

* content
{:toc}


# Prologue
学习Retrofit这些框架，源码涉及到大量注解。


# Terms and concepts
**注解**
>```
public @interface TestAnnotation {
}
```

**元注解**：  
用于给注解进行注解,有 @Retention、@Documented、@Target、@Inherited、@Repeatable 5 种。  

* @Retention
  解释说明了这个注解的的存活时间。它的取值如下：
  * RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
  * RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
  * RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

  >```
  @Retention(RetentionPolicy.RUNTIME)
  public @interface TestAnnotation {
  }
  ```
* @Documented
  将注解中的元素包含到 Javadoc 中去。

* @Target
  指定注解运用的地方，限定运用的场景。取值如下：
  * ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
  * ElementType.CONSTRUCTOR 可以给构造方法进行注解
  * ElementType.FIELD 可以给属性进行注解
  * ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
  * ElementType.METHOD 可以给方法进行注解
  * ElementType.PACKAGE 可以给一个包进行注解
  * ElementType.PARAMETER 可以给一个方法内的参数进行注解
  * ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举


* @Inherited
  一个超类被 @Inherited 注解过的注解进行注解，如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。
  >```
  @Inherited
  @Retention(RetentionPolicy.RUNTIME)
  @interface Test {}
  >
  @Test
  public class A {}
  >
  public class B extends A {}
  ```

  注解 Test 被 @Inherited 修饰，之后类 A 被 Test 注解，类 B 继承 A,类 B 也拥有 Test 这个注解。

* @Repeatable
  是 Java 1.8 才加进来的，所以算是一个新的特性。表示注解可以重复应用取值。
  例如：一个人他既是程序员又是产品经理,同时他还是个画家。
  >```
  @interface Persons {
      Person[]  value();
  }
  >
  @Repeatable(Persons.class)
  @interface Person{
      String role default "";
  }
  >
  @Person(role="artist")
  @Person(role="coder")
  @Person(role="PM")
  public class SuperMan{
  >
  }
  ```

  @Repeatable 注解了 Person，而 @Repeatable 后面括号中的类相当于一个容器注解。  
  **容器注解** 就是用来存放其它注解的地方。它本身也是一个注解，内部必须要有一个 value 的属性，属性类型是一个被 @Repeatable 注解过的注解数组。

**注解的属性**  
  注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。
  >```
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface TestAnnotation {
  >
    int id();
    String msg();
  }
  ```

  上面代码定义了 TestAnnotation 这个注解中拥有 id 和 msg 两个属性。在使用的时候，应该给它们进行赋值。  
  赋值的方式是在注解的括号内以 value=”” 形式，多个属性之前用英文逗号隔开。
  >```
  @TestAnnotation(id=3,msg="hello annotation")
  public class Test {}
  ```

  注解中属性可以有默认值，默认值需要用 default 关键值指定，使用时可以不用在括号里赋值。
  >```
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface TestAnnotation {
  >
    public int id() default -1;
    public String msg() default "Hi";
  }
  >
  @TestAnnotation()
  public class Test {}
  ```

  另外，如果一个注解内仅仅只有一个名字为 value 的属性时，应用这个注解时可以直接将属性值填写到括号内。
  >```
  public @interface Check {
    String value();
  }
  >
  @Check("hi")
  int a;
  ```

  最后，还有一种情况是一个注解没有任何属性，在应用这个注解的时候，括号都可以省略。
  >```
  public @interface Perform {}
  >
  @Perform
  public void testMethod(){}
  ```

  **NOTE**: 在注解中定义属性时它的类型必须是 8 种基本数据类型外加 String、Enums、类、接口、注解及它们的数组。


**Java预置的注解**

* @Deprecated
  这个元素是用来标记过时的元素，想必大家在日常开发中经常碰到。编译器在编译阶段遇到这个注解时会发出提醒警告，告诉开发者正在调用一个过时的元素比如过时的方法、过时的类、过时的成员变量。

* @Override
  子类复写父类中被 @Override 修饰的方法。

* @SuppressWarnings
  阻止警告的意思。之前说过调用被 @Deprecated 注解的方法后，编译器会警告提醒，而有时候开发者会忽略这种警告，他们可以在调用的地方通过 @SuppressWarnings 达到目的。

* @SafeVarargs

* @FunctionalInterface
  函数式接口注解，这个是 Java 1.8 版本引入的新特性。
  函数式接口 (Functional Interface) 就是一个具有一个方法的普通接口，函数式接口可以很容易转换为 Lambda 表达式。
  >```
  @FunctionalInterface
  public interface Runnable {
      /**
       * When an object implementing interface <code>Runnable</code> is used
       * to create a thread, starting the thread causes the object's
       * <code>run</code> method to be called in that separately executing
       * thread.
       * <p>
       * The general contract of the method <code>run</code> is that it may
       * take any action whatsoever.
       *
       * @see     java.lang.Thread#run()
       */
      public abstract void run();
  }
  ```

# 注解的提取

https://docs.oracle.com/javase/tutorial/java/annotations/  

To be continue…
