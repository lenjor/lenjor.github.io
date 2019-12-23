---
layout: post
title: Spring Boot 内置Tomcat启动原理
tags: SpringBoot  
---

### 问题：Spring Boot是如何创建Tomcat的
我们都知道，SpringBoot启动只有一个main方法的入口，只需要运行main方法就能把web项目部署到tomcat容器中，在main方法中，tomcat到底是如何启动的
我们从SpringBoot的main方法一路进去，就会进入到以下的run方法
![](/images/posts/myBlog/2019-12-23-Spring-Boot-Tomcat-01.png)

当Spring的环境和参数初始化设置好以后，创建ConfigurableApplicationContext成功后会调用refreshContext()方法来刷新上下文

下面来看看这个方法做了什么事情
![](/images/posts/myBlog/2019-12-23-Spring-Boot-Tomcat-02.png)

发现里面还是继续调用了 onRefresh() 方法，我们进一步找到它的实现类 ServletWebServerApplicationContext，

![](/images/posts/myBlog/2019-12-23-Spring-Boot-Tomcat-03.png)

发现这里有一行代码：createWebServer(); 

很明显是创建Web服务器的意思，点进去查看
![](/images/posts/myBlog/2019-12-23-Spring-Boot-Tomcat-04.png)

我们来分析一下这一小段代码，如果webServer 和 serverletContext不存在，则从工厂中创建webServer

下面看一下webServer的创建过程，发现该方法有几个实现类，有TomcatServerlet、JettyServerlet、UndertowServlet，这里我们看一下Tomcat的实现
![](/images/posts/myBlog/2019-12-23-Spring-Boot-Tomcat-05.png)

这里我们终于揭开了SpringBoot的内置Tomcat的神秘面纱，Tomcat其实就是通过new出来的一个对象而已，然后给Tomcat进行一些参数的设置，启动Tomcat容器设置监听等待连接


### 内嵌Tomcat的启动
我们回到之前的onRefresh入口，容器刷新后，有一个完成刷新的finishRefresh()操作，这里就是容器启动的入口，如下图
![](/images/posts/myBlog/2019-12-23-Spring-Boot-Tomcat-06.png)

我们发现Tomcat里面也有这个完成刷新的finishRefresh()方法
![](/images/posts/myBlog/2019-12-23-Spring-Boot-Tomcat-06.png)


### 手写main方法创建并启动Tomcat容器
SpringBoot也是通过new来创建Tomcat容器的，那么就意味着，我们可以不使用SpringBoot的run方法来启动web项目，我们自己也可以手写一个main方法来启动web项目，下面我们就来实现一下这个过程。



