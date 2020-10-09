---
layout: post
title: RocketMQ填坑指南
date: 2020-7-5
categories: blog
tags: [架构,MQ,RocketMQ]
description: 天下无难事，只要肯放弃。
---

**RocketMQ集群坑**

RocketMQ集群分为多种模式，比如主从，双主，双主从等；其中有个“特性”，RocketMQ并不提供主从切换的机制，意思就是主永远是主，而从也永远是从，当主挂掉后，从存在的意义就是能够继续支持消息的消费，而这时候写操作是不会被响应的；另一个需要注意的点就是，硬盘空间对于RocketMQ特别重要，除了有个硬盘空间的检查外，当硬盘空间不足时，RocketMQ的Master仍会继续响应写入的请求，而客户端也只是发送出错而已，这个Master并不会从Namesrv中剔除。

**RocketMQ重试机制坑**

当消息消费失败后，RocketMQ提供自动重试机制，而重试次数默认是16次，而重试的时间窗口模式是1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h，然而重试并不是从1s开始，而是从第三个窗口10s开始，且重试的前提是客户端空闲。

**RocketMQ Consumer Group 坑**

当Consumer监听不同的Topic的时候，同时也要使用不同的Consumer Group，否则当使用相同的Consumer Group监听不同Topic的时候，会出现Topic部分queue没有监听的情况，也就是有一部分消息永远不会被人消费

**RocketMQ Queue数量坑**

一个Topic queue的数量决定了最多能支持Consumer的数量 ，比如你创建的Topic的时候只指定了4个queue，则当启动第五个Consumer时，是没有任何作用的

**RocketMQ Consumer Docker部署坑**

当Consumer使用Docker部署的时候会出现相同的instance name的情况，这是因为RocketMQ client模式使用的IP+进程号的方式来生成instance name, 而在docker里取到的IP竟然是docker0（比如172.12.0.1），进程号在docker容器内则都是相同的，而从出现了负载均衡的错误，导致部分queue无法消费；解决方法是手动设置系统变量“rocketmq.client.name”，使用一个唯一性的值如snowflake，这样就会覆盖进程号部分，而从获得不同的instance name

**当使用主从同步且同步刷盘坑**

当使用主从同步且同步刷盘模式时，当发送频率高的话，容易出现"broker busy, xxx"错误，此时需要优化broker配置，增加如下配置

```
#根据服务器CPU配置
sendMessageThreadPoolNums=16
useReentrantLockWhenPutMessage=true
#default is 200ms, change to 1000ms
waitTimeMillsInSendQueue=1000
```

