title = "java容器简介"
date = "2019-10-22"
description = "java容器简介"
tags = [
    "集合",
    "java基础"
]
categories = [
    "集合"
]

# Java容器-简介

## 综述

- 概述：

  - java.util.Collection：

    - List：
    - ArrayList
      - LinkedList
      - Vector
        - Stack
    - Set
      - HashSet
      - TreeSet
      - LinkedSet
  
  - java.util.Map
  
    - HashMap
    - HashTable
    - WeakHashMap
  
  - 同步容器：
  
    - Vector、Stack
    - HashTable
    - 封装容器类：Collections.synchronized方法生成的
  
  - 并发容器：
  
    - ConcurrentHashMap：线程安全的HashMap的实现
  
    - CopyOnWrite容器：CopyOnWriteArrayList、CopyOnWriteArraySet；
  
    - 阻塞队列：ArrayBlockingQueue、LinkedBlockingQueue；
  
      

## ArrayList

- 底层数据结构是数组；线程不安全的；
- 默认大小为10；扩容1.5倍：newCapacity = oldCapacity + (oldCapacity >> 1)
- 随机访问

## LinkedList

- 底层数据结构是双向链表；线程不安全的；
- 随机增、删

## Vector

- 类似ArrayList，但是Vector是**同步**的
- vector所谓的多线程安全，只是针对单纯地调用某个方法它是有同步机制的
- vector的多线程安全，在组合操作时不是线程安全的，需要自己用Synchronized将组合操作进行同步

## Stack

- 继承自Vector，实现一个后进先出的栈。
- 常用方法：
  - push（E e）：入栈
  - pop（）：出栈
  - peek（）：获得栈顶元素
  - empty（）：检测是否为空
  - search（）：检测一个元素在栈中的位置

## HashSet

- 不重复，无序；
- 底层是HashMap；

## TreeSet

- TreeSet描述的是Set的一种变体——可以实现排序等功能的集合

## HashMap

- 初始大小为16，加载因子为0.75f，扩容为2的n次幂
- 数组+链表（1.8红黑树）
- 哈希值 =  hashcode高低十六位异或过的值；

## HashTable

- extends Dictionary
- 方法是同步的
- kay，value都不允许出现null值
- 哈希值直接使用的是对象的hashCode
- hash数组默认大小是11，扩容是old*2+1

## WeakHashMap

- WeakHashMap是一种改进的HashMap，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收



## REF

https://blog.csdn.net/u013030086/article/details/84791668#LinkedList_63http://blog.csdn.net/wonxxx/article/details/51787041)



