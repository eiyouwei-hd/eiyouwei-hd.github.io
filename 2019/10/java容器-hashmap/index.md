# HashMap之100问


<!--more-->

# Java容器-HashMap

## HashMap原理

- 结构

  ![img](https://uploader.shimo.im/f/srFPXdsKK6Q8sJ7k.png!thumbnail)

  ![img](https://uploader.shimo.im/f/LTOTYzRy3ek2bgor.png!thumbnail)

- 相关参数

  - 默认初始化大小DEFAULT_INITIAL_CAPACITY = 16；
- 默认加载因子DEFAULT_LOAD_FACTOR = 0.75f；
  
  - 数组+链表（1.8红黑树）
- 扩容为2的n次幂
  - hash = h^(h>>>16)（1.8）
  - index = h & (length-1)

## Put/Get

- put：以下为个人总结，具体请参看源码
  - 定位桶：对key的hashCode()做hash运算，计算index
  - 判断是否hash碰撞：
    - 发生了hash碰撞时，判断节点是否是树节点，是就按红黑树的方式存放，否则就按列表的方式存放
      - 列表存放：比较该位置上存储的每一个key是否与新存入的相等，如果相等就替换之，（equals），否则就在该位置增加一个值，新加入的放在链头，最先加入的放在链尾（头插法1.7）
      - 当链表大于预设的阈值8，就要转换成红黑树；
  - 如果bucket满了(超过load factor*current capacity)，就要resize
- get
  - 定位桶：对key的hashCode()做hash运算，计算index;
  - 如果在bucket里的第一个节点里直接命中，则直接返回；
  - 如果有冲突，则通过key.equals(k)去查找对应的Entry;
    - 若为树，则在树中通过key.equals(k)查找，O(logn)；
    - 若为链表，则在链表中通过key.equals(k)查找，O(n)。

## 多线程并发问题

- 多线程扩容时，可能会阐释循环链表，在执行get的时候会触发死循环
- 多线程put的时候可能导致元素丢失
- put非null元素后get出来的却是null
- ref：https://juejin.im/post/5a66a08d5188253dc3321da0

## 1.8做了哪些优化

- 结构改为：数组+链表+红黑树，优化查询性能
  - 在链表元素数量超过8时改为红黑树，少于6时改为链表，中间7不改是避免频繁转换降低性能；
  - 相对于链表，改为红黑树后碰撞元素越多查询效率越高。链表O(n)，红黑树O(logn)
- 优化高位运算的hash算法，降低hash冲突的几率：
  - h^(h>>>16)，将hashcode无符号右移16位，让高16位和低16位进行异或；
- 扩容后，元素要么在原位置上，要么在原位置上移动2的n次幂的位置，且链表的顺序不变
  - 用的尾插法新的数组链表不会倒置，所以不会出现多线程下扩容的死循环问题

## 为什么用数组+链表

- 数组是用来确定桶的位置，利用元素的key的hash值对数组长度取模得到.(定位)
- 链表是用来解决hash冲突问题，当出现hash值一样的情形，就在数组上的对应位置形成一条链表

## 解决hash冲突的方法

- 开放定址法
- 链地址法(Java hashmap就是这么做的)
- 再hash法
- 公共溢出区域法
- 常见Hash算法：MurmurHash、MD4、MD5等等
- 参考链接: 
  - https://blog.csdn.net/zeb_perfect/article/details/52574915
  - https://blog.csdn.net/woaitingting1985/article/details/84621589

## 用LinkedList代替数组结构可以吗？

- 可以
- 为什么用数组不用LinkedList;
  - 查找效率高: 因为用数组效率是最高的,(hashMap可以直接定位到桶的位置,数组的查找效率比linkedList高)
- 为什么用数组不用ArrayList
  - 扩容性能好: 采用基本数组结构，扩容机制可以自己定义，HashMap中数组扩容刚好是2的n次幂，在做取模运算的效率高。而ArrayList的扩容机制是1.5倍扩容

## 为什么扩容是2的n次幂

- 因为下标是通过(n-1)&hash计算出来的

  ![img](https://uploader.shimo.im/f/Nhf6T9YB7PsRwvEm.png!thumbnail)

## 为什么在解决hash冲突的时候，不直接用红黑树?

- 红黑树需要进行左旋，右旋，这些操作来保持平衡：查询快，新增节点慢
- 元素较少时，单链表就能够保证查询性能

## 用二叉查找树代替红黑树，可以么

- 可以
- 但是二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢

## 当链表转为红黑树后，什么时候退化为链表？

- 为6的时候退转为链表。中间有个差值7可以防止链表和树之间频繁的转换。

## 为什么阀值是8呢？

- 时间和空间的权衡

- ref：https://www.javazhiyin.com/34651.html

  

## 一般用什么作为HashMap的key？

- 一般用Integer、String这种**不可变**类当HashMap当key，而且String最为常用
- 因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算，。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。
- 因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的,这些类已经很规范的覆写了hashCode()以及equals()方法。

## 用可变类当HashMap的key有什么问题

- hashcode可能发生改变，导致put进去的值，无法get出

## 如果让你实现一个自定义的class作为HashMap的key该如何实现

- 重写hashcode和equals方法注意什么
  - 两个对象想等，hashCode值一定相等
  - 两个对象不等，hashCode值可能相等
  - hashcode相等，两个对象不一定相等
  - hashcode不等，两个对象一定不等
- 如何设计一个不变类
  - 类添加final修饰符，保证类不被继承。
  - 保证所有成员变量必须私有，并且加上final修饰
  - 不提供改变成员变量的方法，包括setter
  - 通过构造器初始化所有成员，进行深拷贝(deep copy)
  - 在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝



## REF

基本原理：https://juejin.im/post/5b551e8df265da0f84562403

相关问题：https://mp.weixin.qq.com/s/EZQHek-gl2SVnDRQs16nOw

死循环：https://juejin.im/post/5a66a08d5188253dc3321da0



