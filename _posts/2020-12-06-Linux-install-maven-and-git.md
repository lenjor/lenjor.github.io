---
layout: post
title: Linux安装Maven和Git
tags: java  

---

<!-- TOC -->

- [一、Maven的安装](#一maven的安装)
    - [方法一：直接输入 `mvn -v` 命令， 直接系统安装法，如下图，直接按提示输入y，就能直接安装成功](#方法一直接输入-mvn--v-命令-直接系统安装法如下图直接按提示输入y就能直接安装成功)
    - [方法二：解压包安装法](#方法二解压包安装法)
        - [1、maven包的下载有两种方法](#1maven包的下载有两种方法)
        - [2、解压到对应的目录，我安装的目录是：`/usr/local/maven3`。](#2解压到对应的目录我安装的目录是usrlocalmaven3)
        - [3、配置maven环境变量](#3配置maven环境变量)
        - [4、查看安装结果， `mvn -v` 有出现安装的maven信息证明安装成功](#4查看安装结果-mvn--v-有出现安装的maven信息证明安装成功)
- [二、Git安装](#二git安装)
- [参考文章](#参考文章)

<!-- /TOC -->
# 一、Maven的安装
## 方法一：直接输入 `mvn -v` 命令， 直接系统安装法，如下图，直接按提示输入y，就能直接安装成功
![](/images/posts/myBlog/2020-12-06-Linux-install-maven-and-git-01.png)
## 方法二：解压包安装法
### 1、maven包的下载有两种方法

(1) 可以直接通过 wget命令直接下载，进入想要保存到的目录，然后执行wget命令
```properties
    wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```
![](/images/posts/myBlog/2020-12-06-Linux-install-maven-and-git-02.png)
(2)在Maven的官网下载
官网地址：[https://maven.apache.org/download.cgi
](https://maven.apache.org/download.cgi)

### 2、解压到对应的目录，我安装的目录是：`/usr/local/maven3`。

```properties
# 创建安装的目录（自己想装哪里装哪里）
mkdir /usr/local/maven3
# 进入下载的maven包的目录，解压
tar zxf apache-maven-3.6.3-bin.tar.gz
# 把解压出来的目录移动到我想要的安装的目录
mv apache-maven-3.6.3 /usr/local/maven3/

```
### 3、配置maven环境变量

```properties
# 打开环境变量文件
vi /etc/profile

# 添加下面这段配置，安装路径记得替换成自己的实际安装路径
export M2_HOME=/usr/local/maven3/apache-maven-3.6.3
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin

# 保存退出，刷新文件挂载
source /etc/profile
```
### 4、查看安装结果， `mvn -v` 有出现安装的maven信息证明安装成功
![](/images/posts/myBlog/2020-12-06-Linux-install-maven-and-git-03.png)


# 二、Git安装
Git安装同样也有两种方式，我采用的是最简单的`yum`命令安装的形式，只需要输入这个命令就可以安装成功
```
# 安装命令
yum -y install git
# 查看安装结果
git --version
```
![](/images/posts/myBlog/2020-12-06-Linux-install-maven-and-git-04.png)

如果想要自己从Git官网下载Git安装包，自己配置，可以参考这这篇文章
[在Linux系统上安装Git](https://www.cnblogs.com/wulixia/p/11016684.html)


# 参考文章
- [CentOS安装Maven](https://www.cnblogs.com/bincoding/p/6156236.html)
- [在Linux系统上安装Git](https://www.cnblogs.com/wulixia/p/11016684.html)

