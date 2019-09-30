---
layout: post
title: 单例模式(Singleton pattern)
tags: 设计模式   
---

### 为什么要使用单例模式
单例模式属于上篇说过的设计模式三大分类中的第一类——创建型模式。顾名思义，单例设计模式就是为了保证创建出来的对象实例只有一个。
- 通过控制创建对象的数量，节约系统资源开销。
- 有些场景下，不使用单例模式，会导致系统同一时刻出现多个状态缺乏同步，用户自然无法判断当前处于什么状态
- 全局数据共享


### 饿汉式
饿汉式：在类加载的时候完成初始化，不管是否使用该单例对象，先初始化出来再说。
缺点：可能不使用初始化单例对象，但调用类其他静态成员是，类加载就完成了初始化，这是不必要的开销，如果单例对象的初始化非常复杂，就会造成资源的浪费。

``` java
public class HungrySingleton {
  private HungrySingleton() {}  // 构造方法私有化，防止通过new创建对象实例

  private static HungrySingleton instance = new HungrySingleton();  //持有私有static修饰的对象实例引用，

  public static HungrySingleton getInstance() {     //公开静态获取对象实例的方法
    return instance;
  }
}
```

### 懒汉式
懒汉式：在需要用到单例对象实例的时候才完成初始化工作，能不初始化就不初始化。
实现的关键是：定义单例对象的引用时，没有初始化，在获取实例对象的方法中判定是否完成初始化，没有初始化才进行单例对象的创建工作。
``` java
public class LazySingleton {
  private static LazySingleton instance = null; //静态私有空对象引用
  private LazySingleton() {  //构造函数私有化
  }
  public static LazySingleton getInstance() { //公开获取实例方法
    if(instance == null){   //对象为空时，创建对象实例
      instance = new LazySingleton();
    }
    return instance;
  }
}
```

思考：这段程序看起来没什么问题，但是如果在高并发的环境下，就可能出现问题，因为可能有多个线程同时执行到判空操作，都通过了空指针的条件，进行对象的创建，解决方法就是对创建对象的过程进行同步，引入同步代码块。

``` java
public class LazySingleton {
  private LazySingleton() {
  }
  private static volatile LazySingleton instance = null;

  public static synchronized LazySingleton getInstance() {
    if(instance == null){
      instance = new LazySingleton();
    }
    return instance;
  }
}
```
这样加同步锁，确实也能解决了高并发的问题了，但是，锁的粒度是否有点过大呢？如果对象已经创建，还是需要排队进入同步代码块，效率有点低下了，其实可以直接返回实例，优化代码如下,传说中的双重检测锁
``` java
public class LazySingleton {
  private LazySingleton() {
  }
  private static volatile LazySingleton instance = null;    //注意要用volatile修饰，保证线程可见性

  public static LazySingleton getInstance() {
    if(instance == null){
      synchronized(LazySingleton.class){
        if(instance == null)
          instance = new LazySingleton();
      }
    }
    return instance;
  }
}
``` 
### 单例模式的其他实现方式
#### 静态内部类实现
根据类加载机制，外部类的初始化并不会导致静态内部类的初始化。
``` java
public class StaticInnerSingleton {
    private StaticInnerSingleton() {
    }
    private static class StaticInnerSingletonInstance {
      private static final StaticInnerSingleton instance = new StaticInnerSingleton();
    }
    public static StaticInnerSingleton getInstance() {
      return StaticInnerSingletonInstance.instance;
    }
}
```

1. StaticInnerSingletonInstance是一个静态内部类，内部静态字段instance负责创建对象。因为上面的结论，所以外部类StaticInnerSingleton初始化时，并不会导致StaticInnerSingletonInstance初始化，进而导致instance的初始化。所以实现了延迟加载。
2. 当外部调用getInstance()时，通过StaticInnerSingletonInstance.instance对instance引用才会导致对象的创建。由于static的属性只会跟随类加载初始化一次，天然保证了线程安全问题。


#### 枚举实现
用枚举实现单例是最简单的了，因为，Java中的枚举类型本身就天然单例的

``` java
enum EnumSingletonInstance{
   INSTANCE;
   public static EnumSingletonInstance getInstance(){
   		return INSTANCE;
   }
}
```
唯一遗憾的是，这个方案和饿汉式一样，没法延迟加载。枚举类加载自然就会初始化INSTANCE。

### 破解单例
#### 反射破解法
``` java
public static void main(String[] args) throws Exception {
        System.out.println(HungrySingleton.getInstance());
        System.out.println(HungrySingleton.getInstance());
        System.out.println("反射破解单例...");
        HungrySingleton instance1 = HungrySingleton.class.newInstance();
        HungrySingleton instance2 = HungrySingleton.class.newInstance();
        System.out.println(instance1);
        System.out.println(instance2);
}
```
反射的原理是调用默认的无参构造函数进行初始化，防止反射破解单例的方法就是在无参构造函数加入单例对象引用是否初始化的判断，如果已经初始化，抛出异常
``` java
private HungrySingleton() {
    if (instance != null) {
      try {
        throw new Exception("只能创建一个对象！");
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
}
```


#### 反序列化破解法
通过序列化和反序列化也可以破解单例。（前提是单例类实现了Serializable接口）代码如下：
``` java
public static void main(String[] args) throws Exception {
        System.out.println(HungrySingleton.getInstance());
        System.out.println(HungrySingleton.getInstance());
        System.out.println("反序列化破解单例...");
        HungrySingleton instance1 = HungrySingleton.getInstance();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream out = new ObjectOutputStream(baos);
        out.writeObject(instance1);	//序列化
        ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(baos.toByteArray()));
        HungrySingleton instance2 = (HungrySingleton) ois.readObject();	//反序列化
        System.out.println(instance1);
        System.out.println(instance2);
}
```
要防止序列化破坏单例，只需要在单例类中添加如下readResolve()方法，然后在方法体中返回我们的单例实例即可。为什么？因为readResolve()方法是在readObject()方法之后才被调用，因而它每次都会用我们自己生成的单实例替换从流中读取的对象。这样自然就保证了单例。
``` java
private Object readResolve() throws ObjectStreamException{
  return instance;
}
```


#### 
### 总结

    从安全性角度考虑，枚举显然是最安全的，保证绝对的单例，因为可以天然防止反射和反序列化的破解手段。而其它方案一定场合下全部可以被破解。

    从延迟加载考虑，懒汉式、双重检测锁、静态内部类方案都可以实现，然而双重检测锁方案代码实现复杂，而且还有对JDK版本的要求，首先排除。懒汉式加锁性能较差，
    而静态内部类实现方法既能够延迟加载节约资源，另外也不需要加锁，性能较好，所以这方面考虑静态内部类方案最佳。

    饿汉式和懒汉式实现有几个要点：
    1. 构造函数私有化
    2. private static修饰的单例对象引用
    3. 公开的获取对象实例的静态方法