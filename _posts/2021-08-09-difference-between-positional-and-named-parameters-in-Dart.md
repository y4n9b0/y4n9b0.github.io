---
layout: post
title: Dart 可选位置参数和可选命名参数的区别
date: 2021-08-09 17:00:00 +0800
categories: dart
tags: dart
published: true
---

* content
{:toc}

## 可选位置参数

在 Dart 里，将参数包裹在方括号中，可以使其变成可选位置参数(optional positional parameters)：

```dart
  String getHttpUrl(String host, String path, [int port = 80, int numRetries = 3]) {
    // ...
  }

  getHttpUrl('example.com', '/index.html');
  getHttpUrl('example.com', '/index.html', 8080);
  getHttpUrl('example.com', '/index.html', 8080, 5);
```

## 可选命名参数

将参数包裹在大括号中，可以使其变成可选命名参数(optional named parameters)：

```dart
  String getHttpUrl(String host, String path, {int port = 80, int numRetries = 3}) {
    // ...
  }

  getHttpUrl('example.com', '/index.html');
  getHttpUrl('example.com', '/index.html', port = 8080);
  getHttpUrl('example.com', '/index.html', numRetries = 5);
  getHttpUrl('example.com', '/index.html', port = 8080, numRetries = 5);
  getHttpUrl('example.com', '/index.html', numRetries = 5, port = 8080);
```

## 区别

可选位置参数始终是位置参数，它使用参数位置顺序 position 来标记对应的是哪个参数，后一个参数一定是依赖前一个参数的存在（如果前一个参数不存在，则我们期望的后一个参数就补位成了前一个参数）。一旦需要设置某个可选位置参数时，为了保证 position 的正确，在其之前的可选位置参数都不能省略。
而可选命名参数通过 name 来标记对应的是哪个参数，不再使用参数顺序的 position，不需要依赖其他可选命名参数的顺序，任何两个命名参数之间都是独立的。

以上面的 `getHttpUrl` 为例，假设我们要设置重试的次数为 5:<br>
定义为可选位置参数时，我们应该调用 `getHttpUrl('example.com', '/index.html', 8080, 5)` 而不是 `getHttpUrl('example.com', '/index.html', 5)`，后者5所在的位置已经对应成了port，这与我们的期望是不符的。<br>
定义为可选命名参数时，我们只需要这样调用 `getHttpUrl('example.com', '/index.html', numRetries = 5)` 即可。

现在我们站在更高的角度回头来看这两种参数设计的本质，函数传参需要解决的一个核心问题就是实参和形参的一一对应。位置参数通过位置顺序来确定对应关系，而命名参数通过参数名来确定对应关系。前者位置顺序是隐式的，后者需要显式地标记名字。所以本质区别其实就是函数实参和形参对应关系的实现手段不一致。

相对于可选位置参数，可选命名参数的应用更广泛。但并不是绝对的，在可选参数具有层级依赖的时候使用可选位置参数会更加方便。

假如我们的日常剁手地址是四川省成都市高新区天府五街C区13栋7楼，实际上C区的快递基本都是统一在食堂外面的空地上，所以只需要填到C区就可以了。
但有时候老板大气，给发的顺丰，提供上门派送服务，这时候我们就需要填到具体的楼层。
这种场景，使用可选位置参数显然是一个更优的选择，因为 floor 是依赖于 building 的（不带楼栋的楼层信息是没有意义的）：

```dart
  String addr(String city, String district, String street, String block, [String? building, String? floor]){
    // ...
  }

  String express4tong1da() => addr("成都市", "高新区", "天府五街", "C区");
  String expressShunFeng() => addr("成都市", "高新区", "天府五街", "C区", "13栋", "7楼");
```

但是，如果函数参数过多，即便参数有层级依赖，还是建议使用可选命名参数，显式的参数 name 会使得代码更具可读性。

## 注意事项

* 不可以在一个函数里同时使用可选位置参数和可选命名参数。
* 无论可选位置参数还是可选命名参数永远都是放在方法参数列表的最后。
* 可选命名参数之间可以以任何顺序赋值。

<!-- https://stackoverflow.com/a/13264231/7368406 -->
<!-- https://dart.dev/guides/language/language-tour#parameters -->
<!-- https://dart.dev/codelabs/dart-cheatsheet#optional-positional-parameters -->
<!-- https://dart.dev/codelabs/dart-cheatsheet#optional-named-parameters -->
