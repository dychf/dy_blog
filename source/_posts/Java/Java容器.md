---
title: Java容器
tags: Java 代理
date: 2022-02-15
---

## 集合概览
Java 集合， 也叫作容器，主要是由两大接口派生而来：一个是 Collection（有三个主要的子接口：List，Set，Queue）接口，主要用于存放单一元素；另一个是 Map 接口，主要用于存放键值对。

<!-- more -->

## Arraylist 和 Vector 的区别
* ArrayList 是 List 的主要实现类，底层使用 Object[ ]存储，适用于频繁的查找工作，线程不安全；
* Vector 是 List 的古老实现类，底层使用Object[ ] 存储，线程安全的。

## Arraylist 与 LinkedList 区别?
* **线程安全**：两者都不能保证线程安全
* **数据结构**：Arraylist 底层使用的是 Object 数组；LinkedList 底层使用的是 双向链表 数据结构
* **插入和删除是否受元素位置的影响**：ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 LinkedList在头尾插入或者删除元素不受元素位置的影响。
* **是否支持随机访问**：ArrayList 支持，LinkedList 不支持。
* **内存占用**：ArrayList 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间。 LinkedList 因为要存放直接后继和直接前驱以及数据，因此每个元素都需要更多的空间。

## ArrayList的扩容机制
ArrayList有三种构造方法：
* 无参构造函数，返回一个空素组，在添加元素时构造一个容量为10的列表。
* 带有一个整数型参数的构造函数，返回有指定长度的列表。
* 带有Collection的构造方法，构造一个包含指定元素的列表。
ArrayList是List接口的实现类，它是支持根据需要而动态增长的数组。
ArrayList扩容发生在add()方法调用的时候，下面是add()方法的源码：
```
public boolean add(E e) {
       //扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
根据意思可以看出***ensureCapacityInternal***()是用来扩容的，形参为最小扩容量，进入此方法后：
```
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```
**ensureExplicitCapacity**方法可以判断是否需要扩容：
```
 private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 如果最小需要空间比elementData的内存空间要大，则需要扩容
    if (minCapacity - elementData.length > 0)
        //扩容
        grow(minCapacity);
}
```
如果修改后的数组容量大于当前的数组长度那么就需要调用grow进行扩容。ArrayList扩容的关键方法grow():
```
 private void grow(int minCapacity) {
    // 获取到ArrayList中elementData数组的内存空间长度
    int oldCapacity = elementData.length;
    // 扩容至原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 再判断一下新数组的容量够不够，够了就直接使用这个长度创建新数组，
    // 不够就将数组长度设置为需要的长度
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //若预设值大于默认的最大值检查是否溢出
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 调用Arrays.copyOf方法将elementData数组指向新的内存空间时newCapacity的连续空间
    // 并将elementData的数据复制到新的内存空间
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
首先对旧容量oldCapacity进行1.5倍的扩容，之后如果newCapacity仍然小于最小要求容量minCapacity，就把minCapacity的值给到newCapacity。如果newCapacity的值大于最大要求容量，则会走hugeCapacity()方法，进行一个最终扩容后的大小确认，最后调用Arrays.copyof()方法，在原数组的基础上进行复制，容量变为newCapacity的大小。


## comparable 和 Comparator 的区别
* comparable 接口实际上是出自java.lang包 它有一个 compareTo(Object obj)方法用来排序 
* comparator接口实际上是出自 java.util 包它有一个compare(Object obj1, Object obj2)方法用来排序

## 无序性和不可重复性
* 无序性：存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的哈希值决定的。
* 添加的元素按照 equals()判断时 ，返回 false，需要同时重写 equals()方法和 HashCode()方法。

## HashSet/LinkedHashSet/TreeSet
 * 线程安全：HashSet、LinkedHashSet 和 TreeSet 都是 Set 接口的实现类，都能保证元素唯一，并且都不是线程安全的。
 * 数据结构：HashSet 的底层数据结构是哈希表。LinkedHashSet 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。TreeSet 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。
 * 使用场景：HashSet 用于不需要保证元素插入和取出顺序的场景；LinkedHashSet 用于保证元素的插入和取出顺序满足 FIFO 的场景；TreeSet 用于支持对元素自定义排序规则的场景。

## PriorityQueue
* PriorityQueue利用了二叉堆的数据结构来实现的，底层使用可变长的数组来存储数据
* 非线程安全
* 默认是小顶堆，但可以接收一个 Comparator 作为构造参数，从而来自定义元素优先级的先后。


## HashMap 和 Hashtable
* 线程安全：HashMap 是非线程安全的；Hashtable 内部的方法基本都经过synchronized 修饰，是线程安全的。
* 效率：因为线程安全的问题，HashMap的效率更高。
* null key/value：HashMap可以存储空的key和value，但null key只能有一个； Hashtable 不允许有 null 键和 null 值。
* 初始容量及扩容：Hashtable 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1；HashMap 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。
* 底层数据结构：JDK1.8后的HashMap在解决哈希碰撞时，当链表长度大于阈值（默认为8）时，会将链表转换为红黑树。

## HashMap 的底层实现
JDK1.8 之前 HashMap 底层是 数组和链表 结合在一起使用也就是 链表散列。HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

 JDK1.8 之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

**扰动函数** 指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。

## HashMap扩容机制


## ConcurrentHashMap/Hashtable
两者区别主要体现在实现线程安全的方式上不同。
* 底层数据结构：JDK1.7 的 ConcurrentHashMap 底层采用 分段的数组+链表 实现，JDK1.8 采用的数据结构跟 HashMap1.8 的结构一样，数组+链表/红黑二叉树。Hashtable 采用 数组+链表 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的
* 实现线程安全的方式：
在 JDK1.7 的时候，ConcurrentHashMap（分段锁） 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 到了 JDK1.8 的时候已经摒弃了 Segment 的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6 以后 对 synchronized 锁做了很多优化） 整个看起来就像是优化过且线程安全的 HashMap，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本；② Hashtable(同一把锁) :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

## ConcurrentHashMap线程安全具体实现方式
* JDK1.7 首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

* JDK1.8 ConcurrentHashMap 取消了 Segment 分段锁，采用 CAS 和 synchronized 来保证并发安全。数据结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))。
synchronized 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。