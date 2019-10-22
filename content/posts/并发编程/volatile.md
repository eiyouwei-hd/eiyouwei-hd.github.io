title = "volatile原理"
date = "2019-10-22"
description = "volatile原理"
tags = [
    "并发编程",
    "volatile"
]
categories = [
    "并发编程"
]

## volatile

- volatile可以保证线程可见性且提供了一定的有序性，但是无法保证原子性。在JVM底层volatile是采用“内存屏障”来实现的。（lock前缀指令）
  - 保证可见性、不保证原子性
  - 禁止指令重排序
- 实现原理：有volatile变量修饰的共享变量进行写操作的时候会使用CPU提供的Lock前缀指令。
  - 将当前处理器缓存行的数据写回到系统内存。
  - 这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。
- 使用需满足两个条件（保证原子性）：
  - 对变量的写操作不依赖当前值；
  - 该变量没有包含在具有其他变量的不变式中
- 常用场景：状态标记、double check；

## REF

https://www.javazhiyin.com/15589.html

