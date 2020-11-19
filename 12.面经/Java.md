### 1.SpringBoot如何进行自动装配的？

SpringBoot启动的时候会通过 `@EnableAutoConfiguration` 注解找到 `META-INF/spring.factories` 配置文件中的所有自动配置类，并对其进行加载，而这些自动配置类都是以AutoConfiguration结尾来命名的。它实际上就是一个JavaConfig形式的Spring容器配置类，它能通过以Properties结尾命名的类中取得在全局配置文件中配置的属性如 `server.port` ，而 `XxxxProperties` 类是通过 `@ConfigurationProperties` 注解与全局配置文件中对应的属性进行绑定的。

### 2.简述 Java 的 happen befor 原则

在Java内存模型（JMM）中，如果一个操作的执行结果需要对另一个操作可见，那么两个操作必须要存在happens-before关系。

happens-before原则是判断数据是否存在竞争，线程是否安全的主要依据。happends-before 也可解释为`生效可见于` 

**happens-before原则定义如下**：

1. 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。 
2. 两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行。如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法。

**happens-before原则规则**：

* 程序次序规则：在同一个线程中，写在前面的操作happen-before后面的操作
* 锁定规则：同一个锁的unlock操作happen-before此锁的lock操作
* volatile变量规则：对一个volatile变量的写操作happen-before对此变量的任意操作
* 传递性规则：如果A操作 happen-before B操作，B操作happen-before C操作，那么A操作happen-before C操作
* 线程启动规则：同一个线程的start方法happen-before此线程的其它方法
* 线程中断规则：对线程interrupt方法的调用happen-before被中断线程的检测到中断发送的代码
* 线程终结规则：线程中所有的操作都先行发生于线程的终止检测
* 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法



### 3.简述 synchronize、volatile和可重入锁的不同使用场景及优缺点

占坑

### 4.简述 Spring AOP 的原理

占坑

### 5. == 和 equals() 的区别

占坑

### 6.JVM中内存模型是怎么样的，简述新生代与老年代的区别？

占坑

### 7.实现单例设计模式(懒汉、饿汉)

占坑

### 8.简述 BIO、NIO  和 AIO 的区别

占坑

### 9. ThreadLocal实现原理是什么？

占坑

### 10. Java线程和操作系统的线程是怎么对应的？

占坑

### 11. Java线程是怎样进行调度的？

占坑

### 12. String类能不能被继承？为什么？

占坑

### 13. 集合类中的List和Map的线程安全版本是什么？如何保证线程安全的？

占坑

### 14. 简述JVM的内存模型。JVM内存是如何对应到操作系统的？

占坑

### 15. Java中垃圾回收机制中如何判断对象需要回收？常见的GC回收算法有哪些？

占坑

### 16. synchronize 关键字底层是如何实现的？它与Lock相比优缺点是什么？

占坑

### 17. volatile 关键字解决了什么问题，它的实现原理是什么？

占坑

### 18. HashMap 与 ConcurrentHashMap 的实现原理是怎么样的？

占坑

### 19. ConcurrentHashMap 是如何保证线程安全的？

占坑



