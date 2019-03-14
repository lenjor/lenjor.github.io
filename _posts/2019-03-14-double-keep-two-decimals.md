---
layout: post
title: double保留两位小数
tags: java   
---
<!-- TOC -->

- [返回类型为double(四舍五入)](#返回类型为double四舍五入)
- [返回类型是 String](#返回类型是-string)

<!-- /TOC -->

我们都知道double和float都是浮点型，在转型或者比较的时候可能出现问题，这里讲一下怎么针对double类型做精度处理

#### 返回类型为double(四舍五入)
1. 使用Math.round转成long再转回double
```java
        double dou = 3.1487426;
        dou = (double) Math.round(dou * 100) / 100;   
        System.out.println(dou);
```
2. 使用BigDecimal进行格式化
```java
        double dou = 3.1487426;
        BigDecimal bigDecimal = new BigDecimal(dou).setScale(2, RoundingMode.HALF_UP);
        double newDouble = bigDecimal.doubleValue();
        System.out.println(newDouble);
```

#### 返回类型是 String
1. 使用String.format()格式化(四舍五入)
```java
        double dou = 3.1487426;
        String douStr = String.format("%.2f", dou)
        System.out.println(douStr);
``` 

2. 使用NumberFormat进行格式化(四舍五入)
```java
        double dou = 3.1487426;
        NumberFormat numberFormat = NumberFormat.getNumberInstance();
        numberFormat.setMaximumFractionDigits(2);
        // numberFormat.setRoundingMode(RoundingMode.UP);  //默认的模式就是四舍五入,可以省略
        String douStr = numberFormat.format(dou)
        System.out.println(douStr);
```

3. 使用BigDecimal进行格式化(四舍五入)
```java
        double dou = 3.1487426;
        DecimalFormat decimalFormat = new DecimalFormat("#.00");
        String str = decimalFormat.format(dou);
        System.out.println(str);
```


