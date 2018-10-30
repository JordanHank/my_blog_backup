---
title: ArrayList和LinkedList的区别比较
date: 2018-10-30 22:46:27
toc: true
tags:
    - 集合
    - ArrayList
    - LinkedList
---
Java 集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合，另一种是图（Map），存储键/值对映射。而在集合类型中最常用的就是ArrayList和LinkedList这个集合类，所以了解这个的区别对于使用集合类很有帮助。

### ArrayList与LinkedList的继承关系
ArrayList与LinkedList都是List接口的实现类,因此都实现了List的所有未实现的方法,只是实现的方式有所不同。ArrayList实现了List接口,它是以数组的方式来实现的,数组的特性是可以使用索引的方式来快速定位对象的位置,因此对于快速的随机取得对象的需求,使用ArrayList实现执行效率上会比较好.。LinkedList是采用链表的方式来实现List接口的,它本身有自己特定的方法，如: addFirst(),addLast(),getFirst(),removeFirst()等. 由于是采用链表实现的,因此在进行insert和remove动作时在效率上要比ArrayList要好得多!适合用来实现Stack(堆栈)与Queue(队列),前者先进后出，后者是先进先出。

<!--more-->

![ArrayList与LinkedList的继承关系](/assets/img/listDifferent.png)

### ArrayList 介绍
* ArrayList 是一个数组队列，和数组不同的是能够动态扩容，也就是我们说的动态数组;
* 继承自AbstractLis，有实现了List接口，可以使用添加、修改、删除遍历等功能
* 实现了RandomAccess接口，提供随机访问功能，即通过数组下标访问元素
* 实现了Cloneable接口，覆盖了函数clone()，能被克隆
* 实现了java.io.Serializable接口，支持序列化，能通过序列化去传输
* 和Vector不同的是ArrayList不是线程安全的，所以涉及多线程的时候不建议使用

ArrayList的构造函数：
```
    //默认构造函数，底层数组大小默认为10，空间不如时新的容量=“(原始容量x3)/2 + 1”
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
    //指定数组大小的构造函数，空间不足时容量会添加上次容量的一般
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
        
    //指定Collection的创建   
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

ArrayList的三种遍历方法，使用随机访问(即，通过索引序号访问)效率最高；使用迭代器进行遍历 效率最低！

```
    //默认方式创建初始化
    ArrayList<Integer> array = new ArrayList<Integer>();
    //添加元素
    for (int i = 0; i < 10; i++) {
        array.add(i);
    }
    
     System.out.println("随机访问遍历数组array：");
     for (int i = 0; i < array.size(); i++) {
        //获取元素
        System.out.print(array.get(i) + " ");
     }
     System.out.println();
     
     System.out.println("随机访问遍历数组array：");
     for (Integer i: array) {
        System.out.print(i + " ");
     }
     System.out.println();
    
    //迭代器进行遍历 效率最低！
    Iterator iterator = array.iterator();
    System.out.println("迭代器遍历数组array中包含元素：");
    while (iterator.hasNext()) {
        System.out.print(iterator.next() + " ");
    }
```

### LinkedList 介绍
+ LinkedList 是一个继承于AbstractSequentialList的双向链表,可以被当作堆栈、队列或双端队列进行操作
+ 实现了List接口，可以使用添加、修改、删除遍历等功能
+ 实现 Deque 接口，即能将LinkedList当作双端队列使用
+ 实现了Cloneable接口，覆盖了函数clone()，能被克隆
+ 实现了java.io.Serializable接口，支持序列化，能通过序列化去传输
+ LinkedList是非线程同步的，所以多线程的场景不建议使用

LinkedList的构造函数：
```
 //默认构造函数
 public LinkedList() {
 }
 
 //指定Collection的创建   
 public LinkedList(Collection<? extends E> c) {
     this();
     addAll(c);
 }

```

LinkedList的简单使用：
```
/**
 * LinkedList 实际上是通过双向链表去实现的
 * 内部类Node是双向链表节点所对应的数据结构：当前节点所包含的值，上一个节点，下一个节点
 */
LinkedList linkedList = new LinkedList();

//添加元素
for (int i = 0; i < 10; i++) {
    //属于Collection接口的方法 会抛出异常
    linkedList.add(i);
}

//向头部添加元素 属于Collection接口的方法 会抛出异常
linkedList.addFirst(-1);

//获取第一个元素 属于Collection接口的方法 会抛出异常
Object first = linkedList.getFirst();
System.out.println("linkedList数组的第一个元素是："+ first);
System.out.println();

//向尾部添加元素 属于Collection接口的方法 会抛出异常
linkedList.addLast(100);

//获取最后一个元素 属于Collection接口的方法 会抛出异常
Object last = linkedList.getLast();
System.out.println("linkedList数组的最后一个元素是：" + last);
System.out.println();

//移除第一个元素 属于Collection接口的方法 会抛出异常
Object removeF = linkedList.removeFirst();
System.out.println("linkedList数组移除第一个元素：" + removeF);
first = linkedList.getFirst();
System.out.println("linkedList数组移除第一个元素后第一元素为：" + first);
System.out.println();

//移除最后一个元素 属于Collection接口的方法 会抛出异常
Object removeL = linkedList.removeLast();
System.out.println("linkedList数组移除最后一个元素：" + removeL);
last = linkedList.getLast();
System.out.println("linkedList数组移除最后一个元素后最后一元素为：" + last);
System.out.println();

//通过迭代器遍历
System.out.println("通过迭代器遍历:");
for (Iterator iterator = linkedList.listIterator(); iterator.hasNext();) {
    System.out.print(iterator.next() + " ");
}
System.out.println();

//Deque队列时使用的添加方法 返回boolean值不会抛出异常
boolean offer = linkedList.offer("offer");
System.out.println("linkedList当做队列时添加元素offer返回：" + offer);
System.out.println();

//向队列添加到头部
boolean offerF = linkedList.offerFirst("first");
System.out.println("linkedList当做队列时添加头部元素first返回：" + offerF);
//获取队列头部元素
first = linkedList.peekFirst();
System.out.println("peek linkedList第一个元素：" + first);
System.out.println();

//向队列尾部添加元素
boolean offerL = linkedList.offerLast("last");
System.out.println("linkedList当做队列时添加尾部元素last返回：" + offerL);
//获取队列尾部元素
last = linkedList.peekLast();
System.out.println("peek linkedList最后一个元素：" + last);
System.out.println();

//移除头部元素
Object pollF = linkedList.pollFirst();
System.out.println("linkedList数组移除第一个元素：" + pollF);
//获取第一元素
first = linkedList.element();
System.out.println("linkedList数组移除第一个元素后第一元素为：" + first);
System.out.println();

//移除尾部元素
Object pollL = linkedList.pollLast();
System.out.println("linkedList数组移除最后一个元素：" + pollL);
last = linkedList.getLast();
System.out.println("linkedList数组移除最后一个元素后最后一元素为：" + last);
System.out.println();

//通过for循环遍历 不推荐使用 速度很慢
System.out.println("通过for循环遍历:");
for (int i = 0; i < linkedList.size(); i++) {
    System.out.print(linkedList.get(i) + " ");
}
System.out.println();

//LinkedList可以作为LIFO(后进先出)的栈，作为LIFO的栈时
//添加到头部
linkedList.push("push");
System.out.println("linkedList数组的第一元素为：" + linkedList.peekFirst());
System.out.println();

//移除头部元素
Object pop = linkedList.pop();
System.out.println("linkedList数组移除第一个元素：" + pop);
System.out.println();

//for增强循环 推荐使用
System.out.println("for增强循环：");
for (Object o: linkedList) {
    System.out.print(o + " ");
}
System.out.println();
System.out.println();

//拷贝数组
LinkedList copy = (LinkedList) linkedList.clone();
LinkedList copy1 = (LinkedList) linkedList.clone();
LinkedList copy2 = (LinkedList) linkedList.clone();
LinkedList copy3 = (LinkedList) linkedList.clone();

/**
 * 下面的遍历方法都会移除原始数据：
 * 使用removeFist()或removeLast()效率最高
 */
//通过pollFirst()来遍历LinkedList
System.out.println("通过pollFirst()来遍历LinkedList：");
do {
    if (copy.size() > 0) {
        System.out.print(copy.getFirst() + " ");
    }
} while (copy.pollFirst() != null);

System.out.println();
System.out.println();

//通过pollLast()来遍历LinkedList
System.out.println("通过pollLast()来遍历LinkedList：");
do {
    if (copy1.size() > 0) {
        System.out.print(copy1.getLast() + " ");
    }
} while (copy1.pollLast() != null);
System.out.println();
System.out.println();

//通过removeFirst()来遍历LinkedList
System.out.println("通过removeFirst()来遍历LinkedList：");
try {
    do {
        if (copy2.size() > 0) {
            System.out.print(copy2.getFirst() + " ");
        }
    } while(copy2.size() > 0 && copy2.removeFirst() != null);
    System.out.println();
} catch (NoSuchElementException e) {
    e.printStackTrace();
}
System.out.println();

//通过removeLast()来遍历LinkedList
System.out.println("通过removeLast()来遍历LinkedList：");
try {
    do {
        if (copy3.size() > 0) {
            System.out.print(copy3.getLast() + " ");
        }
    } while(copy3.size() > 0 && copy3.removeLast() != null);
    System.out.println();
} catch (NoSuchElementException e) {
    e.printStackTrace();
}
```

### ArrayList与LinkedList异同
ArrayList与LinkedList都是List的具体实现类，所以存储的都是有序元素，可以通过下标进行随机访问元素，不过由于实现List的方式不同，导致了两者的差异。ArrayList的底层是数组，数组是一块连续的内存空间，优势就是随机访问速度快，但是插入删除元素导致其他元素的位置改变所以速度慢；LinkedList使用了链表的结构，内部类Entry存储了当前节点所包含的值，上一个节点，下一个节点等信息，查询时需要遍历所有元素找到所查元素这就意味着数据量越大查询速度越慢，除非位置离头部很近；但是插入、删除元素只需要对元素的前一元素的尾部和后一个元素的尾部进行修改即可，其他的元素不需要改动，所以速度很快。

|List集合  |  实现List方式  |  查询速度| 插入删除速度 | 是否有序 |
| :------ | :------ | :------ | :-------- | :------ |
|ArrayList  |  数组  |  速度快| 速度慢 | 有序 |
|LinkedList  |  双向链表  |  速度慢| 速度快 | 有序 |