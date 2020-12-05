---
layout: post
title: Linux安装JDK，并配置多个JDK切换
tags: java  

---


# 方法一（不推荐）：直接在Linux控制台使用命令获取可以安装的JDK版本进行安装
安装的方法请查看：[Centos中yum方式安装java
](https://www.cnblogs.com/kevingrace/p/5870814.html)

```
# 查看可以安装的JDK
yum -y list java*

# 安装对应的JDK版本
yum -y install java-1.8.0-openjdk【填对应的版本的名称】

```

# 方法二（推荐）：在Oracle下载对应版本的JDK（这里以二进制的解压版安装为例）
1. 进入Oracle官网：[https://www.oracle.com/
](https://www.oracle.com/)


2. 拉到底部选择Developer, 然后就能在Download选项中找到我们想要的Java SE的下载入口，进去就能看到有download的选项，一路看到Download就进去，最后会进入到版本的选择页面
![](/images/posts/myBlog/2020-12-05-Linux-Java-JDK-install-01.png)
![](/images/posts/myBlog/2020-12-05-Linux-Java-JDK-install-02.png)
![](/images/posts/myBlog/2020-12-05-Linux-Java-JDK-install-03.png)

3. 选择对应的文件，这里我们选择的是x64的二进制解压版，这个版本的解压就可以使用了，我的是intel的CPU，intel是基于x86_x64架构的，所以我们选择x64的文件，记住是选择 tar.gz 的文件格式的
![](/images/posts/myBlog/2020-12-05-Linux-Java-JDK-install-04.png)

4. 使用FTP工具把文件上传到Linux的机器上，我采用的国人开发的免费SSH连接软件FinalShell，自带就有文件管理工具，用这个软件操作文件比较简单方便，可以免去命令的输入，直接拖拉就能实现文件上传，在把文件上传到自己想要安装的目录，我这里采用安装的目录是:  /home/lenjor/java/

5. 解压安装， 环境配置

```
# 进入压缩文件的目录（也是需要安装的目录）
    cd /home/lenjor/java/

# 解压下载的JDK文件
    tar -zxvf jdk-8u271-linux-x64.tar.gz

# 开始配置Java环境变量，取得管理员权限，输入 su ，输入管理员密码切换管理员
    su

# 修改环境配置文件
    vi /etc/profile

# 在配置文件的最末尾加上下面这段配置，路径替换为自己实际安装的路径

    # SET JAVA HOME
    JAVA_HOME=/home/lenjor/java/jdk1.8.0_271
    JRE_HOME=/home/lenjor/java/jdk1.8.0_271/jre
    CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    export JAVA_HOME JRE_HOME CLASS_PATH PATH

# 让配置文件生效
    source /etc/profile
```

6. 安装结果验证 `java -version` ，出现Java的版本信息，就证明安装成功

# 多个版本的JDK配置
1. 和上述的方法二一样，先下载对应的JDK，然后解压到安装的目录，以我个人的安装为例，我就是先操作完方法二全部的步骤，然后安装JDK 15
2. 解压版解压出来就是安装成功了，环境变量之前已经配置过了，我们需要做的，就是默认JDK环境的操作而已
```
    # 用户alternatives命令把两个JDK的目录加入JDK选择列表
    alternatives --install /usr/bin/java java /home/lenjor/java/jdk1.8.0_271/bin/java 1
    alternatives --install /usr/bin/java java /home/lenjor/java/jdk-15.0.1/bin/java 2

    # 然后输入下面这个命令就能进入选择JDK的选项
    alternatives --config java
```
    ![](/images/posts/myBlog/2020-12-05-Linux-Java-JDK-install-05.png)

3. 输入 `java -version` 就能切换对应的JDK了，不过我测试发现有时候再次输入这个命令还是没有切换成功，但是重新连接和新开的Shell链接查看Java版本却切换成功了, 这时候需要重新编辑打开一下 `/etc/profile` 文件，再次 `source /etc/profile` 应该就可以了， 下面是我切换的效果图
![](/images/posts/myBlog/2020-12-05-Linux-Java-JDK-install-06.png)



参考文章：
1. [Centos中yum方式安装java
](https://www.cnblogs.com/kevingrace/p/5870814.html)
2. [Linux jdk安装多个版本并进行切换
](https://www.cnblogs.com/xu-dong/p/6429301.html)