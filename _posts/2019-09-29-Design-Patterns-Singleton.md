---
layout: post
title: 单例模式(Singleton pattern)
tags: 设计模式   
---

### 为什么要使用单例模式
单例模式属于上篇说过的设计模式三大分类中的第一类——创建型模式。顾名思义，单例设计模式就是为了保证创建出来的对象实例只有一个。
- 通过控制创建对象的数量，节约系统资源开销。
- 有些场景下，不使用单例模式，会导致系统同一时刻出现多个状态缺乏同步，用户自然无法判断当前处于什么状态
- 全局数据共享


### 饿汉式
饿汉式
``` java
public class HungrySingleton {
  private HungrySingleton() {}  // 构造方法私有化，防止通过new创建对象实例

  private static HungrySingleton instance = new HungrySingleton();  //持有对象实例引用，static 修饰

  public static HungrySingleton getInstance() {     //返回对象实例
    return instance;
  }
}
```







