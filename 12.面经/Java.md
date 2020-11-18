### 1.SpringBoot如何进行自动装配的？

SpringBoot启动的时候会通过 `@EnableAutoConfiguration` 注解找到 `META-INF/spring.factories` 配置文件中的所有自动配置类，并对其进行加载，而这些自动配置类都是以AutoConfiguration结尾来命名的。它实际上就是一个JavaConfig形式的Spring容器配置类，它能通过以Properties结尾命名的类中取得在全局配置文件中配置的属性如 `server.port` ，而 `XxxxProperties` 类是通过 `@ConfigurationProperties` 注解与全局配置文件中对应的属性进行绑定的。

### 2.简述 Java 的 happen befor 原则

占坑

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



