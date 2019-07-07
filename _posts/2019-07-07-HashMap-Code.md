---
layout: post
title: Java集合HashMap 源码分析
tags: java集合
---
<!-- TOC -->

- [一、HashMap简介](#一hashmap简介)
- [二、底层数据结构分析](#二底层数据结构分析)
    - [(1)存放元素的结构](#1存放元素的结构)
    - [(2)红黑树节点的数据类型定义为：](#2红黑树节点的数据类型定义为)
    - [(3)HashMap类的一些属性](#3hashmap类的一些属性)
- [三、HashMap的构造方法](#三hashmap的构造方法)
- [四、HashMap的关键方法](#四hashmap的关键方法)
    - [(1)put方法](#1put方法)
    - [(2)get方法](#2get方法)
    - [(3)resize方法](#3resize方法)
    - [(4)containsKey方法](#4containskey方法)
    - [(5)containsValue方法](#5containsvalue方法)
- [五、高频HashMap面试题](#五高频hashmap面试题)

<!-- /TOC -->
### 一、HashMap简介
HashMap 主要用来存放键值对，它基于哈希表的Map接口实现，是常用的Java集合之一。

JDK1.8 之前 HashMap 由 数组+链表 组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）.JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树，以减少搜索时间。

**HashMap的键和值都可以为 null，且HashMap不是线程安全的**

### 二、底层数据结构分析

#### (1)存放元素的结构
HashMap的底层数据结构就是数组（hash桶），这个数组存放的数据结构类型为：

```java
  // 继承自 Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
       final int hash;// 哈希值，存放元素到hashmap中时用来与其他元素hash值比较
       final K key;//键
       V value;//值
       // 指向下一个节点
       Node<K,V> next;
       Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        // 重写hashCode()方法
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        // 重写 equals() 方法
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
}
```

#### (2)红黑树节点的数据类型定义为：
```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // 父
        TreeNode<K,V> left;    // 左
        TreeNode<K,V> right;   // 右
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;           // 判断颜色
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
        // 返回根节点
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
       }
```

#### (3)HashMap类的一些属性
```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L; 
    // 默认的初始容量是16,总是2的幂次倍
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的填充因子（负载因子）
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table; 
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 加载因子
    final float loadFactor;
}
```

### 三、HashMap的构造方法
    1. public HashMap(int initialCapacity, float loadFactor)
    2. public HashMap(int initialCapacity)
    3. public HashMap() 
    4. public HashMap(Map<? extends K, ? extends V> m)
构造方法源码如下：
```java
    /**
     * 指定“容量大小”和“加载因子”的构造函数
     *
     * @param  initialCapacity 初始化容量
     * @param  loadFactor      负载因子
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * 指定“容量大小”的构造函数，负载因子默认为0.75
     *
     * @param  initialCapacity 初始化容量
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 默认构造函数，初始容量为16，负载因子为0.75
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * 包含另一个“Map”的构造函数
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```
putMapEntries方法：

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) { // pre-size
            // 未初始化，s为m的实际元素个数
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // 将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```


### 四、HashMap的关键方法
#### (1)put方法
①如果定位到的数组位置没有元素 就直接插入。
②如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。
![](/images/posts/myBlog/2019-07-07-HashMap-Code-01.png)
```java
    /**
    * 提供给外部使用的put方法
    **/
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }


    /**
     * 实际HashMap内部使用的方法
     *
     * @param hash hash值
     * @param key key值
     * @param value 存入元素value
     * @param onlyIfAbsent 如果为true，不改变原来存在的值
     * @param evict if false, the table is in creation mode.
     * @return 原来的值, 如果没有值返回null
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
         // table未初始化或者长度为0，进行扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            // 桶中已经存在元素
            Node<K,V> e; K k;
            // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                  // 将第一个元素赋值给e，用e来记录
                e = p;
            // hash值不相等，即key不相等，且为红黑树结点
            else if (p instanceof TreeNode)
                 // 放入树中
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 为链表结点
            else {
                 // 在链表最末插入结点
                for (int binCount = 0; ; ++binCount) {
                    // 到达链表的尾部
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 结点数量达到阈值，转化为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        // 跳出循环
                        break;
                    }
                    // 判断链表中结点的key值与插入的元素的key值是否相等
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        // 相等，跳出循环
                        break;
                    // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                    p = e;
                }
            }
             // 表示在桶中找到key值、hash值与插入元素相等的结点
            if (e != null) { // existing mapping for key
                // 记录e的value
                V oldValue = e.value;
                 // onlyIfAbsent为false或者旧值为null
                if (!onlyIfAbsent || oldValue == null)
                    //用新值替换旧值
                    e.value = value;
                // 访问后回调
                afterNodeAccess(e);
                // 返回旧值
                return oldValue;
            }
        }
        // 结构性修改
        ++modCount;
        // 实际大小大于阈值则扩容
        if (++size > threshold)
            resize();
        // 插入后回调
        afterNodeInsertion(evict);
        return null;
    }
    
```

这里我们重点分析一下JDK1.8使用的hash函数，这个计算得到的hash值是hashCode的高16位和低16位异或的结果，放入hash桶的时候，取的是 **(table.length - 1) & hash** 得出数组的下标，
```java
static final int hash(Object key) {
        int h;
        //如果key为null，hash值为0，否则，为key的hashCode ^ 它的hashCode右移16位
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

#### (2)get方法
```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### (3)resize方法
进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。
容量扩充为原来的两倍，rehash结果只有两种，一种是在**原下标位置**，另一种是在下标为 **<原下标+原容量>**的位置
```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { 
        // signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```


#### (4)containsKey方法
这个方法其实调用的就是get方法
```java
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
```

#### (5)containsValue方法
遍历这个hash表，查找是否包含value值，由于是遍历整个hash表，耗时较大，不建议使用
```java
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```


### 五、高频HashMap面试题
1. HashMap的原理，内部数据结构？
	+ 底层使用Hash表（数组+链表+红黑树），当链表过长（>=8）会将链表转成红黑树以实现O(logn)时间复杂度内查找
2. 讲一下HashMap中put方法过程？
	i. 对Key求Hash值（hashCode的高16位异或低16位），然后计算 下标 ，计算的hash值 & (数组大小-1)
	ii. 如果没有碰撞，直接放入桶中
	iii. 如果碰撞了，以链表的方式链接在后面
	iv. 如果链表长度超过阀值(TREEIFY_THRESHOLD ==8),就把链表转成红黑树
	v. 如果节点已经存在就替换旧值
	vi. 如果桶满了(容量*负载因子),就需要resize
3. HashMap中hash函数是如何实现的？
	i. 高16bit不变，低16bit和高16bit做了一个异或
	ii. (n-1) & hash --->得到下标
4. HashMap怎么解决冲突，讲一下扩容过程，加入一个值在原数组中，现在移动了新数组，位置肯定改变了，那是什么定位到在这个新值数组中的位置？
    - 将新节点加到链表后
    - 容量扩容为原来的两倍，然后对每个节点重新计算哈希值
    - 这个值只可能在两个地方：一个是原下标的位置，另一种是在下标为 <原下标+原容量> 的位置
5. 抛开HashMap，hash冲突有哪些解决办法？
	- 开放地址法，链地址法

6. 针对HashMap中某个Entry链太长，查找的时间复杂度可能达到O(n)，怎么优化
	- 将链表转为红黑树，JDK1.8已经实现了，查找复杂度为log(n)


注：本文参考JavaGuide项目文章，文章地址：
- [https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/HashMap.md](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/HashMap.md)


