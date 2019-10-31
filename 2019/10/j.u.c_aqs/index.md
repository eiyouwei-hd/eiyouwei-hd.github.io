# j.u.c_AQS

<!--more-->



## AQS

- AQS：AbstractQueuedSynchronizer队列同步器，它是构建锁或者其他同步组件的基础框架

- 采用模板方法的设计模式，子类通过继承的方式，实现它的抽象方法来管理同步状态

- 定义了一个volatile int state变量作为共享资源，维护了一个FIFO同步队列：CLH；

  - 当线程获取资源成功，则执行临界区代码，否则就进入队列中等待；

  - 执行完释放资源时，会通知同步队列中的等待线程来获取资源后出队并执行；

  - 状态常量：

    ![1571652697725](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571652697725.png)

- AQS提供了大量的模板方法来实现同步，主要是分为三类：
  
  - 独占式：同一时刻仅有一个线程持有同步状态
  - 共享式：共享式在同一时刻可以有多个线程获取同步状态；
  - 查询同步队列中的等待线程情况



## 独占式

- 同步状态获取：acquire(int arg)；

  - 源码：

    ![1571642217922](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571642217922.png)

  - 流程：

    ![【死磕Java并发】—– J.U.C之AQS：同步状态的获取与释放](https://www.javazhiyin.com/wp-content/uploads/2018/08/java9-1534474804.jpg)

  - 先去尝试获取同步状态，若获取失败则，向CLH队列尾部插入一个节点，并自选等待，直到获取锁为止，

  - tryAcquire：尝试获取锁，获取成功则设置锁状态并返回true，否则返回false；需要自己实现：要保证线程安全的获取同步状态；

  - addWaiter：将当前线程加入到CLH同步队列尾部

  - acquireQueued：当前线程会根据公平性原则来进行阻塞等待（自旋）,直到获取锁为止；

    ![1571651927069](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571651927069.png)

    - 当且仅当只有其前驱节点为头结点才能够尝试获取同步状态
    - 获取同步状态成功，则将当前节点设为头节点
    - 失败则继续等待

  - selfInterrupt：产生一个中断

- 获取响应中断：

  - acquireInterruptibly(int arg)：在等待获取同步状态时，如果当前线程被中断了，会**立刻**响应中断抛出异常InterruptedException

  - 原因：AQS提供了acquire(int arg)方法以供独占式获取同步状态，但是该方法对中断不响应：即加入队列后不会移除，会一直等待获取同步状态；所以AQS提供了获取响应中断的模板方法

  - 源码：

    ![1571643910225](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571643910225.png)

  - 先校验该线程是够已经中断，是的话直接抛异常：InterruptedException；否则，尝试去或同步状态tryAcquire，若失败执行doAcquireInterruptibly；

  - doAcquireInterruptibly(int arg)

    - 与acquire方法差不多，不同的是acquire在自旋等待时返回是否被中断过，而doAcquireInterruptibly是直接抛异常；

      ![1571644268632](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571644268632.png)

    

- 超时获取：

  - tryAcquireNanos(int arg,long nanos)：当前线程没有在指定时间被获取同步状态则会返回false；

    - 响应中断存；
    - 超时控制；
  
- 源码：
  
  ![1571644482722](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571644482722.png)
  
    ![1571644567489](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571644567489.png)
  
- 流程图
  
  ![【死磕Java并发】—– J.U.C之AQS：同步状态的获取与释放](https://www.javazhiyin.com/wp-content/uploads/2018/08/java6-1534474804.jpg)
  
- 释放同步状态
  
  - release(int arg)：释放同步状态，唤醒后继节点
  
  - 源码：
  
      ![1571645095971](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571645095971.png)



## 共享式

- 获取同步状态：

  - acquireShared

  - tryAcquireShared：尝试获取同步状态，返回值为int，当其 >= 0 时，表示能够获取到同步状态，这个时候就可以从自旋过程中退出。

  - 源码：

    ![1571650114343](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571650114343.png)

    ![1571650149456](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571650149456.png)

- 获取响应中断：acquireSharedInterruptibly

- 超时获取：tryAcquireSharedNanos

- 释放：

  - releaseShared

  - 可能会存在多个线程同时进行释放同步状态资源，所以需要确保同步状态安全地成功释放，一般都是通过**CAS+循环**来完成的

  - 源码：

    ![1571650423041](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571650423041.png)

    ![1571650461541](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571650461541.png)



## 线程阻塞or唤醒

- 当获取同步方法失败后需要阻塞当前线程并自旋等待：

  - shouldParkAfterFailedAcquire：根据前驱节点，判断当前线程是否需要阻塞，检查逻辑：
  
    - 前驱节点状态为SINNAL，则表明当前线程需要被阻塞，调用unpark()方法唤醒，直接返回true，当前线程阻塞
  - 前驱节点状态为CANCELLED（ws > 0），则表明该线程的前驱节点已经等待超时或者被中断了，则需要从CLH队列中将该前驱节点删除掉，直到回溯到前驱节点状态 <= 0 ，返回false
    - 非SINNAL，非CANCELLED，则通过CAS的方式将其前驱节点设置为SINNAL，返回false
  
    ![1571651865192](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571651865192.png)
    
  - parkAndCheckInterrupt：阻塞当前线程
  
- 当线程释放同步状态时，需要唤醒后续节点：

  - unparkSuccessor

    ![1571652382011](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1571652382011.png)

  - 当前节点状态<0，则设置为0；

  - 后继节点为null || 状>0（超时或被中断了）

  - 从tail尾节点开始找，找到第一个可用节点，唤醒他；



## 总结

- AQS是j.u.c包实现同步的基础工具，使用了模板方法的设计模式，提供了独占、共享、中断、超时等特性的方法；
- AQS的子类可以定义不同的资源实现不同特性的方法，比如后文要写，的ReentrantLock，定义state为0时可以获取资源并置为1。若已获得资源则不断+1，在释放资源时减1，直至减为0；再比如CountDownLatch、CyclicBarrier、Semaphore等等；

## REF

https://www.javazhiyin.com/15063.html

https://www.javazhiyin.com/15242.html