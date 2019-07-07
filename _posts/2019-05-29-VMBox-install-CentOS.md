---
layout: post
title: VM Box 安装CentOS7.6和网络设置
tags: 虚拟机    
---

<!-- TOC -->

- [安装Oracle VM VirtualBox](#安装oracle-vm-virtualbox)
- [网络配置](#网络配置)
- [使用XShell连接](#使用xshell连接)

<!-- /TOC -->
### 安装Oracle VM VirtualBox
1. 官网下载安装包，官网地址：[https://www.virtualbox.org/](https://www.virtualbox.org/)
2. 下载CentOs镜像，官网：[https://www.centos.org/](https://www.centos.org/)
    - 建议使用阿里云的镜像地址
    - 建议采用迅雷下载比较快
   ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-01.jpg) 
3. 安装Virtual Box，一路点击next就好了

4. 安装Virtual Box完成以后，新建一个虚拟机
   ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-02.jpg) 
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-03.jpg) 
    注意合理分配磁盘的大小
5. 配置虚拟机
    + 进入设置
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-04.jpg) 

    - 让软驱排在第一位
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-05.jpg)
    
    - 配置启动项，添加下载好的CentOS 镜像
        ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-06.jpg)
    - 点OK保存

6. 启动虚拟机，安装系统
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-07.jpg)    
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-08.jpg)  
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-09.jpg)  
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-10.jpg)
    
    创建root密码和用户，创建完成重启即可
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-11.jpg)


### 网络配置
安装好的虚拟机系统是没办法上网的，因为网络还没有配置完毕
1. 配置虚拟机网卡设置
- 网卡1的设置如下：
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-12.jpg)
- 网卡2设置如下：
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-13.jpg)


2. 连接虚拟机，设置网卡配置
    - 输入命令，编辑图中黄色标注的配置为 yes
    ```java
    vi etc/sysconfig/network-scripts/ifcfg-enp0s3
    ``` 
3. 配置网关
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-14.jpg)


    - 进入本机的cmd 执行 ipconfig ,获取本机的默认网关
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-15.jpg)
   
    - 输入命令，编辑图中黄色标注的配置,UUID不需要一样
    ```java
    vi etc/sysconfig/network-scripts/ifcfg-enp0s8
    ``` 
    ![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-16.jpg)

4. 重启网关
执行命令：systemctl restart network

5. 测试网络情况
    - 虚拟机上： ping www.baidu.com
    - 本机访问虚拟机： ping ping 192.168.0.2

### 使用XShell连接
使用虚拟机连接窗口直接操作有很多的问题，如每次都需要输入密码登录，无法使用复制和粘贴命令，所以我们需要通过XShell来连接，如未安装XShell的朋友请自行安装
1. 启动虚拟机
2. 创建一个新的SSH连接
![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-17.jpg)
3. 输入账号和密码
4. 连接成功
![在这里插入图片描述](/images/posts/myBlog/2019-05-29-VMBox-install-CentOS-18.jpg)


参考文章：
- [https://www.jianshu.com/p/ab97d961f453](https://www.jianshu.com/p/ab97d961f453)
- [https://blog.csdn.net/qq_38669394/article/details/80051356](https://blog.csdn.net/qq_38669394/article/details/80051356)