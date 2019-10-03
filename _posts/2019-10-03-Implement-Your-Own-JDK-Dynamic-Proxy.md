---
layout: post
title: 手写JDK动态代理（Implememt your own JDK dynamic proxy)
tags: java   
---
<!-- TOC -->

- [手写自己的动态代理需要重写那些内容](#手写自己的动态代理需要重写那些内容)
- [动态生成的 $Proxy0 对象长什么样？](#动态生成的-proxy0-对象长什么样)
    - [反编译$Proxy.class代码分析](#反编译proxyclass代码分析)
- [实现动态代理的关键](#实现动态代理的关键)
    - [实现完整代码](#实现完整代码)
    - [自定义代理类实现](#自定义代理类实现)
    - [测试方法](#测试方法)
- [运行结果](#运行结果)
- [总结](#总结)

<!-- /TOC -->

### 手写自己的动态代理需要重写那些内容
首先我们来看看，上一篇文章[代理模式(Proxy pattern)](https://lenjor.github.io/2019/10/Design-Patterns-Proxy/)的动态代理类中，使用到了那些内容，从中找出需要重新实现的类和方法。

![](/images/posts/myBlog/2019-10-03-Implement-Your-Own-JDK-Dynamic-Proxy-01.png)

### 动态生成的 $Proxy0 对象长什么样？
把上一篇文章的动态代理的测试类，改造一下，从JDK，保存成文件，我们来看看 $Proxy0 对象长什么样。
``` java
/**
 * JDK实现的动态代理测试类
 */
public class JdkProxyTest {
    public static void main(String[] args) {
        Customer customer = new Customer("陈大春");

        Object proxyObj = new JdkHouseProxy().getInstance(customer);
        Rent rent = (Rent) proxyObj;

        rent.rent("独栋别野小洋楼，400平，室内泳池");
        rent.buy(4000000)
        
        //从JDK的代理生成器中，获取代理类 Proxy0 ，保存到磁盘中
        try {
            byte[] bytes = ProxyGenerator.generateProxyClass("$Proxy0", new Class[]{Rent.class});
            FileOutputStream os = new FileOutputStream("D://$Proxy0.class");
            os.write(bytes);
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

运行程序以后，在D盘下，会生成了一个 $Proxy.class 文件，把改文件拖入IDEA自动反编译，反编译后的代码如下：

``` java

import com.lenjor.jdkproxy.Rent;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements Rent {
    private static Method m1;
    private static Method m3;
    private static Method m4;
    private static Method m2;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String buy(int var1) throws  {
        try {
            return (String)super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String rent(String var1) throws  {
        try {
            return (String)super.h.invoke(this, m4, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("com.lenjor.jdkproxy.Rent").getMethod("buy", Integer.TYPE);
            m4 = Class.forName("com.lenjor.jdkproxy.Rent").getMethod("rent", Class.forName("java.lang.String"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

```

#### 反编译$Proxy.class代码分析
![](/images/posts/myBlog/2019-10-03-Implement-Your-Own-JDK-Dynamic-Proxy-02.png)
![](/images/posts/myBlog/2019-10-03-Implement-Your-Own-JDK-Dynamic-Proxy-03.png)
从上面的两张图中，我们就可以知道，$Proxy0 类继承自Proxy类，是JVM动态生成的类被final修饰。同样实现了被代理类的接口，当调用代理的方法时，执行的是
``` java
 public final String buy(int var1) throws  {
        try {
            return (String)super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }
```
请注意这里的InvokecationHandler,实际上是从哪里来的？
![](/images/posts/myBlog/2019-10-03-Implement-Your-Own-JDK-Dynamic-Proxy-03.png)

看到这里，你是不是豁然开朗，原来代理方法是这样执行的。那么下面我们就开始进入本文的重点，自己实现一个JDK动态代理。


### 实现动态代理的关键
关键在于获取动态代理的方法
    public static Object newProxyInstance(MyClassLoader loader, Class<?>[] interfaces, MyInvocationHandler h)
            throws IllegalArgumentException{}

    1. 动态生成源代码.java文件
    2. Java文件输出磁盘
    3. 把生成的.java文件编译成.class文件
    4. 编译生成的.class文件加载到JVM中
    5. 返回字节码重组以后的新的代理对象

#### 实现完整代码
``` java
/**
 * 租房,买房
 */
public interface Rent {
    public String rent(String need);
    public int buy(int money);
    public void sale(int money);
}



/**
 * 房客
 */
public class Customer implements Rent{

    private String name;

    public Customer(String name){
        this.name = name;
    }

    @Override
    public String rent(String need) {
        System.out.println(name + "要租的房子的需求是：" + need);
        return name + "'s Tenant.rent()";
    }

    @Override
    public int buy(int money) {
        System.out.println(name + "购买房子出价 ：" + money);
//        return name + "'s Tenant.buy()";
        return money;
    }

    @Override
    public void sale(int money) {
        System.out.println(name + "卖出房子出价 ：" + money);
    }

}



/**
 * 自定义类加载器
 */
public class MyClassLoader extends ClassLoader {
    private File classPathfile;

    public MyClassLoader() {
        String classpth = MyClassLoader.class.getResource("").getPath();
        classPathfile = new File(classpth);
    }

    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {
        String className = MyClassLoader.class.getPackage().getName() + "." + name;
        if (classPathfile != null) {
            File file = new File(classPathfile, name + ".class");
            FileInputStream fileInputStream = null;
            ByteArrayOutputStream outputStream = null;
            try {
                fileInputStream = new FileInputStream(file);
                outputStream = new ByteArrayOutputStream();
                byte[] buff = new byte[1024];
                int len;
                while ((len = fileInputStream.read(buff)) != -1) {
                    outputStream.write(buff, 0, len);
                }
                return defineClass(className, outputStream.toByteArray(), 0, outputStream.size());
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (null != fileInputStream) {
                    try {
                        fileInputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (null != outputStream) {
                    try {
                        outputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        return null;
    }
}



/**
 * 自定义InvocationHandler接口
 */
public interface MyInvocationHandler {

    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable;

}



/**
 * 代理类
 */
public class MyHouseProxy implements MyInvocationHandler{
    // 目标对象
    private Object target;

    /**
     * 绑定关系，也就是关联到哪个接口（与具体的实现类绑定）的哪些方法将被调用时，执行invoke方法。
     */
    public Object getInstance(Object target) {
        this.target = target;

        Class clazz = target.getClass();
        //使用自己定义的ClassLoader和InvokecationHandler
        return MyProxy.newProxyInstance(new MyClassLoader(), clazz.getInterfaces(), this);
    }


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (args != null) {
            for (int i = 0; i < args.length; i++) {
                System.out.println("arg" + i + "=" + args[i]);
            }
        }
        System.out.println("我是中间商----开始干活");
        Object invokeResult = method.invoke(target, args);
        System.out.println("中间商干活完毕，收取差价");
        System.out.println("invokeResult =" + invokeResult + "\n");
        return invokeResult;
    }
}


```

#### 自定义代理类实现
``` java

/**
 * 自定义代理类实现
 */
public class MyProxy {
    public static final String ln = "\r\n";

    /**
     * 获取代理对象方法
     *
     * @param loader
     * @param interfaces
     * @param h
     * @return
     * @throws IllegalArgumentException
     */
    public static Object newProxyInstance(MyClassLoader loader, Class<?>[] interfaces, MyInvocationHandler h)
            throws IllegalArgumentException {
        try {
            // 1 java源碼
            String src = generateSrc(interfaces);
            // 2 將源码输出到java文件中
            String filePath = MyProxy.class.getResource("").getPath();
            System.out.println("生成的.java文件目录:" + filePath);
            File f = new File(filePath + "$Proxy0.java");
            FileWriter fw = new FileWriter(f);
            fw.write(src);
            fw.flush();
            fw.close();

            //3、将java文件编译成class文件
            JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
            StandardJavaFileManager manage = compiler.getStandardFileManager(null, null, null);
            Iterable iterable = manage.getJavaFileObjects(f);

            JavaCompiler.CompilationTask task = compiler.getTask(null, manage, null, null, null, iterable);
            task.call();
            manage.close();

            //4、将class加载进jvm
            Class proxyClass = loader.findClass("$Proxy0");
            //TODO 删除动态生成的.java文件
            f.delete();

            //5、返回代理类对象
            Constructor constructor = proxyClass.getConstructor(MyInvocationHandler.class);
            return constructor.newInstance(h);

        } catch (IOException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 生成.Java文件源代码
     *
     * @param interfaces
     * @return
     */
    private static String generateSrc(Class<?>[] interfaces) {
        StringBuffer sb = new StringBuffer();
        sb.append("package com.lenjor.myproxy;" + ln);
        sb.append("import java.lang.reflect.Method;" + ln);
        sb.append("public class $Proxy0 implements " + interfaces[0].getName() + "{" + ln);
        sb.append("private MyInvocationHandler h;" + ln);
        sb.append("public $Proxy0(MyInvocationHandler h) {" + ln);
        sb.append("this.h = h;" + ln);
        sb.append("}");
        //遍历接口定义的方法，实现接口所有的方法
        for (Method m : interfaces[0].getMethods()) {
            sb.append("public " + m.getReturnType().getName() + " " + m.getName() + "(");
            //方法的参数
            for (int i = 0; i < m.getParameterTypes().length; i++) {
                sb.append(m.getParameterTypes()[i].getName() + " " + m.getParameters()[i].getName());
                if (i != m.getParameterTypes().length - 1) {
                    sb.append(",");
                }
            }
            sb.append(") {" + ln);
            sb.append("try { " + ln);
            sb.append("Method m = " + interfaces[0].getName()
                    + ".class.getMethod(\"" + m.getName()
                    + "\",new Class[]{");
            //方法的参数列表
            for (int i = 0; i < m.getParameterTypes().length; i++) {
                sb.append(m.getParameters()[i].getType().getName() + ".class");
                if (i != m.getParameterTypes().length - 1) {
                    sb.append(",");
                }
            }
            sb.append("});" + ln);
            //是否有返回值
            boolean returnFlag = !m.getReturnType().getSimpleName().equals(Void.TYPE.toString());
            if (returnFlag) {
                sb.append("return (" + m.getReturnType().getSimpleName() + ")");
            }
            sb.append("this.h.invoke(this, m, new Object[]{");
            for (int i = 0; i < m.getParameterTypes().length; i++) {
                sb.append(m.getParameters()[i].getName());
                if (i != m.getParameterTypes().length - 1) {
                    sb.append(",");
                }
            }
            sb.append("});" + ln);
            sb.append("} catch(Throwable e) {" + ln);
            sb.append("e.printStackTrace();" + ln);
            if (returnFlag) {
                switch (m.getReturnType().getSimpleName()) {
                    case "int":
                    case "long":
                    case "short":
                    case "float":
                    case "double":
                        sb.append("return 0;");
                        break;
                    case "boolean":
                        sb.append("return false;");
                        break;
                    case "char":
                    case "byte":
                        sb.append("return ' ';");
                        break;
                    default:
                        sb.append("return null;");
                }
            }
            sb.append("}" + ln);
            sb.append("}" + ln);
        }

        sb.append("}" + ln);
        return sb.toString();
    }
}

```


#### 测试方法
``` java
public class MyProxyTest {
    public static void main(String[] args){
        Customer customer = new Customer("陈大春");

        Object proxyObj = new MyHouseProxy().getInstance(customer);
        Rent rent = (Rent) proxyObj;
        System.out.println("proxyObj = " + proxyObj.getClass());
        rent.rent("独栋别野小洋楼，400平，室内泳池");
    }
}
```


### 运行结果
![](/images/posts/myBlog/2019-10-03-Implement-Your-Own-JDK-Dynamic-Proxy-05.png)

### 总结
大功告成，手写JDK动态代理的过程就完成了。
使用Java代码动态生成.java代码文件时，使用StringBuffer拼接代码的时候容易出错，建议把生成的文件放到IDEA格式化后，排查语法错误。当然JDK的真正生成类的代码，并不是像我手写的MyProxy类这样生成的，有兴趣的同学可以自行研究一下JDK的Proxy类源码。