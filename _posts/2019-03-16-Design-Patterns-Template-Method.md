---
layout: post
title: 模板方法模式(Template Method Pattern)
tags: 设计模式   
---
<!-- TOC -->

- [定义](#定义)
- [业务场景](#业务场景)
    - [UML图](#uml图)
    - [代码表现](#代码表现)
        - [模板方法类](#模板方法类)
        - [具体支付类](#具体支付类)
        - [运行main方法示例](#运行main方法示例)

<!-- /TOC -->

### 定义
模板方法模式定义了一个算法的步骤，并允许子类别为一个或多个步骤提供其实践方式。让子类别在不改变算法架构的情况下，重新定义算法中的某些步骤。

什么叫做模板？既然是模板，那么就是说我使用这个模板必须要遵循模板的设计原则，以达到规范业务代码的作用

### 业务场景
小A是一间公司的程序员，老板让他开发一个聚合支付的功能（类似微信/支付宝，一个付款码可选择多种支付方式），小A查看对接的金流商googlePay，Appstore，Wechat，AliPay等等，开发完成了
有一天应届生小B作为实习生过来你们公司，要对接一个新的支付渠道PayPal，他拷贝了你的代码，修修改改，也完成了功能，后来小B离职了，支付出了故障，你查阅小B的代码，发现各种命名一塌糊涂，你心里坑定一万只草泥马在崩腾。
那么有没有一种方法，让所有的人都强制遵守我的编码规范呢？答案就是模板方法设计模式

首先我们总结一下出支付通用的步骤：
1. 创单加解密校验
2. 支付参数准备
3. 请求支付
4. 支付后状态处理

所以我们就可以抽象出三个基本的固定步骤：支付参数准备--》请求支付---》支付回调状态处理
#### UML图
![在这里插入图片描述](/images\posts\myBlog\2019-03-16-Design-Patterns-Template-Method-1.png)


#### 代码表现


##### 模板方法类

```java
/**
 * 支付抽象类，定义抽象的模板方法
 */
public abstract class AbstractPay {
    /**
     * 模板方法，定义了支付的抽象步骤
     */
    public void pay(){
        checkPay();
        preparePay();
        callPay();
        afterPay();
    }

    public abstract void checkPay();

    public abstract void preparePay();

    public abstract void callPay();

    public abstract void afterPay();

}

```

##### 具体支付类
```java
/**
 * 微信支付
 */
public class WeChatPay extends AbstractPay {
    @Override
    public void checkPay() {
        System.out.println("WeChatPay checkPay");
    }

    @Override
    public void preparePay() {
        System.out.println("WeChatPay preparePay");
    }

    @Override
    public void callPay() {
        System.out.println("WeChatPay callPay");
    }

    @Override
    public void afterPay() {
        System.out.println("WeChatPay afterPay");
    }
}



/**
 * google支付
 */
public class GooglePay extends AbstractPay {
    @Override
    public void checkPay() {
        System.out.println("GooglePay checkPay");
    }

    @Override
    public void preparePay() {
        System.out.println("GooglePay preparePay");
    }

    @Override
    public void callPay() {
        System.out.println("GooglePay callPay");
    }

    @Override
    public void afterPay() {
        System.out.println("GooglePay afterPay");
    }
}



/**
 * 支付宝支付
 */
public class AliPay extends AbstractPay {
    @Override
    public void checkPay() {
        System.out.println("AliPay checkPay");
    }

    @Override
    public void preparePay() {
        System.out.println("AliPay preparePay");
    }

    @Override
    public void callPay() {
        System.out.println("AliPay callPay");
    }

    @Override
    public void afterPay() {
        System.out.println("AliPay afterPay");
    }
}




/**
 * AppStore支付
 */
public class AppStorePay extends AbstractPay {
    @Override
    public void checkPay() {
        System.out.println("AppStorePay checkPay");
    }

    @Override
    public void preparePay() {
        System.out.println("AppStorePay preparePay");
    }

    @Override
    public void callPay() {
        System.out.println("AppStorePay callPay");
    }

    @Override
    public void afterPay() {
        System.out.println("AppStorePay afterPay");
    }
}

```


##### 运行main方法示例
```java

    public static void main(String[] args) {
        //AliPay支付
        AbstractPay pay = new AliPay();
        pay.pay();

        //google支付
        pay = new GooglePay();
        pay.pay();

        //AppStorePay支付
        pay = new AppStorePay();
        pay.pay();

        //WeChat支付
        pay = new GooglePay();
        pay.pay();

    }
```


Tip: 想要查看我的更多博文，请前往我的[个人github博客](https://lenjor.github.io/archive/)：[https://lenjor.github.io/archive/](https://lenjor.github.io/archive/)