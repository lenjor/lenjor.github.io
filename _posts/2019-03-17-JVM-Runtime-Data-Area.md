---
layout: post
title: JVM运行时数据区(JVM Runtime Data Area)
tags: 设计模式   
---


<!-- TOC -->

- [JVM运行时数据区组成](#jvm运行时数据区组成)
- [程序计数器(Program Counter Register)](#程序计数器program-counter-register)
- [虚拟机栈(VM Stack)](#虚拟机栈vm-stack)
    - [在讲述栈帧具体的构成前需要一个辅助程序](#在讲述栈帧具体的构成前需要一个辅助程序)
    - [局部标量表](#局部标量表)
    - [操作数栈](#操作数栈)
    - [动态链接](#动态链接)
    - [出口（返回值）](#出口返回值)
- [本地方法栈(Native Method Stack)](#本地方法栈native-method-stack)
- [方法区（Method Area）](#方法区method-area)
- [堆（Heap）](#堆heap)
- [线程运行内存模型图](#线程运行内存模型图)

<!-- /TOC -->
### JVM运行时数据区组成
Java虚拟机在执行Java程序的过程中会将其管理的内存划分为若干个不同的数据区域，这些区域有各自的用途、创建和销毁的时间，有些区域随虚拟机进程的启动而存在，有些区域则是依赖用户线程的启动和结束来建立和销毁。Java虚拟机所管理的内存包括以下几个运行时数据区域，如图：
![JVM运行时数据区](/images\posts\myBlog\2019-03-17-JVM-Runtime-Data-Area-01.png)
1. 程序计数器(Program Counter Register)
2. 虚拟机栈(VM Stack)
3. 本地方法栈(Native Method Stack)
4. 方法区（Method Area）
5. 堆（Heap）

### 程序计数器(Program Counter Register)
**作用**：指向当前线程正在执行的字节码指令地址（行号 ），线程私有

**引导思考：**
- 在Java中最小的执行单位是什么? --》**线程**
- 多线程中，线程切换是如何知道上一个时间片执行到的程序指令的? --》**程序计数器**
- 程序计数器标志着线程正在执行的字节码指令地址，那么每个线程的执行状态是否一致？--》该程序计数器是否线程私有--》 **程序计数器线程私有**

###  虚拟机栈(VM Stack)
**作用**：虚拟机栈是当前线程执行方法的内存模型。每个方法被执行的时候，都会创建一个栈帧，把栈帧压人栈，当方法正常返回或者抛出未捕获的异常时，栈帧就会出栈。

**引导思考**：
- 虚拟机栈是什么类型的数据结构？--》**栈**
- 栈这种数据结构有什么特点？--》**先进后出（FILO）**
- 一个方法包含了那些内容？ --》**数据、指令、返回地址**
- 线程执行一个方法，需要把这个方法压入栈中，压入栈的方法的数据结构是什么？ --》**栈帧**
- 栈帧是不是装载了一个方法，需要包含那些内容？ --》 **局部标量表，操作数栈，动态链接、出口（返回值）**、附加信息（不强制要求附带，一般忽略）
- 每个线程执行的方法时要压入栈帧到虚拟机栈，每个线程执行方法是否一样？--》 虚拟机栈是否线程私有？ --》 **虚拟机栈线程私有**
- 线程每执行一个方法就要压入一个装载需要执行的方法的栈帧，方法嵌套调用方法如何执行 --》**把方法中嵌套调用的方法栈帧压入栈顶**
- 递归调用是否需要重复压栈帧入栈--》**线程每执行一个方法就要压入一个装载需要执行的方法的栈帧**--》是
- 是否能无限嵌套递归？--》虚拟机栈是否有大小限制？--》栈都有大小限制--》不能
- 当虚拟机栈大小不够用的会怎么样 --》 **抛出栈内存溢出异常 StackOverflowError**

#### 在讲述栈帧具体的构成前需要一个辅助程序
```java
public class JVM {
    //成员变量
    private Object obj = new Object();
    private int sss = 0;

    //局部变量
    public void methodOne(int i){
        int j = 0;
        int sum = i + j;
        Object acb = obj;
        long start = System.currentTimeMillis();
        methodTwo();
        return;
    }

    public void methodTwo(){
        File file = new File("");
    }
}
```

反编译出来的结果如下
```java
public class learn.com.JVM {
  public learn.com.JVM();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: new           #2                  // class java/lang/Object
       8: dup
       9: invokespecial #1                  // Method java/lang/Object."<init>":()V
      12: putfield      #3                  // Field obj:Ljava/lang/Object;
      15: aload_0
      16: iconst_0
      17: putfield      #4                  // Field sss:I
      20: return

  public void methodOne(int);
    Code:
       0: iconst_0
       1: istore_2
       2: iload_1
       3: iload_2
       4: iadd
       5: istore_3
       6: aload_0
       7: getfield      #3                  // Field obj:Ljava/lang/Object;
      10: astore        4
      12: invokestatic  #5                  // Method java/lang/System.currentTimeMillis:()J
      15: lstore        5
      17: aload_0
      18: invokevirtual #6                  // Method methodTwo:()V
      21: return

  public void methodTwo();
    Code:
       0: new           #7                  // class java/io/File
       3: dup
       4: ldc           #8                  // String
       6: invokespecial #9                  // Method java/io/File."<init>":(Ljava/lang/String;)V
       9: astore_1
      10: return
}
```
javap指令集：下面显示只是抽取的部分指令，查看更全的指令：[javap指令集](https://www.cnblogs.com/JsonShare/p/8798735.html)
```java
栈和局部变量操作
将常量压入栈的指令
aconst_null 将null对象引用压入栈
iconst_m1 将int类型常量-1压入栈
iconst_0 将int类型常量0压入栈
iconst_1 将int类型常量1压入栈
iconst_2 将int类型常量2压入栈
iconst_3 将int类型常量3压入栈
iconst_4 将int类型常量4压入栈
iconst_5 将int类型常量5压入栈
astore 将将引用类型或returnAddress类型值存入局部变量
istore_0 将int类型值存入局部变量0
istore_1 将int类型值存入局部变量1
istore_2 将int类型值存入局部变量2
istore_3 将int类型值存入局部变量3
iload_0 从局部变量0中装载int类型值
iload_1 从局部变量1中装载int类型值
iload_2 从局部变量2中装载int类型值
iload_3 从局部变量3中装载int类型值
iadd 执行int类型的加法
```


#### 局部标量表
**定义**：局部标量表是一组变量值的存储空间，用于存放 **方法参数** 和 **局部变量**。局部变量数组所需要的空间在编译期间完成分配，在方法运行期间不会改变局部变量数组的大小。**变量槽** （Variable Slot）是局部变量表的最小单位，没有强制规定大小为 32 位。

**引导思考**：
- 线程执行过程中的变量存储在哪里？--》局部变量表
- 局部变量表的数据长度是多少位？ --》 32位 --》寻址空间是多大？ --》 2^32 = 4G
- 我们从代码中看到，方法methodOne中，第一行代码

  		j = 0;

	被编译成字节码指令

		0: iconst_0       //将int类型常量0压入栈
    	1: istore_2       //将int类型值存入局部变量2

- 为什么我们的 局部变量 j 在局部变量表中的位置是2 ？--》成员函数是否有一个隐式的引用 this，方法有形参？---》this引用存入了成员变量表0的位置，形参存入了1的位置--》j 只能存入为2的位置
 
#### 操作数栈
操作变量的内存模型。操作数栈的最大深度在编译的时候已经确定（写入方法区code属性的max_stacks项中）。操作数栈的的元素可以是任意Java类型，包括long和double，方法刚开始执行的时候，栈是空的，当方法执行过程中，各种字节码指令往栈中存取数据。

程序代码

	int sum = i + j;
	
编译成字节码：

	 2: iload_1		//从局部变量1中装载int类型值
     3: iload_2		//从局部变量2中装载int类型值
     4: iadd		//iadd 执行int类型的加法
     5: istore_3	//将int类型值存入局部变量3

**引导思考**：
- 线程运行时数据计算的中间过程中，数据是如何存储的？ --》存储在操作数栈中（32位）
- double类型占用多少个栈单位空间 --》double是64位，栈是32位 --》double占用2个栈单位空间
- 在程序计算操作的中间值存储在哪里？ --》 操作数栈，计算完成后写回局部变量表

#### 动态链接
每个栈帧都包含一个执行运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接（Dynamic Linking）。

Class 文件中存放了大量的符号引用，字节码中的方法调用指令就是以常量池中指向方法的符号引用作为参数。这些符号引用一部分会在类加载阶段或第一次使用时转化为直接引用，这种转化称为静态解析。另一部分将在每一次运行期间转化为直接引用，这部分称为动态连接。

**引导思考**：
- 方法内除了基本的类型，还有引用类型（多态），引用类型（多态）如何存储记录 --》 动态链接，运行期间指向引用的对象/方法地址

#### 出口（返回值）
如果有返回值的话，压入调用者栈帧中的操作数栈中，并且把PC的值指向 方法调用指令 后面的一条指令地址。

**引导思考**：
- 程序什么时候会到达出口？ --》
		**当执行遇到返回指令时**，会将返回值传递给上层的方法调用者，这种退出的方式称为正常完成出口（Normal Method Invocation 	Completion），一般来说，调用者的PC计数器可以作为返回地址。
		**当执行遇到异常时**，并且当前方法体内没有得到处理，就会导致方法退出，此时是没有返回值的，称为异常完成出口（Abrupt Method Invocation Completion），返回地址要通过异常处理器表来确定。
- 当程序返回时，有哪些可能的操作？ --》
	1. 恢复上层方法的局部变量表和操作数栈
	2. 把返回值压入调用者调用者栈帧的操作数栈
	3. 调整 PC 计数器的值以指向方法调用指令后面的一条指令


### 本地方法栈(Native Method Stack)
与VM Strack相似，VM Strack为JVM提供执行JAVA方法的服务，Native Method Stack则为JVM提供使用native 方法的服务。
（1）调用本地native的内存模型
（2）线程独享。

### 方法区（Method Area）
**Object Class Data**(加载类的类定义数据) 是存储在方法区的。存储已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据。
　　垃圾回收在这个区域会比较少出现，这个区域内存回收的目的主要针对常量池的回收和类的卸载。
（1）线程共享的
（2）运行时常量池：

	A、是方法区的一部分
	B、存放编译期生成的各种字面量和符号引用
	C、Class文件中除了存有类的版本、字段、方法、接口等描述信息，还有一项是常量池，存有这个类的 编译期生成的各种字面量和符号引用，这部分内容将在类加载后，存放到方法区的运行时常量池中。


### 堆（Heap）
　Heap（堆）是JVM的内存数据区。

	1. Java堆是虚拟机管理的内存中最大的一块
	2. Java堆是所有线程共享的区域
	3. 在虚拟机启动时创建
	4. 此内存区域的唯一目的就是存放对象实例，几乎所有对象实例都在这里分配内存。实际上也只是保存对象实例的属性值，属性的类型和对象本身的类型标记等，并不保存对象的方法（以帧栈的形式保存在Stack中）
	5. 对象实例在Heap 中分配好以后，需要在Stack中保存一个4字节的Heap 内存地址（引用地址）
	6. Java堆是垃圾收集器管理的内存区域，因此很多时候称为“GC堆”
	7. java堆处于物理不连续的内存空间中，只要逻辑上连续即可。

### 线程运行内存模型图
![在这里插入图片描述](/images\posts\myBlog\2019-03-17-JVM-Runtime-Data-Area-02.jpg)
