---
layout: post
title: 策略模式(Strategy pattern)
tags: 设计模式   
---

<!-- TOC -->

- [定义](#定义)
- [示例代码场景描述](#示例代码场景描述)
- [源代码](#源代码)
    - [支付简单工厂（switch 和反射两种实现）](#支付简单工厂switch-和反射两种实现)
    - [测试客户端](#测试客户端)
- [运行结果](#运行结果)
- [总结](#总结)

<!-- /TOC -->

### 定义
策略模式：指的是对象具备某个行为，但是在不同的场景中，该行为有不同的实现算法。
举个几个生活的例子，就是不同收入的人群采用不同的税率，登录的不同方式，支付的不同方式；
在代码我们也见过策略模式，如ArrayList的排序传入的Comparator

``` java
    ArrayList list = new ArrayList();
    list.sort(new Comparator() {
        @Override
        public int compare(Object o1, Object o2) {
            //此处的本质就是一个策略
            return 0;
        }
    });
```


### 示例代码场景描述
用户可以选择不同类型的支付方式，有支付宝支付，微信支付，京东白条支付，银联支付，根据用户选择地不同支付方法，执行不同的支付方式的代码。

### 源代码
```java
/**
 * 支付接口
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:23
 */
public interface Pay {
    public void pay(Order order);
}


/**
 * 支付宝支付
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:22
 */
public class AliPay implements Pay {

    public void pay(Order order) {
        System.out.println("使用支付宝支付");
        System.out.println(order);
    }
}




/**
 * 京东支付
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:22
 */
public class JDPay implements Pay {
    public void pay(Order order) {
        System.out.println("使用京东白条支付");
        System.out.println(order);
    }
}




/**
 * 微信支付
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:22
 */
public class WechatPay implements Pay {
    public void pay(Order order) {
        System.out.println("使用微信支付");
        System.out.println(order);
    }
}


/**
 * 银联支付
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:23
 */
public class UnionPay implements Pay {
    public void pay(Order order) {
        System.out.println("使用银联支付");
        System.out.println(order);
    }
}




/**
 * 订单类
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:24
 */
public class Order {
    private String orderNo;
    private double amount;
    private String product;

    public Order(double amount, String product) {
        this.amount = amount;
        this.product = product;
        this.orderNo = ("NO." + System.currentTimeMillis() + System.nanoTime()).substring(0, 32);
    }

    public String getOrderNo() {
        return orderNo;
    }

    public void setOrderNo(String orderNo) {
        this.orderNo = orderNo;
    }

    public double getAmount() {
        return amount;
    }

    public void setAmount(double amount) {
        this.amount = amount;
    }

    public String getProduct() {
        return product;
    }

    public void setProduct(String product) {
        this.product = product;
    }

    @Override
    public String toString() {
        return "[商品名: " + product + " , 价格: ¥ " + amount + " , 订单号：" + orderNo + " ]";
    }
}



/**
 * 支付类型
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:35
 */
public enum PayType {
    ALI_PAY("AliPay"),
    WECHAT_PAY("WechatPay"),
    JD_PAY("JDPay"),
    UNION_PAY("UnionPay");

    private final String payType;

    private PayType(String payType) {
        this.payType = payType;
    }

    public String getPayType() {
        return this.payType;
    }
}

```

#### 支付简单工厂（switch 和反射两种实现）
``` java
/**
 * 支付简单工厂
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:34
 */
public class PayFactory {
    private Pay pay = null;

    private PayFactory(PayType payType) {
        //做法一：直接使用switch获取到对应类型的支付对象
//        switch (payType) {
//            case ALI_PAY:
//                pay = new AliPay();
//                break;
//            case JD_PAY:
//                pay = new JDPay();
//                break;
//            case WECHAT_PAY:
//                pay = new WechatPay();
//                break;
//            case UNION_PAY:
//                pay = new WechatPay();
//                break;
//        }


        try {
            //做法二：利用反射获取对应类型的支付对象
            Class clazz = Class.forName(this.getClass().getPackage().getName() + "." + payType.getPayType());
            pay = (Pay) clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }

    }

    public static PayFactory getInstance(PayType payType) {
        return new PayFactory(payType);
    }

    public void pay(Order order) {
        pay.pay(order);
    }
}

```

#### 测试客户端
``` java
/**
 * 测试类
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 16:21
 */
public class Client {

    public static void main(String[] args) {
        Order order = new Order(999.99, "RedMi 7 pro");
        PayFactory payFactory = PayFactory.getInstance(PayType.ALI_PAY);
        payFactory.pay(order);
    }
}
```

### 运行结果
![](/images/posts/myBlog/2019-10-03-Design-Patterns-Strategy-01.png)


### 总结
策略模式关心的是选择不同，策略就不同，侧重点是不同的选择导致不同的策略。所以关键在于，把不同的算法（策略）封装，使得不同的算法之间不会相互影响，当需要增加一种新的算法（策略）时，只需要新增一个新的策略类和一个枚举值即可，不会影响到其他原有的策略。

设计模式从来都不会是单独存在的，这里除了使用到了策略模式，也使用到了简单工厂模式，而工厂类持有一个Pay对象的引用，又有点像静态代理的味道，只是这里工厂类这里没有实现Pay接口。
