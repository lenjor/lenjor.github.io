---
layout: post
title: 网络通信基础（二）HTTP和HTTPS通信协议
tags: 网络通信基础  
---

# HTTP/HTTPS请求的组成部分
## 相关概念
### URI
URI的全称是（Uniform Resource Identifier），中文名称是统一资源标识符，使用它就能够唯一地标记互联网上资源。

### URL
URL的全称是（Uniform Resource Locator），中文名称是统一资源定位符，也就是我们俗称的网址，它实际上是 URI 的一个子集。
URI 不仅包括 URL，还包括 URN（统一资源名称），它们之间的关系如下
![](/images/posts/myBlog/2020-01-13-Network-Communication-Protocol-HTTPS-01.png)

### DNS
我们前面介绍了 `HTTP` 数据包传输过程中的 `TCP` 头信息会加入 `ip` 地址，但是我们访问服务器是通过域名的形式访问的，当浏览器键入 `www.google.com` 时，如何知道我们需要访问的目标 `ip` （门牌号）的？ 所以我们需要有一个解析域名的成 `IP` 地址的方法，这个协议就是 `DNS` 协议。
`DNS` 的全称是域名系统（`Domain Name System`，缩写：`DNS`），它作为将域名和 IP 地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。

### CDN
`CDN` 的全称是 `Content Delivery Network`，即内容分发网络，它应用了 `HTTP` 协议里的缓存和代理技术，代替源站响应客户端的请求。`CDN` 是构建在现有网络基础之上的网络，它依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。`CDN` 的关键技术主要有内容存储和分发技术。

打比方说你在京东下单买书，请求的收货地址是广州，那么京东就会有先选择离你最近的广州的分仓库里调货，而不选择通过远在北京的仓库调货。

## HTTPS
`HTTP` 一般是明文传输，很容易被攻击者窃取重要信息，鉴于此，`HTTPS` 应运而生。`HTTPS` 的全称为 （`Hyper Text Transfer Protocol over SecureSocket Layer`），全称有点长，`HTTPS` 和 `HTTP `有很大的不同在于 `HTTPS` 是以安全为目标的 `HTTP` 通道，在 `HTTP` 的基础上通过传输加密和身份认证保证了传输过程的安全性。`HTTPS` 在 `HTTP` 的基础上增加了 `SSL` 层，也就是说 `HTTPS` = `HTTP` + `SSL`

## HTTP/HTTPS请求包含的内容
下面是一个完整的HTTP请求：
