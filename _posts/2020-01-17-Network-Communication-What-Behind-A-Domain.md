---
layout: post
title: 网络通信基础（三）浏览器输入网址背后发生了什么
tags: 网络通信基础  

---
<!-- TOC -->

- [问题：当我们输入一个域名后，发生了什么](#问题当我们输入一个域名后发生了什么)
- [一、DNS 域名解析](#一dns-域名解析)
- [二、发送数据包](#二发送数据包)
    - [以太帧组成](#以太帧组成)
- [三、`ARP` 地址解析协议（IPv4使用，IPv6使用 NDP协议）](#三arp-地址解析协议ipv4使用ipv6使用-ndp协议)
    - [`ARP` 工作过程详解](#arp-工作过程详解)
- [四、数据包传输过程总结](#四数据包传输过程总结)
- [参考文章](#参考文章)

<!-- /TOC -->

# 问题：当我们输入一个域名后，发生了什么
当我们在浏览器输入一个域名，按下回车键之后，到我们能够看到页面，这个过程中，到底发生了什么事情，下面我们开始一步步分析。

# 一、DNS 域名解析
`DNS` 域名解析主要可以分为三个过程
1. 读缓存

    浏览器会把已经解析过的域名缓存起来，如果缓存中存在，解析结束。缓存的时效性可以通过TTL属性设置
2. 读本地Host文件

    如果缓存中没有该域名，Windows系统会读取本地的Host文件（这也是我们最常用的修改域名映射地址的方法）
3. 请求域名服务器

    请求域名服务器这个过程就需要联系前面学过的`HTTP`通信协议了，这个过程就是通过`HTTP`协议实现主机和域名服务器的交互的，具体的交互过程如下图（[图片来源：DNS域名解析全过程](https://blog.csdn.net/m0_37812513/article/details/78775629)）

![](/images/posts/myBlog/2020-01-17-Network-Communication-What-Behind-A-Domain-01.png)

    1. 请求本地域名服务器（LDNS）来解析这个域名，这台服务器一般都在所在城市附近，同样会缓存其域名解析结果，大约80%的域名解析到这里就完成了。
    2. 如果LDNS没有该域名映射，则请求根域名服务器（`Root Server`） 请求解析
    3. 根域名服务器返回给LDNS一个所查询域的主域名服务器（gTLD Server，国际顶尖域名服务器，如.com .cn .org等）地址
    4. 此时LDNS再请求gTLD域名服务器解析需要解析的域名
    5. gTLD域名服务器返回这个域名对应的`Name Server`的地址，这个`Name Server`就是网站注册的域名服务器
    6. Name Server根据映射关系表找到目标ip，返回给LDNS
    7. LDNS缓存这个域名和对应的ip
    8. LDNS把解析的结果返回

# 二、发送数据包
上面的步骤解析完域名之后，就拿到了域名对应的ip地址，根据前面 [HTTP（TCP/IP）通信协议](/2020/01/Network-Communication-Protocol-HTTP/) 我们知道，浏览器是在应用层的，接下来就是往下逐层加上对应的协议头，具体过程不再累述。这里重点讲一下其寻址过程。

## 以太帧组成
结合前面的知识，我们已经知道了，数据帧（以太帧）的组成应该如下图所示（以太帧的组成版本比较多，这里采用的说明版本是 IEEE 802.3 ）：
![](/images/posts/myBlog/2020-01-17-Network-Communication-What-Behind-A-Domain-02.png)

以太帧组成：
- 前导码：7 个字节，用于数据传输过程中的双方发送与接收的速率的同步。
- SFD：帧开始符，1 个字节，用于标识一个以太网帧的开始。 
- 目的 MAC 地址：6 个字节，指明帧的接收者。
- 源 MAC 地址：6 个字节，指明帧的发送者。
- 长度：2 个字节，指明该帧数据字段的长度，但不代表数据字段长度能够达到 2^16 字节。
- 类型：2 个字节，指明帧中数据的协议类型，比如常见的 IPv4 中的 ip 协议采用 0x0800。
- 数据与填充：46~1500 个字节，包含了上层协议传递下来的数据，如果加入数据字段后帧长度不够 64 字节，会在数据字段加入填充字段达到 64 字节。

校验和：4 个字节，对接收网卡(主要是检测数据与填充字段)提供判断是否传输错误的一种方法，如果发现错误，则丢弃此帧。目前最为流行的用于校验和(FCS)的算法是循环冗余校验(cyclic redundancy check -- CRC)。

# 三、`ARP` 地址解析协议（IPv4使用，IPv6使用 NDP协议）
前面我们已经知道TCP协议里面地址的标记是IP，但是并不能直接使用IP地址通信，因为IP地址并不能确定具体的机器，打个最简单的比方就是同一个WiFi的所有设备的IP地址都是同一个，一个设备唯一的标识是Mac地址。网络中传输是需要经过很多路由的，只有IP目标地址显然是不够的，这就需要用
地址解析协议（`Address Resolution Protocol`），其基本功能为透过目标设备的IP地址，查询目标设备的MAC地址，以保证通信的顺利进行。

数据在不同的路由之间传输，经过的每一个跳，都需要把以太帧拆包，移除自身的Mac地址，确定下一跳的Mac地址，如果本地的ARP表中没有下一跳地址，则使用ARP协议来广播查询得到，使用 `traceroute` 命令可以查询经过的路由，如下图所示，经过了很多的路由IP。
![](/images/posts/myBlog/2020-01-17-Network-Communication-What-Behind-A-Domain-03.png)


## `ARP` 工作过程详解
1. 当主机A想本局域网上的某个主机B发送IP数据报时，就先在自己的`ARP`缓冲表中查看有无主机B的IP地址
2. 如果有，就可以查到对应的硬件地址（Mac地址），再将其硬件地址写入Mac帧，然后通过以太网将数据包发送到目的的主机中
3. 如果本地缓冲表查不到主机B的IP地址。运行ARP协议，进行寻址过程
- （1）在本局域网上广播一个`ARP`请求分组。主要包含的内容有：我是IP 192.168.0.2，我的硬件地址是 00-00-00-AA-BB-CC， 我想知道IP地址为 192.168.0.10 的主机硬件地址
- （2）在本局域网的所有主机都收到ARP请求分组
- （3）主机B在ARP请求分组看到自己的IP地址，就向主机A发送`ARP`响应分组，并写入自己的硬件地址。其余的主机不用响应这个`ARP`请求分组。`ARP`请求分组是广播发送的，但响应的请求是普通的单播。主要内容有：我的IP地址是 192.168.0.10 ，我的硬件地址是 
AC-00-00-00-B1-A2
- （4）主机A收到主机B的`ARP`响应分组后，在`ARP`高速缓冲表中写入主机B的IP地址和硬件地址

`ARP` 协议作用的范围是局域网，如果两个主机是不同的网段，那就需要在路由表中找到下一跳的地址，但其实原理是一样的，也可以看做是本地局域网的扩大版，如下图，本质就是路由器拥有多个IP地址，同时属于不同的局域网当中，不断重复这个过程，通过ARP协议来确定下一跳的地址，直到传输到目的IP主机B。
![](/images/posts/myBlog/2020-01-17-Network-Communication-What-Behind-A-Domain-04.png)


# 四、数据包传输过程总结
以下是我个人的浅薄理解，可能不是特别准确，欢迎指出
- （1）DNS域名解析出目标地址的IP
- （2）使用HTTP来传输，需要在封装成TCP数据报，加上TCP头，IP头（源主机IP和目的IP地址）
- （3) 数据链路层运行ARP协议IP查询下一跳的Mac地址，封装源Mac地址和下一跳的Mac地址到以太帧
- （4）网卡把以太帧转换成二进制流传输
- （5）目标网卡收到二进制流，剥去Mac头，如果不是目的IP地址，查询下一跳Mac地址
- （6）把下一跳地址的Mac地址封装到以太帧，传输到网卡
- （7）重复这个过程，直到接收到以太帧的IP和目的IP地址一致，剥去TCP头，交由上层协议处理
- （8）根据目标端口号，转发到对应的应用程序，程序接收到数据进行对应的处理


# 参考文章
1. [DNS域名解析全过程: https://blog.csdn.net/m0_37812513/article/details/78775629](https://blog.csdn.net/m0_37812513/article/details/78775629)
2. [网络数据包传输过程：https://www.cnblogs.com/songwenlong/p/6103406.html](https://www.cnblogs.com/songwenlong/p/6103406.html)
3. [浅谈通信网络（四）——报文转发：IP/https://www.cnblogs.com/daiaiai/p/9080738.html](https://www.cnblogs.com/daiaiai/p/9080738.html)