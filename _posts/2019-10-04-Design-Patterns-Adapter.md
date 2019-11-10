---
layout: post
title: 适配器模式(Adapter pattern)
tags: 设计模式   
---

### 定义
适配器模式(Adapter Pattern)：将一个接口转换成客户希望的另一个接口，使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

### 适用场景
1、 你想使用一个已经存在的类，而它的接口不符合你的需求
2、 你想创建一个可以复用的类，该类可以与其他不相关的类或不可预见的类（即那些接口不一定兼容的类）协同工作。
3、 （仅适用于对象Adapter）你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口。对象适配器可以适配它的父类接口。


### 示例代码

``` java
/**
 * 测试类
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-11-10 18:37
 */
public class Client {

    public static void main(String[] args) {
        Phone phone = new Phone();
        ChinaAdapter adapter = new ChinaAdapter();
        phone.setAdapter(adapter);
        phone.charge();

        JapanAdapter japanAdapter = new JapanAdapter();
        phone.setJapanAdapter(japanAdapter);
        phone.japanCharge();
    }

}



/**
 * 充电接口
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-10-03 23:14
 */
public interface Charge {
    public int charge();
}


/**
 * 中国充电适配器
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-11-10 18:38
 */
public class ChinaAdapter implements Charge {

    private final static int INPUT = 220;

    public int charge() {
        System.out.println("正在充电...");
        System.out.println("原始电压：" + INPUT + "V");
        int output = INPUT - 200;
        System.out.println("经过中国变压器转换之后的电压:" + output + "V");
        return output;
    }
}


/**
 * 日本充电适配器
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-11-10 18:48
 */
public class JapanAdapter implements Charge{
    private final static int INPUT = 110;

    public int charge() {
        System.out.println("正在充电...");
        System.out.println("原始电压：" + INPUT + "V");
        int output = INPUT - 90;
        System.out.println("经过日本变压器转换之后的电压:" + output + "V");
        return output;
    }
}

/**
 * 手机
 *
 * @author Lenjor
 * @version 1.0
 * @date 2019-11-10 18:38
 */
public class Phone {

    private ChinaAdapter chinaAdapter;

    private JapanAdapter japanAdapter;

    public int charge () {
        return chinaAdapter.charge();
    }

    public void setAdapter(ChinaAdapter adapter) {
        this.chinaAdapter = adapter;
    }

    public int japanCharge () {
        return japanAdapter.charge();
    }

    public void setJapanAdapter(JapanAdapter adapter) {
        this.japanAdapter = adapter;
    }
}

```


### 测试结果：
![](/images/posts/myBlog/2019-10-04-Design-Patterns-Adapter-01.png)


### 总结
适配器模式，简单来说就是转换器，当我们需要本来不兼容的两个类协同工作的时候，就可以采用适配器让两者联系起来，在适配器内部做转换，对用户来说是无感的，只需要根据场景使用对应的适配器即可。