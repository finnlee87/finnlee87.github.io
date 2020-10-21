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

##### 1.引入已有的Starter，如

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

##### 2.在自定义的新的Starter中对支持覆盖的Bean进行覆盖并调整AutoConfiuration的顺序

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

##### 3.配置spring.factories

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.example.MyAutoConfiguration
```

**扩展性不好的Starter**

对于不是很“标准”的Starter，一般处理方式有两种，第一种就是引入其依赖的jar包，然后不引入其Starter，模仿其Starter进行一个实现；第二种方式是因为其没有依赖jar包，或者说Starter就仅仅是一个Jar或者几个类而已，这时则需要排除到其AutoConfiguration的加载。

第一种方式不做过多介绍，属于VC大法的实践；第二种方式操作如下：

##### 1.引入已有的Starter

##### 2.使用AutoConfigurationImportFilter排除掉已有的AutoConfiguration

在spring.factories中增加
```properties
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=com.example.MyImportFilter
```

com.example.MyImportFilter code:
```java
public class MyImportFilter implements AutoConfigurationImportFilter {
  public static final String EXCLUDE_AUTOCONFIGURATION = "com.example.DemoAutoConfiguration";
  
  @Override
  public boolean[] match(String[] autoConfigurationClasses, AutoConfigurationMetadata autoConfigurationMetadata) {
    int length = autoConfigurationClasses.lenght
    boolean[] results = new boolean[length];
    
    for (int i = 0; i < length; i++) {
      results[i] = !EXCLUDE_AUTOCONFIGURATION.equals(autoConfigurationClasses[i]);
    }
    
    return results;
  }
  
}
```

##### 3.编写自定义的AutoConfiguration

```java
public class MyAutoConfiguration {
  @Bean
  public DemoBean demoBean() {
    DemoBean demoBean = new DemoBean();
    demoBean.setName("My demo bean")
    return demoBean;
  }
}
```

##### 4.配置spring.factories
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.example.MyAutoConfiguration
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=com.example.MyImportFilter
```
