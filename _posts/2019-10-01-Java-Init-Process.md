---
layout: post
title: Java类静态代码块初始化顺序
tags: java
---
<!-- TOC -->

- [代码块和构造方法的执行顺序](#代码块和构造方法的执行顺序)
    - [测试代码](#测试代码)
    - [执行结果：](#执行结果)
- [总结](#总结)
    - [一个类中的初始化顺序](#一个类中的初始化顺序)
    - [两个具有继承关系类的初始化顺序](#两个具有继承关系类的初始化顺序)

<!-- /TOC -->


### 代码块和构造方法的执行顺序
我相信大家多做过一道笔试题，有父类和子类都有静态代码块和普通代码块，写出个代码块的执行顺序，今天就是来看一下其初始化执行的顺序

#### 测试代码
``` java
/**
 * 父类
 */
public class Father {
    public Father(){
        System.out.println("Father init.");
    }
    
    {
        System.out.println("Father code block init");
    }
    
    static {
        System.out.println("Father static block init");
    }
}

/**
 * 子类
 */
public class Son extends Father{
    public Son(){
        System.out.println("Son init.");
    }

    static {
        System.out.println("Son static block init");
    }

    {
        System.out.println("Son code block init");
    }
    private static void staticMethod(){
        System.out.println("son static method init");
    }
}

/**
 * 测试类
 */
public class Test {
    public static void main(String[] args) {
        Father father= new Father();
        System.out.println("-----------------------------------");
        Son son = new Son();
    }

}
```

#### 执行结果：

![执行结果](/images/posts/myBlog/2019-10-01-Java-Init-Process-01.png)

### 总结
#### 一个类中的初始化顺序
类内容（静态变量 => 静态代码块） => 实例内容（变量 => 一般代码块 => 构造器）

#### 两个具有继承关系类的初始化顺序
父类的（静态变量 => 静态代码块）=> 子类的（静态变量 => 静态代码块）=> 父类的（变量 => 一般代码块 => 构造器）=> 子类的（变量 => 一般代码块 => 构造器）