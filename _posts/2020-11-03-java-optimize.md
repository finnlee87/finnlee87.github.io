---
layout: post
title: Java代码性能优化小技巧
date: 2020-11-03
categories: blog
tags: [Java,性能优化]
description: 天下无难事，只要肯放弃。
---

简单总结，工作中遇到的问题，仅供参考，如出现任何问题，不接受任何投诉与反驳，哈哈。

**双层循环组合数据篇**

业务开发中经常会遇到两个List

```java
List<Demo1> demo1List;
List<Demo2> demo2List;
for (Demo1 demo1 : demo1List) {
    for (Demo2 demo2 : demo2List) {
        if (demo1.getName.equals(demo2.getName())) {
            demo1.setDemo2(demo2);
        }
    }
}
```
以上伪代码存在两个List，且Demo1 List需要将Demo2 List中的name相同的Demo2合并进来，业务上经常会有这样的操作，假设Demo1 List大小为m，Demo2 List大小为n，则这样时间符合度则是O(m*n)

当然也有人学会了Stream API会如下实现

```java
List<Demo1> demo1List;
List<Demo2> demo2List;
demo1List.forEach(demo1 -> {
    List<Demo2> demo2FilterList demo2List.stream().filter(demo2 -> demo2.getName().equals(demo1.getName())).collect(Collectors.toList());
    demo1.setDemo2(demo2FilterList.get(0));
})
```
以上的方式其实复杂度基本没有任何变化，即使计算开始了stream的parellel，只是利用的多线程优势，复杂度仍然没有进步

那……推荐的样子是什么呢？如下
```java
List<Demo1> demo1List;
List<Demo2> demo2List;
Map<String, Demo2> demo2Map = demo2List.stream.collect(Collectors.toMap(Demo2::getName, value -> value));
demo1List.forEach(demo1 -> {
    demo1.setDemo2(demo2Map.get(demo1.getName()));
})
```
以上代码现将Demo2 List转换为key是name、value是Demo2对象，这样在循环Demo1 List获取对应Demo2的时候则可以利用Map数据结构复杂度O(1)的特性，此段代码复杂度则变成O(n+m+m*1)，复杂度大大降低

**循环内调用IO篇**

review过大量的业务代码，时常会发现有在循环中调用IO的情况，包括不限于访问数据库，读写文件，甚至访问Redis等；IO操作是非常耗时的操作，对于大部分的业务系统，系统处理时间大部分都集中在IO处理上，所以应尽量避免此类操作，可采取批量读取出来，然后再使用上面的组合方法进行优化。

**集合应尽量初始化容量使用篇**

Java开发中会大量使用集合类，如Map，List等，而这些类大多都是有容量和自动扩容机制的，比如ArrayList的默认容量是10，HashMap的默认容量是16等，当容量达到上限则会自动扩容，扩容则会带来系统的开销，ArrayList则需要做数组拷贝，HashMap则需要rehash操作；所以在明确知道需要容量的情况下，可以在初始化阶段就指定其容量来减少大量的扩容开销，这是一个非常实用的优化手段。（注意：HashMap计算实际要使用的容量需要考虑load factor问题，所以推荐使用guava进行初始化操作）

待续…………
