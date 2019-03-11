---
layout: post
title: IDEA Spring Boot项目搭建
tags: spring boot   
---

##### 备注
小编使用的是IDEA专业版，社区版和社区版的Spring Boot插件名称有点不太一样，建议各位使用社区版的同学，安装专业版，因为社区版有很多的功能都没有，对于入门可能影响不大，但如果是工作需要，那就有很大的影响了。
#### 项目搭建
1. 新建一个空的项目，在空的项目中新建一个module，选择Spring Initializr 方式创建Spring Boot项目
![Spring Boot项目搭建](https://img-blog.csdnimg.cn/20181124145937332.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8yOTA1MzU2MQ==,size_16,color_FFFFFF,t_70)


2. 填写项目的信息，一般项目的构建使用maven的方式
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181124150358401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8yOTA1MzU2MQ==,size_16,color_FFFFFF,t_70)


3. 选择合适的项目依赖项，一般选择MyBatis，Mysql，Web，按需添加，就算不选择也没关系，后续再pom.xml，自行加入相应依赖即可，不过要主要Spring Boot的版本号，不同版本的SpringBoot有不同特性
![选择依赖项](https://img-blog.csdnimg.cn/20181124150853966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8yOTA1MzU2MQ==,size_16,color_FFFFFF,t_70)

4. 项目结构，这是没有建立项目分层结构的目录，后续可自行添加相应的目录如（controller，service，manager等）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181124151522362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8yOTA1MzU2MQ==,size_16,color_FFFFFF,t_70)

5. 启动Spring Boot项目，进入启动类，点击运行按钮即可启动Spring Boot项目
![启动类](https://img-blog.csdnimg.cn/2018112415212384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8yOTA1MzU2MQ==,size_16,color_FFFFFF,t_70)
**![加粗样式](https://img-blog.csdnimg.cn/20181124152223437.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8yOTA1MzU2MQ==,size_16,color_FFFFFF,t_70)**