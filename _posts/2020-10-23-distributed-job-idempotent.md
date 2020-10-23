---
layout: post
title: 一次分布式任务幂等处理的探索
date: 2020-10-14
categories: blog
tags: [架构,分布式任务]
description: 天下无难事，只要肯放弃。
---

当设计一个分布式任务系统的时候，往往会遇到任务幂等的处理，比如同一个任务一天只能执行一次，或者同一个任务实例只能执行一次，否则会造成数据混乱；而这时需要怎么做呢，有几种做法呢？

**分析**

假设任务中有以下伪代码
```
//业务处理, 获取结果数据
data = business.do();
//将业务数据插入至数据库
//开启事务
Transaction.start()
dao.insertOrUpdate(data)
//提交事务
Transaction.commit()
```

假设以上任务一天只能执行一次的话，那如果由于重试或者消息等机制，导致被触发多次，那么只能数据库中就会存在脏数据，甚至造成一定的损失，所以要进行幂等处理，即让其一天只能执行一次

这时可能想到是不是可以借助中间件如Redis进行处理呢？于是伪代码修改如下：

```
//判断是否执行过
flag = redisClient.get('20200101')
if (flag == null) {
    //业务处理, 获取结果数据
    data = business.do();
    //将业务数据插入至数据库
    //开启事务
    Transaction.start()
    dao.insertOrUpdate(data)
    //提交事务
    Transaction.commit()
    redisClient.set('20200101', '1')
}
```
这样是否就可以了呢？显然是不行的，因为在分布式场景下，可能有两个以上节点并发执行，当它们并发执行到“flag = redisClient.get('20200101')”，得到的flag可能都是null，这时就仍会出现多次执行业务逻辑的问题。

试想如果是单机多线程，我们会选择如何处理呢？立刻想到了锁进行同步处理，那在分布式环境中，则显然可以使用分布式锁。

**分布式锁方案**

伪代码修改如下：

```
distributedLock.lock()
//判断是否执行过
flag = redisClient.get('20200101')
if (flag == null) {
    //业务处理, 获取结果数据
    data = business.do();
    //将业务数据插入至数据库
    //开启事务
    Transaction.start()
    dao.insertOrUpdate(data)
    //提交事务
    Transaction.commit()
}
distributedLock.release()
```

这样貌似就解决了我们的问题，至于分布式锁可以使用Zookeeper等CP模型的中间件进行实现或者数据库悲观锁等，但是不是给我们带来了新的问题呢，比如系统变得复杂了，增加了中间件的维护成本，而且分布式事务会使我们的应用性能大幅度下降。

那有没有更加简单的方式呢？让我们简单的利用数据库事务的特性

**本地事务表方式**

首先我们在业务的数据库中创建一张表，比如叫job_record，其中有两个字段job_name, 和date，且二者为联合主键。

此时伪代码修改如下：
```
//判断是否执行过
flag = jobRecordDao.select('testJob', '20200101')
if (flag == null) {
    //业务处理, 获取结果数据
    data = business.do();
    //将业务数据插入至数据库
    //开启事务
    Transaction.start()
    jobRecordDao.insert('testJob', '20200101')
    dao.insertOrUpdate(data)
    //提交事务
    Transaction.commit()
}
```

此时的flag判断只是在非高并发场景下，减少对数据库的尝试提交事务以及业务逻辑处理操作，而当出现并发，flag判断不起作用的情况下，insert job_record会触发数据库的主键重复检测，抛出异常，而job_record的插入操作和业务数据的提交又在一个数据库事务里，则会发生回滚操作，这样就保证了同一个job即使在高并发场景下一天仍然只能执行一次的目的；好处也显而易见，没有增加任何中间件，也没有使性能出现严重的下降，大部分场景只是增加了一次select查询而已。

至此完成了这次分布式任务的幂等探索，成果可人！
