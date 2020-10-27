---
layout: post
title: 为什么选择RocketMQ，而不是Kafka
date: 2020-10-26
categories: blog
tags: [架构,MQ,RocketMQ,Kafka]
description: 天下无难事，只要肯放弃。
---

一个复杂的分布式系统，必然会考虑使用MQ来解决通讯解耦等问题，市面上开源的MQ越来越多，也让选择变得越来越难，比如Kafka，RabbitMQ，RocketMQ，让人看得眼花缭乱，好像每个MQ都很强大，究竟使用哪个真是无从下手；而在一次技术选型中，我们落地了RocketMQ，而不是Kafka或者其他，以下也对这次选择做一次总结，讲述以下RocketMQ和Kafka的比较。
