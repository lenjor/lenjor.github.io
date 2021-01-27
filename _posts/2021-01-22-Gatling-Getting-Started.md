---
layout: post
title: Galtling基础入门，快速开始
tags: 测试，Gatling  

---

<!-- TOC -->

- [一、准备工作](#一准备工作)
- [二、安装Scala插件](#二安装scala插件)
- [三、下载Gatling包](#三下载gatling包)
- [四、运行官方的Demo测试用例](#四运行官方的demo测试用例)

<!-- /TOC -->
# 一、准备工作

```
（1）Java JDK
（2）IntelliJ IDEA (网上教程很多，不再赘述)
```

# 二、安装Scala插件

打开 IDEA ，点击【IntelliJ IDEA】 -> 【Preferences】 -> 【Plugins】，搜索 “Scala”，搜索到插件安装重启即可。

![](/images/posts/myBlog/2021-01-22-Gatling-Getting-Started-01.png)



# 三、下载Gatling包

下载地址：[https://gatling.io/open-source/start-testing/](https://gatling.io/open-source/start-testing/)

找到下载按钮，直接下载解压即可使用

```
解压后目录结构：
├── bin		目录下有2个脚本，gatling和recorder， gatling用来运行测试， recorder用来启动录制脚本的UI的（不推荐使用）
├── conf		Gatling自身的一些配置。
├── lib		Gatling自身依赖的库文件
├── results		存放测试报告
├── target		你启动运行组件后，gatling会为你编译好所有的.scala脚本，而编译后的class文件就会在这里
└── user-files	脚本存放位置 user-files/simulations ，默认下载好的包会有几个官方的示例测试Demo


当运行gating脚本的时候，其会扫描user-files目录下的所有文件，列出其中所有的Simulation(一个测试类，里面可以包含任意多个测试场景)。选择其中一个Simulation，然后填写Simulation ID和运行描述，这个都是为报告描述服务的
```

# 四、运行官方的Demo测试用例

控制台终端进入解压的bin目录，然后运行 gatling.sh 脚本，我是直接使用IDEA打开了下载好的Gatling项目，在IDEA终端进行的操作为例

```
# 执行gatling.sh
lenjor@coding bin % ./gatling.sh
```

Gatling 会遍历`user-files/simulations`，列出所有的Simulation

```
GATLING_HOME is set to /Users/lenjor/file/project/myProject/gatling/gatling-charts-highcharts-bundle-3.5.0
Choose a simulation number:
     [0] computerdatabase.BasicSimulation
     [1] computerdatabase.advanced.AdvancedSimulationStep01
     [2] computerdatabase.advanced.AdvancedSimulationStep02
     [3] computerdatabase.advanced.AdvancedSimulationStep03
     [4] computerdatabase.advanced.AdvancedSimulationStep04
     [5] computerdatabase.advanced.AdvancedSimulationStep05
1			# 这个是我选择运行的测试用例ID
Select run description (optional)
测试Demo01		# 填入测试用例的描述，可以直接回车跳过
```

这里我们在终端中输入 `1`，代表选择`AdvancedSimulationStep01`执行，
之后按提示输入内容或回车跳过，就可以开始执行了，执行完成会在`results`目录下生成网页报告。

```
Reports generated in 0s.
Please open the following file: /Users/lenjor/file/project/myProject/gatling/gatling-charts-highcharts-bundle-3.5.0/results/advancedsimulationstep01-20210123021823428/index.html
```

直接复制最后的输出的链接地址（或者自己去`result`目录找到报告的html文件）打开就能看到非常详细的测试报告

![](/images/posts/myBlog/2021-01-22-Gatling-Getting-Started-02.png)


最后，直接在IDEA打开解压好的Gatling项目，在user-files 目录下新建自己的测试脚本，即可完成自己想要的测试内容