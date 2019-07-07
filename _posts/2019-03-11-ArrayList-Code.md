---
layout: post
title: Java集合ArrayList 源码分析
tags: java集合   
---
<!-- TOC -->

- [一、ArrayList简介](#一arraylist简介)
- [二、ArrayList的数据结构](#二arraylist的数据结构)
- [三、ArrayList构造方法](#三arraylist构造方法)
- [四、ArrayList的方法](#四arraylist的方法)
    - [常用方法解析](#常用方法解析)
- [五、ArrayList支持3种遍历方式](#五arraylist支持3种遍历方式)
- [六、toArray()异常](#六toarray异常)

<!-- /TOC -->
#### 一、ArrayList简介
ArrayList 是一个数组队列，相当于 动态数组。与Java中的数组相比，它的容量能动态增长。它继承于AbstractList，实现了List, RandomAccess, Cloneable, java.io.Serializable这些接口。

ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。
ArrayList 实现了RandmoAccess接口，即提供了随机访问功能。**RandmoAccess**是java中用来被List实现，为List提供快速访问功能的（实际上只是一个**标志接口**，没有实现，用于标志判断）。在ArrayList中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。稍后，我们会比较List的“快速随机访问”和“通过Iterator迭代器访问”的效率。

ArrayList 实现了Cloneable接口，即覆盖了函数clone()，能被克隆。

ArrayList 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输。

和Vector不同，**ArrayList中的操作不是线程安全的！**所以，建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。


#### 二、ArrayList的数据结构
ArrayList的数据结构非常简单，就是数组Object[]，只不过在多了一个size（数组大小）
```java
    /**
     * Default initial capacity.
     * 默认的初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;
	 /**
     * Shared empty array instance used for empty instances.
     * 共享空数组对象实例（final修饰）
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
  
    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     * 默认大小的共享空数组实例，我们将此与EMPTY_ELEMENTDATA区分开，
     * 以了解何时膨胀多少第一个元素被添加。
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     * ArrayList保存元素的数组，transient修饰，该成员不需要序列化
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     * @serial
     * ArrayList所包含元素的数量
     */
    private int size;

```

#### 三、ArrayList构造方法
ArrayList有三个构造方法
	public ArrayList()
	public ArrayList(int initialCapacity)
	public ArrayList(Collection<? extends E> c)

也就是有三种方式获取到一个ArrayList实例

```java
 /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

#### 四、ArrayList的方法
```java
// Collection中定义的API
boolean             add(E object)
boolean             addAll(Collection<? extends E> collection)
void                clear()
boolean             contains(Object object)
boolean             containsAll(Collection<?> collection)
boolean             equals(Object object)
int                 hashCode()
boolean             isEmpty()
Iterator<E>         iterator()
boolean             remove(Object object)
boolean             removeAll(Collection<?> collection)
boolean             retainAll(Collection<?> collection)
int                 size()
<T> T[]             toArray(T[] array)
Object[]            toArray()
// AbstractCollection中定义的API
void                add(int location, E object)
boolean             addAll(int location, Collection<? extends E> collection)
E                   get(int location)
int                 indexOf(Object object)
int                 lastIndexOf(Object object)
ListIterator<E>     listIterator(int location)
ListIterator<E>     listIterator()
E                   remove(int location)
E                   set(int location, E object)
List<E>             subList(int start, int end)
// ArrayList新增的API
Object               clone()
void                 ensureCapacity(int minimumCapacity)
void                 trimToSize()
void                 removeRange(int fromIndex, int toIndex)
```

##### 常用方法解析
+ add方法：public boolean add(E e)
	我们这里重点解析一下ArrayList是如何进行自动扩容的，我们简单总结一下扩容的基本逻辑
	1. 计算是否需要扩容
	2. 如果需要扩容，则计算扩容，数组结构被修改时会触发：modCount++，用于跌代时判定数组是否被修改，抛出ConcurrentModificationException 异常，标志数组已经被修改；
	3. 如果需要扩容，进行扩容过程：数组拷贝到新的数组中，新的数组为原来数组的 3 / 2 倍
	4. 往新数组中设置添加的元素
    
```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    //确保容量，自动扩容
	 private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
	
	//返回容量大小
	private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
    
    //
	private void ensureExplicitCapacity(int minCapacity) {
        modCount++;	// 标志数组的结构被修改次数，每次修改都会加1，主要用于迭代器的判定，如果在迭代其中有修改数组的结构，将抛出 ConcurrentModificationException

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

	/**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);		//重点！！扩容的大小是原来的 3/2 倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);		//是否大于最大容量，最大容量是 Integer.MAX_VALUE - 8，如果超过这个容量，则容量为 Integer.MAX_VALUE
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);		//调用系统的数组复制方法对数组进行复制
    }
```

+ **contains 方法：public boolean contains(Object o)**
```java
	//判断一个列表里面是否包含某个元素
	public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
    //查找某元素的下标，遍历整个列表
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

#### 五、ArrayList支持3种遍历方式
+  第一种，通过迭代器遍历。即通过Iterator去遍历。
```java
	Integer value = null;
	Iterator iter = list.iterator();
	while (iter.hasNext()) {
    	value = (Integer)iter.next();
	}
```
+ 第二种，随机访问，通过索引值去遍历。
由于ArrayList实现了RandomAccess接口，它支持通过索引值去随机访问元素。
```java
	Integer value = null;
	int size = list.size();
	for (int i=0; i<size; i++) {
    	value = (Integer)list.get(i);        
	}
```

+ 第三种，for循环遍历。如下：
```java
	Integer value = null;
	for (Integer integ:list) {
    	value = integ;
	}
```

tips: 遍历ArrayList时，使用**随机访问(即，通过索引序号访问)效率最高**，而使用**迭代器的效率最低**！有兴趣的小伙伴可以自行测试一下


#### 六、toArray()异常

+ Object[] toArray()
+ <T> T[] toArray(T[] contents)

调用 toArray() 函数会抛出“java.lang.ClassCastException”异常，但是调用 toArray(T[] contents) 能正常返回 T[]。

toArray() 会抛出异常是因为 toArray() 返回的是 Object[] 数组，将 Object[] 转换为其它类型(如如，将Object[]转换为的Integer[])则会抛出“java.lang.ClassCastException”异常，因为Java不支持向下转型。具体的可以参考前面ArrayList.java的源码介绍部分的toArray()。
解决该问题的办法是调用 <T> T[] toArray(T[] contents) ， 而不是 Object[] toArray()。

调用 toArray(T[] contents) 返回T[]的可以通过以下几种方式实现。
```java
// toArray(T[] contents)调用方式一
public static Integer[] vectorToArray1(ArrayList<Integer> v) {
    Integer[] newText = new Integer[v.size()];
    v.toArray(newText);
    return newText;
}

// toArray(T[] contents)调用方式二。最常用！
public static Integer[] vectorToArray2(ArrayList<Integer> v) {
    Integer[] newText = (Integer[])v.toArray(new Integer[0]);
    return newText;
}

// toArray(T[] contents)调用方式三
public static Integer[] vectorToArray3(ArrayList<Integer> v) {
    Integer[] newText = new Integer[v.size()];
    Integer[] newStrings = (Integer[])v.toArray(newText);
    return newStrings;
}
```



