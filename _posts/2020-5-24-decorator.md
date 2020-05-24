---
layout: post
title: 设计模式之装饰者
date: 2020-5-24
categories: blog
tags: [架构,设计模式]
description: 天下无难事，只要肯放弃。
---

#### 定义

装饰者模式又名包装(Wrapper)模式。装饰者模式以对客户端透明的方式扩展对象的功能，是继承关系的一个替代方案。

装饰者模式动态地将责任附加到对象身上。若要扩展功能，装饰者提供了比继承更有弹性的替代方案。

#### 如何使用

还记得在大学时候完了一个游戏叫《海盗王》，玩家可以在自己的武器等装备上镶嵌各种宝石来增加属性，如增加攻击力，增加防御，血量等等，这就是一个典型的可以应用装饰者的场景。

首先应该有个装备的接口

```java
public interface Equipment {
    /**
     * 攻击力
     * @return
     */
    int attack();

    /**
     * 防御力
     * @return
     */
    int defense();

    /**
     * 生命值
     * @return
     */
    int lifeValue();
}
```

然后我拥有一把大宝剑，它拥有超凡的攻击力1000，佩戴时增加生命值200

```java
public class Sword implements Equipment {

    @Override
    public int attack() {
        return 1000;
    }

    @Override
    public int defense() {
        return 0;
    }

    @Override
    public int lifeValue() {
        return 200;
    }
}
```
像装备上镶嵌的宝石，分别为红色宝石（增加攻击力50/颗），蓝色宝石（增加防御力50/颗），绿色宝石（增加生命值100/颗）

**红色宝石**
```java
public class RedGem implements Equipment {

    private Equipment equipment;

    public RedGem(Equipment equipment) {
        this.equipment = equipment;
    }

    @Override
    public int attack() {
        return this.equipment.attack() + 50;
    }

    @Override
    public int defense() {
        return this.equipment.defense();
    }

    @Override
    public int lifeValue() {
        return this.equipment.lifeValue();
    }
}
```

**蓝色宝石**

```java
public class BlueGem implements Equipment {

    private Equipment equipment;

    public BlueGem(Equipment equipment) {
        this.equipment = equipment;
    }

    @Override
    public int attack() {
        return this.equipment.attack();
    }

    @Override
    public int defense() {
        return this.equipment.defense() + 50;
    }

    @Override
    public int lifeValue() {
        return this.equipment.lifeValue();
    }
}
```

**绿色宝石**

```java
public class GreenGem implements Equipment {

    private Equipment equipment;

    public GreenGem(Equipment equipment) {
        this.equipment = equipment;
    }

    @Override
    public int attack() {
        return this.equipment.attack();
    }

    @Override
    public int defense() {
        return this.equipment.defense();
    }

    @Override
    public int lifeValue() {
        return this.equipment.lifeValue() + 100;
    }
}
```

现在我想知道在大宝剑上镶嵌了2颗红宝石1颗蓝宝石2颗绿宝石后，我能得到什么样的属性呢？

```java
Equipment equipment = new RedGem(new RedGem(new BlueGem(new GreenGem(new GreenGem(new Sword)))));
equipment.attack(); //攻击力总计
equipment.defense(); //防御力总计
equipment.lifeValue(); //生命值总计
```

#### 总结
至此我们完成了装饰者模式的例子，可能有人会问这样做看起来code也不是也简单啊，但是想想“开闭原则”，如果这时候又来个装备比如鞋子，或者有加入了一个黑色的宝石，是不是只需要进行简单的扩展即可，所以这才是使用装饰者模式的目的！