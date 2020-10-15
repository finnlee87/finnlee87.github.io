---
layout: post
title: 如何基于已有的SpringBoot Starter定制自己的Starter
date: 2020-10-14
categories: blog
tags: [架构,SpringBoot]
description: 天下无难事，只要肯放弃。
---

使用SpringBoot开发应用程序，在架构设计或封装代码时免不了要自定义自己的SpringBoot Starter，而有的时候并不会从零开始，则会基于第三方的Starter重新封装为满足技术需求的Starter，这时就必须了解如何基于已有的SpringBoot Starter定制自己的Starter，此篇将介绍其中做法。

**分类**

SpringBoot Starter的实现由于Coder水平的不同，一般会分为容易扩展的和不容易扩展的；容易扩展的Starter在XXXAutoConfiguration类中使用了大量的@ConditionOnXXX注解，反之则几乎没有使用。

**标准的扩展性好的Starter**

#### 1.引入已有的Starter，如

```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>thirdparty-starter</artifactId>
  <version>1.0.0</version>
</dependency>
```

其中包含AutoConfiguration

```java
public class DemoAutoConfiguration {
  @Bean
  @ConditionalOnMissingBean
  public DemoBean demoBean() {
    return new DemoBean();
  }
}
```

#### 2.在自定义的新的Starter中对支持覆盖的Bean进行覆盖并调整AutoConfiuration的顺序

```java
@AutoConfigureBefore(DemoAutoConfigration.class)
public class MyAutoConfiguration {
  @Bean
  public DemoBean demoBean() {
    DemoBean demoBean = new DemoBean();
    demoBean.setName("My demo bean")
    return demoBean;
  }
}
```

#### 3.配置spring.factories

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.example.MyAutoConfiguration
```

**扩展性不好的Starter**
