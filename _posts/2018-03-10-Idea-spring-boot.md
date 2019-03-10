---
layout: post
title: 简单聊聊工厂设计模式
tags: 设计模式   
---

<!-- TOC -->

- [工厂设计模式概述](#工厂设计模式概述)
    - [需求描述](#需求描述)
    - [不使用任何设计模式](#不使用任何设计模式)
    - [简单工厂模式（静态工厂）](#简单工厂模式静态工厂)
    - [工厂设计模式](#工厂设计模式)
    - [抽象工厂](#抽象工厂)
    - [总结](#总结)

<!-- /TOC -->


### 工厂设计模式概述
工厂设计模式根据其抽象程度可以分为:简单工厂（静态工厂），一般工厂，抽象工厂
我们学习一种设计模式，首先要搞清楚其应用场景，为什么需要引入这种设计模式，有什么优点和缺点。
这里先埋个坑：实际编码使用反射能更灵活获取到相应的对象。反射反射，程序员的最爱

工厂设计模式的优点：
1. 可以使代码结构更清晰，有效地封装变化。
2. 对调用者屏蔽具体的产品类。
3. 降低耦合度。

#### 需求描述
我们先来看一个小程序：
需求描述：我需要生产汽车（Car类），让汽车跑起来，汽车包含引擎（Engine类）、车架（Frame类）、车窗玻璃（Glass类）、轮子（Wheel类）

#### 不使用任何设计模式
```java
/**
 * 引擎类
 */
public class Engine {
    private String engine;
    public Engine(String engine){
        this.engine = engine;
    }
}
/**
* 玻璃类
*/
public class Glass {
	private String glass;
    public Glass(String glass){
        this.glass = glass;
    }
}
/**
 * 车架类
 */
public class Frame {
	private String frame;
    public Frame(String frame){
        this.frame = frame;
    }
}
/**
 * 轮子类
 */
public class Wheel {
    private String wheel;
    public Wheel(String wheel){
        this.wheel = wheel;
    }
}

/**
 * 汽车类
 */
public class Car {

    private static final Logger logger = LoggerFactory.getLogger(Car.class);
    private Engine engine;
    private Frame frame;
    private Glass glass;
    private Wheel wheel;
    public Car(Engine engine,Frame frame,Glass glass,Wheel wheel){
        this.engine = engine;
        this.frame = frame;
        this.glass = glass;
        this.wheel = wheel;

    }

    public void run() {
        logger.info("汽车启动");
    }
}

/**
 * 客户端类
 */
public class Client {
    private static final Logger logger = LoggerFactory.getLogger(Client.class);

	public static void main(String [] args){
        Engine engine = new Engine("奔驰");
        Frame frame = new Frame("奔驰");
        Glass glass = new Glass("奔驰");
        Wheel wheel = new Wheel("奔驰");
        Car car = new Car(engine,frame,glass,wheel);
        car.run();
    }
}
```
从上面的代码我们可以发现，客户他只想要车子，调用车子的run（）方法，但是前面却不得不实例化很多他不需要的对象（Engine，Frame，Glass，Wheel），而且也没有办法扩展，现在是生产奔驰汽车，那以后生产其他的汽车，还得一个个给这个汽车配相应的配件。下面我们就开始封装“变化”。

#### 简单工厂模式（静态工厂）
```java
/**
 * 汽车接口
 */
public interface ICar {
    void run();
}

```
```java
/**
 * 汽车父类
 */
public class Car implements ICar {
    private Engine engine;
    private Frame frame;
    private Glass glass;
    private Wheel wheel;
    @Override
    public void run() {

    }

    public Engine getEngine() {
        return engine;
    }

    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    public Frame getFrame() {
        return frame;
    }

    public void setFrame(Frame frame) {
        this.frame = frame;
    }

    public Glass getGlass() {
        return glass;
    }

    public void setGlass(Glass glass) {
        this.glass = glass;
    }

    public Wheel getWheel() {
        return wheel;
    }

    public void setWheel(Wheel wheel) {
        this.wheel = wheel;
    }
}

```
```java

/**
 * 奔驰汽车类
 */
public class BenzCar extends Car {
    private static final Logger logger = LoggerFactory.getLogger(BenzCar.class);
    BenzCar(){
        this.setEngine(new Engine("奔驰"));
        this.setFrame(new Frame("奔驰"));
        this.setGlass(new Glass("奔驰"));
        this.setWheel(new Wheel("奔驰"));
    }
    @Override
    public void run() {
        logger.info("奔驰汽车启动");
    }
}

```
```java
/**
 * 奥迪汽车
 */
public class AudiCar extends Car{
    private static final Logger logger = LoggerFactory.getLogger(BenzCar.class);
    AudiCar(){
        this.setEngine(new Engine("奥迪"));
        this.setFrame(new Frame("奥迪"));
        this.setGlass(new Glass("奥迪"));
        this.setWheel(new Wheel("奥迪"));
    }
    @Override
    public void run() {
        logger.info("奥迪汽车启动");
    }
}


```
```java

/**
 * 工厂接口
 */
public interface IFactory {
    Car getCar(String type);
}

```
```java
/**
 * 工厂类
 */
public class Factory implements IFactory {
    public static final Logger logger = LoggerFactory.getLogger(Factory.class);

    private Car car;

    @Override
    public Car getCar(String type) {
        switch (type) {
            case "奔驰":
                car = new BenzCar();
                break;
            case "奥迪":
                car = new AudiCar();
                break;
            default:
                break;
        }
        return car;
    }
}

```
客户端调用代码：
```java
/**
 * 客户端代码
 */
public class Client {

    public static void main(String [] args){
        Factory factory = new Factory();
        Car benZCar = factory.getCar("奔驰");
        Car audiCar = factory.getCar("奥迪");
        audiCar.run();
    }
}
```
从客户端的代码我们可以看出来，客户端只关心得到奔驰汽车或奥迪汽车，至于其实例化交给工厂类去实现，其实工厂的本质就是，将对象的创建延迟到工厂类中，通过封装“变化”，这里的变化就是传到工厂类里面的订单类型。
简单工厂类的最大优点在于在工厂类中包含了必要的逻辑判断，根据客户端的选择条件动态实例化相关的类，对客户端来说，去除了与具体产品的依赖

	那么我们以后需要增加多一种汽车的类型的时候需要做些什么工作呢？
	1. 多定义一个汽车类继承Car类
	2. 在工厂类中的switch中加入相应的“订单”类型判断，实例化对象
	3. 在客户端传入相应的订单类型
	
但是这样做可能会出现一些问题：
	
	每次增加一种汽车类型，都要修改工厂类，在工厂类种加入逻辑判断，这就会影响原有的工厂功能（埋坑）
工厂设计模式，这时候的优点就体现出来了。

#### 工厂设计模式
工厂设计模式和简单工厂设计模式代码查不多，差别在于工厂类和客户端
```java
/**
 * 工厂接口
 */
public interface IFactory {
    Car getCar();
}
```
```java
/**
 * 奔驰工厂类
 */
public class BenzFactory implements IFactory {
    public static final Logger logger = LoggerFactory.getLogger(BenzFactory.class);
    @Override
    public Car getCar() {
        return new BenzCar();
    }
}
```

```java
/**
 * 奥迪工厂类
 */
public class AudiFactory implements IFactory {
    public static final Logger logger = LoggerFactory.getLogger(AudiFactory.class);
    @Override
    public Car getCar() {
        return new AudiCar();
    }
}
```

```java
/**
 * 客户端代码
 */
public class Client {

    public static void main(String [] args){
        BenzFactory benzFactory = new BenzFactory();
        Car benZCar = benzFactory.getCar();
        benZCar.run();
        
        AudiFactory audiFactory = new AudiFactory();
        Car audiCar = audiFactory.getCar();
        audiCar.run();
    }
}
```
从这里我们可以看出来了，现在的工厂是具体的汽车工厂，也就是说，客户想要什么类型的汽车，需要到相应的工厂里下单。工厂不在包含逻辑判断，以后需求更改需要更多种类的汽车，只需要定义汽车相应的汽车类和工厂类，修改客户的提车过程即可，并不会影响原来工厂的生产线，更加的安全，也符合开放-封闭原则。

#### 抽象工厂
抽象工厂，顾名思义，就是抽象的工厂。介绍抽象工厂就要引入产品族的概念，比如生产的奔驰汽车，有排量1.4和排量2.0的两类汽车，而不同排量又有两座的跑车和四座的商务车，这两座和四座的不同类型的汽车就构成了两个不同的产品族。
我们假设，排量在于引擎不同，两座和四座的区别在于车架不同。
四个基本类（Engine，Frame，Glass，Wheel）不变
```java
/**
 * 汽车接口
 */
public interface ICar {
    void run();
}
```
```java
/**
 * 两座汽车接口
 */
public interface ISportCar {
    void sportCarRun();
}
```

```java
/**
 * 4座商务车接口
 */
public interface IBusinessCar {
    void bussinessCarRun();
}

```


```java
/**
 * 奔驰跑车类,排量1.4
 */
public class SportCar1 extends Car implements ISportCar{
    private static final Logger logger = LoggerFactory.getLogger(SportCar1.class);
    SportCar1(){
        this.setEngine(new Engine("1.4排量奔驰"));
        this.setFrame(new Frame("两座跑车"));
        this.setGlass(new Glass("奔驰"));
        this.setWheel(new Wheel("奔驰"));
    }
    @Override
    public void run() {
        logger.info("奔驰汽车启动");
    }

    @Override
    public void sportCarRun() {
        logger.info(this.getFrame().getFrame() + this.getEngine().getEngine() +"启动");
    }
}
```
```java
/**
 * 奔驰跑车类，排量2.0
 */
public class SportCar2 extends Car implements ISportCar{
    private static final Logger logger = LoggerFactory.getLogger(SportCar2.class);
    SportCar2(){
        this.setEngine(new Engine("排量2.0奔驰"));
        this.setFrame(new Frame("两座跑车"));
        this.setGlass(new Glass("奔驰"));
        this.setWheel(new Wheel("奔驰"));
    }
    @Override
    public void run() {
        logger.info("奔驰汽车启动");
    }

    @Override
    public void sportCarRun() {
        logger.info(this.getFrame().getFrame() + this.getEngine().getEngine() +"启动");
    }
}
```
```java
/**
 * 奔驰4座商务车类，排量1.4
 */
public class BussinessCar1 extends Car implements IBusinessCar{
    private static final Logger logger = LoggerFactory.getLogger(BussinessCar1.class);
    BussinessCar1(){
        this.setEngine(new Engine("1.4排量奔驰"));
        this.setFrame(new Frame("四座商务车"));
        this.setGlass(new Glass("奔驰"));
        this.setWheel(new Wheel("奔驰"));
    }
    @Override
    public void run() {
        logger.info(this.getFrame().getFrame() + "启动");
    }

    @Override
    public void bussinessCarRun() {
        logger.info(this.getFrame().getFrame() + this.getEngine().getEngine() +"启动");
    }
}

```
```java
/**
 * 奔驰4座商务车类，排量2.0
 */
public class BussinessCar2 extends Car implements IBusinessCar{
    private static final Logger logger = LoggerFactory.getLogger(BussinessCar2.class);
    BussinessCar2(){
        this.setEngine(new Engine("2.0排量奔驰"));
        this.setFrame(new Frame("四座商务车"));
        this.setGlass(new Glass("奔驰"));
        this.setWheel(new Wheel("奔驰"));
    }
    @Override
    public void run() {
        logger.info(this.getFrame().getFrame() + "启动");
    }

    @Override
    public void bussinessCarRun() {
        logger.info(this.getFrame().getFrame() + this.getEngine().getEngine() +"启动");
    }
}
```
```java
/**
 * 抽象工厂接口
 */
public interface IFactory {
    Car getSportCar();
    Car getBusinessCar();
}
```
```java

/**
 * 奔驰工厂类,生产排量1.4的汽车
 */
public class Factory1 implements IFactory {
    public static final Logger logger = LoggerFactory.getLogger(Factory1.class);

    @Override
    public Car getSportCar() {
        return new SportCar1();
    }

    @Override
    public Car getBusinessCar() {
        return new BussinessCar1();
    }
}

```
```java
/**
 * 奔驰工厂类，生产排量2.0的汽车
 */
public class Factory2 implements IFactory {
    public static final Logger logger = LoggerFactory.getLogger(Factory2.class);

    @Override
    public Car getSportCar() {
        return new SportCar2();
    }

    @Override
    public Car getBusinessCar() {
        return new BussinessCar2();
    }
}
```
```java
/**
 * 客户端代码
 */
public class Client {
    public static void main(String [] args){
        //排量1.4
        Factory1 factory1 = new Factory1();
        SportCar1 sportCar1 = (SportCar1) factory1.getSportCar();
        BussinessCar1 businessCar1 = (BussinessCar1) factory1.getBusinessCar();
        sportCar1.sportCarRun();
        businessCar1.bussinessCarRun();

        //排量2.0
        Factory2 factory2 = new Factory2();
        SportCar2 sportCar2 = (SportCar2) factory2.getSportCar();
        BussinessCar2 businessCar2 = (BussinessCar2) factory2.getBusinessCar();
        sportCar2.sportCarRun();
        businessCar2.bussinessCarRun();
    }
}
```
抽象工厂，也就是工厂是抽象的，根据用户的订单需要（排量），去生产对应排量的工厂下单，再生产出对应的跑车或商务车。我们任务，同样是排量1.4的跑车和商务车就是一个产品族。
工厂是抽象的，定义了抽象的接口，而具体排量的工厂去实现不同排量的汽车生产，引入抽象工厂设计模式，是因为产品同属于一个系列，由同类的工厂同一管理创建，如果没有严格的产品等级划分，抽象工厂反而显得很臃肿。

#### 总结
三种设计模式的区别已经介绍完了，不知道小伙伴们有没有发现，其实我们的log4j日志的logger对象就是通过工厂类获取到的对象，其实工厂设计模式就是为了解决对象的创建问题，并把对象的创建延迟到工厂类中，而达到客户端无需关心对象创建的过程，只关心获取到相应的对象，这就把对象的创建过程封闭。

**其实我个人感觉上还是使用发射会更加方便和快捷，毕竟是程序员的最爱啊**

文章受 程杰 的 《大话设计模式》启发，对设计模式感兴趣的小伙伴可以拜读一下这本书。

最后，感谢各位的阅读。阅读代码，请点击[代码git仓库](https://gitee.com/Lj_coding/My-Learning.git)
链接：https://gitee.com/Lj_coding/My-Learning.git