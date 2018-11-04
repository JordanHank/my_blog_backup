---
title: HashMap和Hashtable的区别比较
date: 2018-11-04 20:37:05
toc: true
tags:
    - 集合
    - HashMap
    - Hashtable
---
Java 集合框架中主要包含的容器类除了集合（Collection），另一种就是图（Map），主要用来存储键值对，其中最常用的就是HashMap,而HashMap和早期设计的Hashtable又有许多相似之处，所以经常将二者进行比较。

### HashMap和Hashtable的继承关系
HashMap 继承自AbstractMap类，实现了Map、Cloneable、java.io.Serializable接口；而Hashtable继承自 Dictionary,同样实现了Map、Cloneable、java.io.Serializable接口。Hashtable是java一开始发布时就提供的键值映射的数据结构，而HashMap产生于JDK1.2。虽然Hashtable比HashMap出现的早一些，但是现在Hashtable基本上已经被弃用了。

<!--more-->

![HashMap和Hashtable的继承关系](/assets/img/mapDifferent.png)

### HashMap 介绍
* HashMap 是一个散列表，用于储存键值对
* HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口
* 可以被克隆，支持序列化，能通过序列化去传输
* HashMap 不是线程同步的，储存的键值对可以是null;但只能有一个键为null
* HashMap 默认初始值为16，扩充因子默认为0.75，扩充容量翻倍：capacity * 2

### HashMap 的构造函数

```
 //默认构造函数 默认初始容量为16
 public HashMap() {
     this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
  }

 //指定初始容量的构造函数 容量不足时会扩充为原来的2倍
 public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
 }
 
 //指定初始容量和扩充因子的构造函数
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
 
 //指定Map集合的构造函数
 public HashMap(Map<? extends K, ? extends V> m) {
     this.loadFactor = DEFAULT_LOAD_FACTOR;
     putMapEntries(m, false);
 }
```

### HashMap 的几种遍历方法
```
//创建初始化
HashMap hashMap = new HashMap();

//添加元素
hashMap.put("name", "Jordan Zhang");
hashMap.put("gender", "man");
hashMap.put("hobby", "sports");

//迭代器遍历键值对
System.out.println("迭代器遍历键值对");
Iterator iterator = hashMap.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry entry = (Map.Entry)iterator.next();
    // 获取key
    String key = (String)entry.getKey();
    // 获取value
    String value = (String)entry.getValue();
    System.out.println("key: " + key + " value: " + value);
}
System.out.println();

//通过遍历键遍历 无法获取键key
System.out.println("通过遍历键遍历");
iterator = hashMap.keySet().iterator();
while (iterator.hasNext()) {
    // 获取key
    String key = (String)iterator.next();
    // 获取value
    String value = (String)hashMap.get(key);
    System.out.println("key: " + key + " value: " + value);
}
System.out.println();

//通过遍历值遍历
System.out.println("通过遍历值遍历");
Collection collection = hashMap.values();
iterator = collection.iterator();
while (iterator.hasNext()) {
    System.out.print(iterator.next() + " ");
}
```

### Hashtable 的介绍
+ Hashtable 和 HashMap一样也是散列表，用来存储键值对
+ Hashtable 继承自 Dictionary,实现了Map、Cloneable、java.io.Serializable接口
+ 可以进行克隆，支持序列化，能通过序列化去传输
+ Hashtable 是线程同步的，所以它是线程安全的，多线程也可以使用
+ Hashtable 的键key 和 值value都不可以是null,存储的映射也不是有序的
+ Hashtable 的初始容量为11，扩充因子为0.75，扩充容量为：capacity * 2 + 1

### Hashtable 的构造函数
```
//默认构造函数 初始容量为11，扩充因子为0.75
public Hashtable() {
    this(11, 0.75f);
}

//指定初始容量的构造函数 扩充容量为 capacity * 2 + 1
public Hashtable(int initialCapacity) {
    this(initialCapacity, 0.75f);
}

//指定初始容量和扩充因子的构造函数
public Hashtable(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load: "+loadFactor);

    if (initialCapacity==0)
        initialCapacity = 1;
    this.loadFactor = loadFactor;
    table = new Entry<?,?>[initialCapacity];
    threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}

//指定Map集合的构造函数
public Hashtable(Map<? extends K, ? extends V> t) {
    this(Math.max(2*t.size(), 11), 0.75f);
    putAll(t);
}
```

### Hashtable 的遍历方法
```
//默认初始化
Hashtable hashTable = new Hashtable();

//添加元素
hashTable.put("name", "Jordan Zhang");
hashTable.put("gender", "man");
hashTable.put("hobby", "sports");

//迭代器遍历键值对
System.out.println("迭代器遍历键值对");
Iterator iterator = hashTable.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry entry = (Map.Entry) iterator.next();
    // 获取key
    String key = (String) entry.getKey();
    // 获取value
    String value = (String) entry.getValue();
    System.out.println("key: " + key + " value: " + value);
}
System.out.println();

//通过遍历键遍历
System.out.println("通过遍历键遍历");
iterator = hashTable.keySet().iterator();
while (iterator.hasNext()) {
    // 获取key
    String key = (String) iterator.next();
    // 获取value
    String value = (String) hashTable.get(key);
    System.out.println("key: " + key + " value: " + value);
}
System.out.println();

//通过遍历值遍历 无法获取键key
System.out.println("通过遍历值遍历");
Collection collection = hashTable.values();
iterator = collection.iterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
System.out.println();
System.out.println();

//通过Enumeration遍历Hashtable的键
System.out.println("通过Enumeration遍历Hashtable的键");
Enumeration keys = hashTable.keys();
while(keys.hasMoreElements()) {
    String key = (String) keys.nextElement();
    String value = (String) hashTable.get(key);
    System.out.println("key: " + key + " value: " + value);
}
System.out.println();

//通过Enumeration遍历Hashtable的值 无法获取键key
System.out.println("通过Enumeration遍历Hashtable的值");
Enumeration values = hashTable.elements();
while (values.hasMoreElements()) {
    System.out.println(values.nextElement());
}

```

### HashMap和Hashtable的异同
HashMap和Hashtable都是实现了Map、Cloneable、java.io.Serializable接口，可以被克隆，支持序列化，能通过序列化去传输；但是HashMap继承自AbstractMap,Hashtable继承自Dictionary,Hashtable是线程安全的，HashMap不是。HashMap允许键或值为null，而Hashtable不允许。

|Map集合 |  HashMap |  Hashtable |
| :------ | :------ | :------ |
| 继承父类 | AbstractMap |  Dictionary |
| 实现接口 | Cloneable,Serializable | Cloneable,Serializable |
| 线程安全 | 非线程安全 | 线程安全 |
| 键值是否支持Null |  键或值可以是Null | 键和值不允许是Null|
| 提供contains方法 | 没有 | 有 | 
| 初始容量 | 默认16 | 默认11|
| 扩充容量 | capacity * 2 | capacity * 2 + 1|
| 遍历实现 | 使用了 Iterator | 使用了 Iterator、Enumeration|
| hash值 | 直接使用对象的hashCode | 重新计算hash值|