---
layout: post
title: 装饰模式(Decorator Pattern)
tags: 设计模式   
---
<!-- TOC -->

- [定义](#定义)
- [应用场景](#应用场景)
- [特点](#特点)
- [讲故事](#讲故事)
- [UML类图(以男性为例）](#uml类图以男性为例)
- [代码表现形式](#代码表现形式)
    - [定义接口和类](#定义接口和类)
    - [下面是三个具体的男性衣服装饰类](#下面是三个具体的男性衣服装饰类)
    - [运行的main方法](#运行的main方法)
    - [最后附上运行结果](#最后附上运行结果)
- [总结](#总结)

<!-- /TOC -->


#### 定义
装饰模式指的是在不必改变原类文件和使用继承的情况下，动态地扩展一个对象的功能。它是通过创建一个包装对象，也就是装饰来包裹真实的对象。

#### 应用场景
从定义可以得出，装饰模式的作用是在不改变原来对象的情况下，往对象添加一些特定功能，符合对象的修改封闭——扩展开放原则

#### 特点
（1） 装饰对象和真实对象有相同的接口。这样客户端对象就能以和真实对象相同的方式和装饰对象交互。
（2） 装饰对象包含一个真实对象的引用（reference）
（3） 装饰对象接受所有来自客户端的请求。它把这些请求转发给真实的对象。
（4） 装饰对象可以在转发这些请求以前或以后增加一些附加功能。这样就确保了在运行时，不用修改给定对象的结构就可以在外部增加附加的功能。在面向对象的设计中，通常是通过继承来实现对给定类的功能扩展。

#### 讲故事
我们假设一个场景，我们人都要穿衣服，我们可以选择穿不同类型的衣服，也就是不同的装饰，假设只要求我们输出所带有的装饰。
假设男性的衣服有领带，皮鞋，长裤，衬衫，运行库
女性有文胸，长袖，羊毛衫，超短裙，丝袜，手提包

#### UML类图(以男性为例）
![在这里插入图片描述](/images/posts/myBlog/2019-03-15-Design-Patterns-Decorator-1.png)
#### 代码表现形式
##### 定义接口和类
```java

/**
 * Person接口
 */
public interface Person {
    void wear();
}


/**
 * 穿衣服的男人
 */
public class Man implements Person {
    private String clothes;

    @Override
    public void wear() {
        System.out.println("I am a Man wear " + clothes);
    }

    public String getClothes() {
        return clothes;
    }

    public void setClothes(String clothes) {
        this.clothes = clothes;
    }
}

/**
 * 装饰器抽象类
 */
public abstract class Decorator implements Person {
    private Person person;

    public void setPerson(Person person) {
        this.person = person;
    }

    @Override
    public void wear() {
        person.wear();
    }
}
```

##### 下面是三个具体的男性衣服装饰类
```java
/**
 * wear pants
 */
public class ManDecoratorA extends Decorator {

    @Override
    public void wear() {
        super.wear();
        System.out.println("ManDecoratorA wear pants");
    }
}


/**
 * wear shoes
 */
public class ManDecoratorB extends Decorator {

    @Override
    public void wear() {
        super.wear();
        System.out.println("ManDecoratorB wear shoes");
    }
}

/**
 * wear shirt
 */
public class ManDecoratorC extends Decorator {

    @Override
    public void wear() {
        super.wear();
        System.out.println("ManDecoratorC wear shirt");
    }
}


```

##### 运行的main方法
```java
	/**
	* 下面是main方法
	*/
    public static void main(String[] args) {
        Man man = new Man();
        man.setClothes("shirt,pants,shoes");

        Decorator decoratorA = new ManDecoratorA();
        decoratorA.setPerson(man);

        Decorator decoratorB = new ManDecoratorB();
        decoratorB.setPerson(decoratorA);

        Decorator decoratorC = new ManDecoratorC();
        decoratorC.setPerson(decoratorB);
        decoratorC.wear();
    }
```

##### 最后附上运行结果
![在这里插入图片描述](/images\posts\myBlog\2019-03-15-Design-Patterns-Decorator-2.png)

#### 总结
装饰模式简单来说就是不断地往对象添加功能，就是用更大的盒子包装原来的盒子，不破坏原来盒子的内部结构。
装饰模式要注意装饰的顺序问题。
判断是否是装饰模式简单的方法是，Decorator中有一个指向Component（本案例是Person）对象的指针，并定义一个与Component接口一致的接口（本案例是wear接口）。