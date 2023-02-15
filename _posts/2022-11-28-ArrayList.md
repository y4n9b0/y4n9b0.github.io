---
layout: post
title:  ArrayList
date:   2022-11-28 10:00:00 +0800
categories: publish
tags: Collection
published: false
---

* content
{:toc}



```java
public List<String> foo(int size) {
    ArrayList<String> list = new ArrayList<>(size);
    int mid = (size + 1) >> 1;
    for (int i = 0; i < mid; i++) {
        String item = "bar" + i;
        list.add(i, item);
        list.add(size - 1 - i, item);
    }
    return list;
}
```

```java
public List<String> foo(int size) {
    String[] array = new String[size];
    int mid = (size + 1) >> 1;
    for (int i = 0; i < mid; i++) {
        String item = "bar" + i;
        array[i] = item;
        array[size - 1 - i] = item;
    }
    return Arrays.asList(array);
}
```
