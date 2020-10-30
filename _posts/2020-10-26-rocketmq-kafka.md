---
layout: post
title: 为什么选择RocketMQ，而不是Kafka
date: 2020-10-26
categories: blog
tags: [架构,MQ,RocketMQ,Kafka]
description: 天下无难事，只要肯放弃。
---

一个复杂的分布式系统，必然会考虑使用MQ来解决通讯解耦等问题，市面上开源的MQ越来越多，也让选择变得越来越难，比如Kafka，RabbitMQ，RocketMQ，让人看得眼花缭乱，好像每个MQ都很强大，究竟使用哪个真是无从下手；而在一次技术选型中，我们落地了RocketMQ，而不是Kafka或者其他，以下也对这次选择做一次总结，讲述以下RocketMQ和Kafka的比较。

**性能**

在性能方面，Kafka是优于RocketMQ的，可以达到几十万的TPS，而RocketMQ很难超过10W TPS，但是这是有提前的，提前就是Kafka的Topic或者Partition不能过多（最好不要超过64个），这是因为Kafka之所以可以高性能是因为采用了顺序写，但如果一旦Topic或者Partition变多，则变成不断的写多个文件相当于随机写，所以性能开始大幅度下降；而RocketMQ则几乎没有这个问题，及时在Topic很多的（官方说可以支持5W），也不会出现明显的性能下降，所以在需要使用很多Topic的场景下，可以选择RocketMQ，且上万的TPS也是不错的性能表现。

