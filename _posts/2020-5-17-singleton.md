---
layout: post
title: 设计模式之单例
date: 2020-5-17
categories: blog
tags: [架构,设计模式]
description: 天下无难事，只要肯放弃。
---

单例模式是一种简单且最常用的设计模式，有人可能会说我也没有用过单例模式，那如果你是一枚可爱的Java程序猿/媛，那你肯定用过Spring吧，而Spring Bean默认的构建模式都是单例的，而Spring为什么会这么做呢，显然这和单例的优势有关。

#### 什么是单例模式？

属于创建类型的一种常用的软件设计模式。通过单例模式的方法创建的类在当前进程中只有一个实例（根据需要，也有可能一个线程中属于单例，如：仅线程上下文内使用同一个实例）（引用自百度百科）

#### 为什么使用单例模式

在应用或者服务运行过程中，有些类的实例是没有必要每次都新建的，而减少新建实例带来的性能损耗也成为了使用单例带来的巨大好处；比如Java在新建一个实例时则需要分配内存，给成员变量赋值等操作，当这种操作很频繁，不但加大了系统的开销，也增加了触发GC的频率。这就如同你新找了一个对象，又要从请吃饭看电影，拉手，亲亲……开始，伤钱伤身体。因此诸如Spring也会大量使用单例，来节省资源减少系统开销。

#### 如何实现单例

实现单例模式通过分为懒汉和饿汉两种方式；其中懒汉(lazy)即是在使用的时候才创建实例，不使用则永远不会创建实例，饿汉则是应用启动时已经创建实例，不管是否使用都会创建。

**饿汉模式**

简单饿汉模式的实现如下，代码简单明了，且没有懒汉模式的线程安全问题，如不考虑在不使用实例的情况下占用的那一丢丢内存，**强烈建议如此使用**

```java
public class Singleton {

    /**
     * 声明为static成员变量
     */
    private static Singleton INSTANCE = new Singleton();

    private Singleton() {
        //私有构造方法
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }

}
```

**懒汉模式**

饿汉模式就更加值得研究，它虽然在不使用实例的情况下节省了系统开销（也只是一丢丢而已），但是同时带来了一些有趣的问题。

```java
public class Singleton {

    private static Singleton INSTANCE = null;

    private Singleton() {
        //私有构造方法
    }

    public static Singleton getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }

}
```
以上是个非常简单的饿汉模式的实现，有经验的猿/媛也会发现，这个方法存在线程安全问题，当多线程并发调用getInstance方法时，会造成创建多次实例的情况，违背了单例模式设计的初衷，于是我们将其更改如下：

```java
public class Singleton {

    private static Singleton INSTANCE = null;

    private Singleton() {
        //私有构造方法
    }

    public static synchronized Singleton getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }

}
```
我们在getIntance方法上加了synchronized，这样则快速解决了线程安全问题，然而追求完美的我们知道这样做在高并发情况下会发生阻塞，它还不够完美（没错Coder就是如此的吹毛求疵），我们应该缩小synchronized的范围，于是产生了如下代码：

```java
public class Singleton {

    private volatile static Singleton INSTANCE = null;

    private Singleton() {
        //私有构造方法
    }

    public static Singleton getInstance() {
        if (INSTANCE == null) {
            synchronized (Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```
这次我们缩小了synchronized的范围，且把INSTANCE使用了volatile修饰，是使用双重check避免了在INSTANCE不为null时还在等待锁的问题，但是代码看起来有点难懂，是不是可能还有更加完美的方式呢，看如下代码：

```
public class Singleton {

    private Singleton() {
        //私有构造方法
    }

    public static Singleton getInstance() {
        return SingletonLazyHolder.INSTANCE;
    }

    private static class SingletonLazyHolder {
        private static Singleton INSTANCE = new Singleton();
    }

}
```
这次我们使用了静态内部类来实现，根据Java classloader原理，内部类只有在使用时才会被初始化，利用这一点可以实现线程安全的懒汉模式。

**另一种模式实现**

以上方式仍然存在问题（毕竟我们是吹毛求疵的Coder），因为私有发放并不安全，我们仍然可以通过反射使用AccessibleObject.setAccessible来创建的实例，同样序列化readObject也可以做到创建新的实例，于是有大神发明了如下实现方式：

```java
public class Singleton {

    private Singleton() {
        //私有构造方法
    }

    public static Singleton getInstance() {
        return SingletonEnum.INSTANCE.getInstance();
    }

    static enum SingletonEnum {
        INSTANCE;
        private Singleton singleton = null;
        private SingletonEnum() {
            singleton = new Singleton();
        }

        public Singleton getInstance() {
            return singleton;
        }
    }

}
```
利用单元素的枚举来实现单例模式，被称为“实现Singleton的最佳方法”，但是否需要如此繁琐复杂，需要Coder们自己考虑了。


实现单例模式并不是我们的目的之一，通过学习不同的实现方式让我们对其他知识也有了思考，至于选择哪种实现方式还是具体问题具体分析；如果不是很处女座，饿汉模式是你不悔的选择！

