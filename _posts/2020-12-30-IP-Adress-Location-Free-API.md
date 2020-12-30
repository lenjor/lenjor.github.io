---
layout: post
title: 免费常用IP归属地查询API
tags: java  

---

记录几个免费的IP地址归属地查询的接口，其实这些IP归属地的查询，实现起来也不难，主要是要有一个IP库文件，就能实现，以后有空自己实现一下
1. 本地IP查询，ip-api.com 可切换显示语言
http://ip-api.com/json/{ip}?lang=zh-CN
``` json
{
    "status": "success",
    "country": "中国",
    "countryCode": "CN",
    "region": "GD",
    "regionName": "广东",
    "city": "广州市",
    "zip": "",
    "lat": 23.1181,
    "lon": 113.2539,
    "timezone": "Asia/Shanghai",
    "isp": "China Mobile communications corporation",
    "org": "China Mobile",
    "as": "AS56040 China Mobile communications corporation",
    "query": "117.136.12.79"
}
```

2. 百度
http://opendata.baidu.com/api.php?query=117.136.12.79&co=&resource_id=6006&oe=utf8
``` json
{
    "status": "0",
    "t": "",
    "set_cache_time": "",
    "data": [
        {
            "location": "广东省广州市 移动",
            "titlecont": "IP地址查询",
            "origip": "117.136.12.79",
            "origipquery": "117.136.12.79",
            "showlamp": "1",
            "showLikeShare": 1,
            "shareImage": 1,
            "ExtendedLocation": "",
            "OriginQuery": "117.136.12.79",
            "tplt": "ip",
            "resourceid": "6006",
            "fetchkey": "117.136.12.79",
            "appinfo": "",
            "role_id": 0,
            "disp_type": 0
        }
    ]
}
```


3. 太平洋
http://whois.pconline.com.cn/ipJson.jsp?ip=117.136.12.79&json=true
``` json
{
    "ip": "117.136.12.79",
    "pro": "广东省",
    "proCode": "440000",
    "city": "佛山市",
    "cityCode": "440600",
    "region": "",
    "regionCode": "0",
    "addr": "广东省佛山市 移动",
    "regionNames": "",
    "err": ""
}
```

4. 126
``` JavaScript
var lo="广东省", lc="";
var localAddress={city:"", province:"广东省"}
```


