[TOC]

# java

## 思想

### 写入时复制（copyOnWrite）
  
CopyOnWriteArrayList使用了一种叫写时复制的方法，当有新元素添加到CopyOnWriteArrayList时，先从原有的数组中拷贝一份出来，然后在新的数组做写操作，写完之后，再将原来的数组引用指向到新数组。
优点：提高并发

缺点：1：因为写的时候需要拷贝，所以会增加内存开销     
     2:只能保证最终数据一致性，不能保证实时数据一致，即写入的数据不一定立即会被读取，因为引用地址可能还没来得及被改变。


### fail-fast 机制

快速失败机制，是java集合(Collection)中的一种错误机制。当在迭代集合的过程中该集合在结构上发生改变的时候，就有可能会发生fail-fast，即抛出ConcurrentModificationException异常。fail-fast机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测bug。

实现：
int cursor;       // index of next element to return
int lastRet = -1; // index of last element returned; -1 if no such
int expectedModCount = modCount;

cursor是指集合遍历过程中的即将遍历的元素的索引，lastRet是cursor -1，默认为-1，即不存在上一个时，为-1，它主要用于记录刚刚遍历过的元素的索引。expectedModCount这个就是fail-fast判断的关键变量了，它初始值就为ArrayList中的modCount。（modCount是抽象类AbstractList中的变量，默认为0，而ArrayList 继承了AbstractList ，所以也有这个变量，modCount用于记录集合操作过程中作的修改次数，与size还是有区别的，并不一定等于size）
从源码知道，每次调用next()方法，在实际访问元素前，都会调用checkForComodification方法，该方法源码如下：
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
可以看出，该方法才是判断是否抛出ConcurrentModificationException异常的关键。在该段代码中，当modCount != expectedModCount
时，就会抛出该异常。但是在一开始的时候，expectedModCount初始值默认等于modCount，为什么会出现modCount != expectedModCount，很明显expectedModCount在整个迭代过程除了一开始赋予初始值modCount外，并没有再发生改变，所以可能发生改变的就只有modCount，在前面关于ArrayList扩容机制的分析中，可以知道在ArrayList进行add，remove，clear等涉及到修改集合中的元素个数的操作时，modCount就会发生改变(modCount ++),所以当另一个线程(并发修改)或者同一个线程遍历过程中，调用相关方法使集合的个数发生改变，就会使modCount发生变化，这样在checkForComodification方法中就会抛出ConcurrentModificationException异常。
类似的，hashMap中发生的原理也是一样的


### CAS

**大概原理：**modcount

cas失败后怎么处理？

缺点？

## 多态

重载（参数列表，为什么不能按返回值？），

重写，

向上转型（spring实现相关），将子类实现赋给父类引用

## 泛型

为什么提出泛型（编译期检查）

泛型方法，框架应用较多

## JVM

### 类加载机制

### 内存模型

1. 常量池
2. 方法区

​                 方法表

​                 ->子类对父类方法的继承

​                  ->子类重写父类方法的实现原理

3. 虚拟机栈

   动态链接（类加载过程中 ，解析时符号引用变为直接引用，链接的是堆中的实例或者变量池的变量）

### 垃圾回收

思想，根节点算法，什么是根节点

垃圾回收思想

​	标记清除，标记整理，复制，压缩，分期（新生代和老年代算法的区别，原因），分片，后面的出现是为了解决前面的缺点

调优思路是什么，死锁，死循环，对象长久不释放的原因可能是？

## 集合

hashmap在1.7和之后版本实现的区别

​	红黑树，为什么遍历快？

hashmap为什么不能支持高并发？

concurrentHashmap在1.7和之后版本实现的区别

​	主要在扩容部分

内部比较器和外部比较器，各自优点

## 注解

​	实现原理

## java8和java9的新特性

​	lambda表达式

​	stream

​	模块化

## 异常

​	常见的异常有哪些，如何处理

​	try...catch在java8中的新的实现?

# 并发

## 线程

1. 线程的几种状态
2. 竞态和临界区
3. 锁池和等待池（涉及到condition和队列同步器的理解）
4. yeild，sleep（TimeUnit），waite区别

## 思想

1. happensbefore

2. volatile（为什么不能保证并发操作，因为可能不是原子操作）

   1. 可见性

   2. 禁止重排序

   3. 上面的两个特性实现原理（更新主存，内存屏障，各自是什么

## synchronize关键字

   1. 用在方法上和代码块在字节码中有什么区别，各自如何实现

   2. 在新版本中如何优化的

      涉及到锁提升过程：

      1. 内置锁
      2. 偏向锁
      3. 轻量锁
      4. 轮询
      5. 重量锁

   3. 可重入为什么？

## lock

优点

为了增加并发效率，读写分离，进一步，可重入

可重入的读写锁需要考虑锁降级，为什么

## 并发工具包类关系

```flow
st=>start: 同步队列
e=>end: 结束
op=>operation: condition(LockSupport.park()和CAS实现)
op2=>operation: 是否共享
sub=>subroutine: 队列同步器(AQS)
sub7=>subroutine: 队列同步器(AQS)
sub8=>subroutine: countdownlatch
sub9=>subroutine: ArrayBlockingQueue
sub10=>subroutine: LinkedBlockingQueue
cond=>condition: 结合（实现原理）
op3=>operation: Sync
sub5=>subroutine: fair
sub6=>subroutine: nonfair
sub2=>subroutine: reentrantlock（CyclicBarrier的实现）
sub3=>subroutine: semaphore
sub4=>subroutine: 阻塞队列
cond2=>condition: 是不是共享锁?
cond3=>condition: 是不是公平锁?
cond4=>condition: 一个condition控制？
io=>inputoutput: 无
io2=>inputoutput: 阻塞队列

st->op->cond
cond(yes)->sub(right)->op3->cond3
cond3(yes,right)->sub5(right)->sub7->cond2
cond3(no)->sub6->sub7->cond2
cond2(yes,right)->sub3->sub8
cond2(no)->sub2->sub4->cond4
cond4(yes,right)->sub9
cond4(no)->sub10



```

## ThreadLocal

## 线程池

1. 主要是哪几个核心部件构成，阻塞队列，核心线程数，最大线程数，线程存活时间，拒绝策略

# Spring

## IOC

实现原理，主要是启动过程，几个核心类的作用

**AbstractBeanFactory**，抽象类，提供装载和实例化bean的**模板**，启动时会加载框架用的bean和监听器等，监听context的启动

**beandefinition**，封装bean的元信息和实例

beandefinitionReader，具体读取bean信息

**Applicationcontext**封装beanFactory和实现了beanFactory提供的读取bean方式的方法，**refresh**()方法中实现初始化处理

启动时从beanFactory取得bean并且提供**beanpostprocessor**在bean初始化前后实现目标方法外的拦截处理，为AOP提供了支点(**何时拦截**)

## AOP

proxycreator将需要拦截的class遍历出来，创建代理类和pointcutAdvisor

ponitcut通过methodmatch方法判断某个方法是否需要拦截（**何处拦截**）

advisor（**哪些增强**）

**反射**的实现原理

## 事务

隔离级别，传播方法。。。

项目应用

## JPA

springData可以了解一下

## springboot

启动流程和注解实现原理

主要是autoconfiguration包下的类

配置文件yaml或properties文件中属性的实现原理

## 设计模式

# mybatis

二级缓存，几个注解

redis？实现原理，跳跃表

# DB

## 常用的方法

## 连接原理

## 索引

1. 实现原理，各种索引间区别，何时需要索引，有哪些注意事项
2. B树和B+树的区别，节点是否有数据，是否存在链表

## 隔离级别

1. 各个级别出现的原因是什么
2. 如何提高并发，快照读，记录锁，间隙锁，临间锁，多版本控制
3. 底层实现原理，redo，undo日志

# NIO

线程模型

netty类似

# HTTP

三次握手和四次挥手，

其他的我也不了解

# 附录：

## Tomcat

pipeline和valve模式

netty类似

## Linux

用户，组，权限，网络，shell编程

## Elasticsearch

## docker









