---
layout: post
title: 代理模式(Proxy pattern)
tags: 设计模式   
---
<!-- TOC -->

- [定义](#定义)
- [组成部分](#组成部分)
- [静态代理](#静态代理)
    - [静态代理分析](#静态代理分析)
- [动态代理（这里以JDK的动态代理为例，动态代理一般有两个主流实现：①JDK，②cglib）](#动态代理这里以jdk的动态代理为例动态代理一般有两个主流实现①jdk②cglib)
- [动态代理分析](#动态代理分析)
    - [动态代理的优点](#动态代理的优点)
- [总结](#总结)

<!-- /TOC -->

### 定义
设计模式来源于生活，这次讲的代理模式也是生活中非常常见的一种场景，如：中介，媒婆，黄牛，VPN网络代理。


代理模式：对某一个目标对象提供它的代理对象，并且由代理对象控制对原对象的引用。代理类负责为委托类预处理消息，过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。

代理分为静态代理和动态代理。
### 组成部分
为了保持行为的一致性，代理类和委托类通常会实现相同的接口，所以在访问者看来两者没有丝毫的区别

1. 抽象角色（代理的业务方法）：通过接口或抽象类声明真实角色实现的业务方法。
2. 代理角色/代理对象（中介）：实现抽象角色，是真实角色的代理，通过真实角色的业务逻辑方法来实现抽象方法，并可以附加自己的操作。
3. 真实角色/被代理对象（被代理的客户）：实现抽象角色，定义真实角色所要实现的业务逻辑，供代理角色调用。

### 静态代理

``` java
/**
 * 租房,买房
 */
public interface Rent {
    public String rent(String need);
    public String buy(int money);
}

/**
 * 客户
 */
public class Customer implements Rent {
    private String name;

    public Customer(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String rent(String need) {
        System.out.println(name + "要租的房子的需求是：" + need);
        return name + "'s Tenant.rent()";
    }

    @Override
    public String buy(int money) {
        System.out.println(name + "购买房子出价 ：¥" + money);
        return name + "'s Tenant.buy()";
    }
}


/**
 * 房地产中间商（代理）
 */
public class HouseProxy implements Rent {

    private Customer customer;

    public HouseProxy(Customer customer) {
        this.customer = customer;
        System.out.println(customer.getName() + " 成为中间商的客户");
    }

    @Override
    public String rent(String need) {
        System.out.println("我是中间商----开始干活");
        customer.rent(need);
        System.out.println("中间商干活完毕，收取差价");
        return "HouseProxy.rent()";
    }

    @Override
    public String buy(int money) {
        System.out.println("我是中间商----开始干活");
        customer.buy(money);
        System.out.println("中间商干活完毕，收取差价");
        return "HouseProxy.buy()";
    }
}

/**
 * 静态代理测试类
 */
public class StaticProxyTest {
    public static void main(String[] args){
        Customer customer = new Customer("陈大秋");
        HouseProxy houseProxy = new HouseProxy(customer);
        houseProxy.rent("独栋别野小洋楼，400平，室内泳池");
        System.out.println("");
        houseProxy.buy(4000000);
    }
}

```

**运行结果：**

![](/images/posts/myBlog/2019-10-02-Design-Patterns-Proxy-01.png)

#### 静态代理分析
从例子里面我们可以发现，代理对象和被代理对象都实现了同样的接口方法，代理对象持有被代理对象的引用，通过代理对象帮我们去完成需要实现的业务方法。

缺点：不宜扩展，代理前所有的业务是已知的，需要手写每个代理方法的过程，一个代理只能代理一种类型。


### 动态代理（这里以JDK的动态代理为例，动态代理一般有两个主流实现：①JDK，②cglib）

``` java
/**
 * 租房,买房
 */
public interface Rent {
    public String rent(String need);
    public String buy(int money);
}



/**
 * 客户
 */
public class Customer implements Rent{
    String name;

    public Customer(String name){
        this.name = name;
    }

    @Override
    public String rent(String need) {
        System.out.println(name + "要租的房子的需求是：" + need);
        return name + "'s Tenant.rent()";
    }

    @Override
    public String buy(int money) {
        System.out.println(name + "购买房子出价 ：" + money);
        return name + "'s Tenant.buy()";
    }
}


/**
 * JDK实现的动态代理对象
 */
public class JdkHouseProxy implements InvocationHandler {
    // 目标对象
    private Object target;

    /**
     * 绑定关系，也就是关联到哪个接口（与具体的实现类绑定）的哪些方法将被调用时，执行invoke方法。
     */
    public Object getInstance(Object target) {
        this.target = target;

        Class clazz = target.getClass();
        //根据传入的目标返回一个代理对象，通过字节码重组实现
        //第一个参数指定产生代理对象的类加载器，需要将其指定为和目标对象同一个类加载器
        //第二个参数要实现和目标对象一样的接口，所以只需要拿到目标对象的实现接口
        //第三个参数表明这些被拦截的方法在被拦截时需要执行哪个InvocationHandler的invoke方法
        return Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
    }


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (args != null) {
            for (int i = 0; i < args.length; i++) {
                System.out.println("arg" + i + "=" + args[i]);
            }
        }
        System.out.println("我是中间商----开始干活");
        Object invokeResult = method.invoke(target, args);
        System.out.println("中间商干活完毕，收取差价");
        System.out.println("invokeResult =" + invokeResult + "\n");
        return invokeResult;
    }
}


/**
 * JDK实现的动态代理测试类
 */
public class JdkProxyTest {
    public static void main(String[] args){
        Customer customer = new Customer("陈大春");

        Object proxyObj = new JdkHouseProxy().getInstance(customer);
        Rent rent = (Rent) proxyObj;
        String rentResult = rent.rent("独栋别野小洋楼，400平，室内泳池");
        String buyResult = rent.buy(4000000);
        System.out.println("rentResult = " + rentResult);
        System.out.println("buyResult = " + buyResult);
        System.out.println("proxyObj.getClass() = " + proxyObj.getClass());
        System.out.println("customer.getClass() = " + customer.getClass());
        System.out.println("(proxyObj == customer) =" + (proxyObj == customer));
    }
}

```

**运行结果**
![](/images/posts/myBlog/2019-10-02-Design-Patterns-Proxy-02.png)

### 动态代理分析
静态一个代理只能代理一种类型，而且是在编译器就已经确定被代理的对象。而动态代理是在运行时，通过反射机制实现动态代理，并且能够代理各种类型的对象。
在Java中要想实现动态代理机制，需要java.lang.reflect.InvocationHandler接口和 java.lang.reflect.Proxy 类的支持

java.lang.reflect.InvocationHandler接口的定义如下：
```java
//Object proxy:被代理的对象  
//Method method:要调用的方法  
//Object[] args:方法调用时所需要参数  
public interface InvocationHandler {  
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;  
}  
```

java.lang.reflect.Proxy类的定义如下：
``` java
//CLassLoader loader:类的加载器  
//Class<?> interfaces:得到全部的接口  
//InvocationHandler h:得到InvocationHandler接口的子类的实例  
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException  

```

由运行的结果可以得到，通过动态代理，所有的方法都会经过invoke方法进行转发，可以在invoke方法中在原方法执行的前后加入特定的处理，Spring 的AOP就是这个原理
JDK实现动态代理的原理，是动态生成代码，动态编译，通过字节码重组的形式生成一个新的类，图中有打印 **proxyObj** 的类对象，是 **class com.sun.proxy.$Proxy0** ，这个**Proxy0**的类对象，就是JDK动态生成的类对象，其本质是重写了一份几乎和目标对象一样的类，原方法中加入了invoke方法中的上下文代码。

#### 动态代理的优点
动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强


### 总结
代理对象就是把被代理对象包装一层，在其内部做一些额外的工作，比如翻墙VPN,在国内普通网络无法直接访问国外的网站，开启VPN网络代理即可翻墙，就访问国外网站资源，如facebook、Google。

代理模式，本质就是引入一个中间商赚差价，不直接通过被代理对象执行，而是通过统一的中间商完成需要执行的方法。对于使用者来说，和直接调用被代理对象的方法没有区别。而动态代理更进一步抽象和封装，利用字节码重组，动态的生成需要被代理的对象，使得一些特殊的操作和原来的业务逻辑分离（AOP中的拦截器，过滤器），进一步解耦，使类的职责更单一，提高可重用性。

关于JDK如果实现动态被代理对象的 **Proxy0** 对象，下面再写一篇文章手动实现JDK动态代理。