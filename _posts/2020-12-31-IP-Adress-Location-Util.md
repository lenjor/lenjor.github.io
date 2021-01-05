---
layout: post
title: Java实现IP库归属地查询
tags: java  

---
<!-- TOC -->

- [一、IP库文件下载，各产品对比](#一ip库文件下载各产品对比)
- [二、IP库的实现有很多种，我这里采用的是GeoIP2](#二ip库的实现有很多种我这里采用的是geoip2)
    - [Jar 包依赖引入](#jar-包依赖引入)
    - [下载好IP库文件](#下载好ip库文件)
    - [代码实现](#代码实现)
    - [运行结果](#运行结果)
- [三、工程代码下载](#三工程代码下载)
- [参考文章](#参考文章)

<!-- /TOC -->

# 一、IP库文件下载，各产品对比
下面这个文章有几个比较通用的IP库产品对比分析，这里就不进行赘述了
[IP查询产品对比](https://blog.csdn.net/jack85986370/article/details/54292977)

# 二、IP库的实现有很多种，我这里采用的是GeoIP2
## Jar 包依赖引入
``` xml
<dependency>
	<groupId>com.maxmind.geoip2</groupId>
	<artifactId>geoip2</artifactId>
	<version>2.15.0</version>
</dependency>
```
## 下载好IP库文件
![](/images/posts/myBlog/2020-12-31-IP-Adress-Location-Util-01.png)

## 代码实现
``` java 

/**
 * IP工具类
 *
 * @author Lenjor
 * @version 1.0
 * @date 2020/12/31 11:21
 */
public class IpUtil {
    public static void main(String[] args) {
        String ip = "117.136.12.79";
        try {
            // 读取当前工程下的IP库文件
            URL countryUrl = IpUtil.class.getClassLoader().getResource("GeoLite2-Country.mmdb");
            URL cityFileUrl = IpUtil.class.getClassLoader().getResource("GeoLite2-City.mmdb");
            File countryFile = new File(countryUrl.getPath());
            File cityFile = new File(cityFileUrl.getPath());

            // 读取IP库文件
            DatabaseReader countryReader = (new DatabaseReader.Builder(countryFile).withCache(new CHMCache())).build();
            DatabaseReader cityReader = (new DatabaseReader.Builder(cityFile).withCache(new CHMCache())).build();
            CountryResponse countryResponse = countryReader.country(InetAddress.getByName(ip));
            Country country = countryResponse.getCountry();
            CityResponse cityResponse = cityReader.city(InetAddress.getByName(ip));
            City city = cityResponse.getCity();

            System.out.println("从country IP库读取国家结果： " + country);
            System.out.println("从city IP库读取国家结果：" + cityResponse.getCountry());
            System.out.println("从city IP库读取城市结果：" + city);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

## 运行结果
``` json
// 从country IP库读取国家结果：
{
    "geoname_id": 1814991,
    "is_in_european_union": false,
    "iso_code": "CN",
    "names": {
        "de": "China",
        "ru": "Китай",
        "pt-BR": "China",
        "ja": "中国",
        "en": "China",
        "fr": "Chine",
        "zh-CN": "中国",
        "es": "China"
    }
}

// 从city IP库读取国家结果：
{
    "geoname_id": 1814991,
    "is_in_european_union": false,
    "iso_code": "CN",
    "names": {
        "de": "China",
        "ru": "Китай",
        "pt-BR": "China",
        "ja": "中国",
        "en": "China",
        "fr": "Chine",
        "zh-CN": "中国",
        "es": "China"
    }
}

// 从city IP库读取城市结果：
{
    "geoname_id": 1790437,
    "names": {
        "de": "Zhuhai",
        "ru": "Чжухай",
        "ja": "珠海",
        "en": "Zhuhai",
        "fr": "Zhuhai",
        "zh-CN": "珠海市"
    }
}
```

## 结果分析
从结果上面来看，功能已经是实现完毕了，不过有一个问题就是IP的识别准确率的问题，随后我对比了各家的IP识别，都存在有IP的识别准确率的问题，总得来说就是收费识别准确率会更高，有条件的可以购买对应的IP查询的产品服务


# 三、工程代码下载
Git项目地址：[IP库实现IP归属地查询](https://gitee.com/Lj_coding/My-Learning/tree/master/ip)

# 参考文章
1. [各个ip地址库对比与java实现](https://blog.csdn.net/jack85986370/article/details/54292977)
2. [GeoIP2 Java API](https://maxmind.github.io/GeoIP2-java/)