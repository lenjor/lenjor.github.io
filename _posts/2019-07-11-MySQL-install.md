---
layout: post
title: MySQL解压版安装
tags: MySQL
---
<!-- TOC -->

- [MySQL下载](#mysql下载)
    - [1. 选择社区版](#1-选择社区版)
    - [2. 选择对应的版本和系统，点击下载](#2-选择对应的版本和系统点击下载)
    - [3. 可以不登录直接下载，解压](#3-可以不登录直接下载解压)
- [配置MySQL默认配置文件](#配置mysql默认配置文件)
    - [1. 在解压完的目录下，新建一个 <font color="red"> my.ini </font> 配置文件,把以下配置保存到 **my.ini** 文件中](#1-在解压完的目录下新建一个-font-colorred-myini-font-配置文件把以下配置保存到-myini-文件中)
    - [2. 修改配置文件中的 <font color="red"> 数据库路径</font> 和 <font color="red">  数据保存路径</font> 成自己的解压目录和想要保存的数据目录（**数据保存目录不用创建，后面初始化数据库时会自动创建**）](#2-修改配置文件中的-font-colorred-数据库路径font-和-font-colorred--数据保存路径font-成自己的解压目录和想要保存的数据目录数据保存目录不用创建后面初始化数据库时会自动创建)
- [初始化MySQL数据库](#初始化mysql数据库)
    - [1. 进入解压后的bin目录，管理员权限打开控制台窗口](#1-进入解压后的bin目录管理员权限打开控制台窗口)
    - [2. 运行初始化命令](#2-运行初始化命令)
    - [3. bin目录下执行](#3-bin目录下执行)
    - [4. 启动mysql](#4-启动mysql)
- [配置MySQL密码和权限](#配置mysql密码和权限)
    - [1. 另外打开一个命令行，执行，用root登陆，是因为刚开始没有密码的](#1-另外打开一个命令行执行用root登陆是因为刚开始没有密码的)
    - [2. 执行下面两条命令，设置密码](#2-执行下面两条命令设置密码)
    - [3. 设置连接权限，没有设置连接权限，使用Navicat连接时会报错](#3-设置连接权限没有设置连接权限使用navicat连接时会报错)
- [问题总结](#问题总结)

<!-- /TOC -->

### MySQL下载
下载地址：[https://dev.mysql.com/downloads/](https://dev.mysql.com/downloads/)
#### 1. 选择社区版
![](/images/posts/myBlog/2019-07-11-MySQL-install-01.jpg)
#### 2. 选择对应的版本和系统，点击下载
![](/images/posts/myBlog/2019-07-11-MySQL-install-02.jpg)
#### 3. 可以不登录直接下载，解压
![](/images/posts/myBlog/2019-07-11-MySQL-install-03.jpg)

### 配置MySQL默认配置文件
#### 1. 在解压完的目录下，新建一个 <font color="red"> my.ini </font> 配置文件,把以下配置保存到 **my.ini** 文件中

```java
# 数据库服务端配置项
[mysqld]
# 数据库路径
basedir=F:\\Mysql\\mysql-8.0.16-winx64
# 数据保存路径
datadir=F:\\Mysql\\mysql-8.0.16-winx64\\data
# 端口号
port=3306
# 默认字符集
character-set-server=utf8mb4
# 存储引擎
default-storage-engine=INNODB

# 客户端配置项
[mysql]
# 默认字符集
default-character-set=utf8mb4

# 连接客户端配置项
[client]
default-character-set=utf8mb4
```
#### 2. 修改配置文件中的 <font color="red"> 数据库路径</font> 和 <font color="red">  数据保存路径</font> 成自己的解压目录和想要保存的数据目录（**数据保存目录不用创建，后面初始化数据库时会自动创建**）
![](/images/posts/myBlog/2019-07-11-MySQL-install-04.jpg)

### 初始化MySQL数据库
#### 1. 进入解压后的bin目录，管理员权限打开控制台窗口
ps：如果是win10的朋友，可以在左下角，右键点开菜单
![](/images/posts/myBlog/2019-07-11-MySQL-install-05.jpg)
使用cd命令进入bin目录，如：

    cd F:\Mysql\mysql-8.0.16-winx64\bin

#### 2. 运行初始化命令

    mysqld --initialize-insecure

会发现程序在你配置在my.ini配置文件中配置的数据保存路径自动创建了data文件夹以及相关的文件

#### 3. bin目录下执行
    mysqld -install
    提示：Service successfully installed

#### 4. 启动mysql
    net start mysql  

### 配置MySQL密码和权限
#### 1. 另外打开一个命令行，执行，用root登陆，是因为刚开始没有密码的

    mysql -u root mysql
#### 2. 执行下面两条命令，设置密码

    update mysql.user set authentication_string=password('123456') where user='root' and Host = 'localhost'; 

    flush privileges;    

#### 3. 设置连接权限，没有设置连接权限，使用Navicat连接时会报错
![](/images/posts/myBlog/2019-07-11-MySQL-install-06.jpg)  
<br>

    alter user 'root'@'localhost' identified with mysql_native_password by '123456';   

授权语法说明：

    grant all privileges on *.* to root@”xxx.xxx.xxx.xxx” identified by “密码”;（xxx.xxx.xxx.xxx用%也行，表示所有IP）
    或者 ​GRANT ALL PRIVILEGES ON *.* TO ‘root’@’xxx.xxx.xxx.xxx’ IDENTIFIED BY ‘123456’ WITH GRANT OPTION;
    这相当于是给IP-xxx.xxx.xxx.xxx赋予了所有的权限，包括远程访问权限。
    然后刷新权限
    flush privileges; 

### 问题总结
完成了上面的操作，就可以使用Navicat和程序连接MySQL了，安装过程如果遇到以下问题，可以在参考这里的连接
1. [初始化时 Failed to find valid data directory 错误](https://www.jianshu.com/p/916f9b111b23)
2. [net start mysql 命令  MySQL 服务无法启动](https://www.jianshu.com/p/916f9b111b23)
3. [Navicat无法连接到MySQL](https://blog.csdn.net/qq_39206238/article/details/80351803)





