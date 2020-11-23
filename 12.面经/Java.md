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

== : 当数据类型是基本数据类型时，比较的是栈中数值是否相等。当数据类型是引用类型时，比较的是对象的引用地址是否相等
equals : equals底层默认实现是采用了 `==` 的方式，效果和使用 `==` 一致，一般会根据业务逻辑进行重写

### 6.JVM中内存模型是怎么样的，简述新生代与老年代的区别？

![img](https://i.loli.net/2020/11/20/OgFdkEBMTt56qCI.png)



新生代也称为年轻代，分为 `Eden` 和 `Survivor` 两个区，而 `Survivor` 又分为 `To Survivor` 和 `From Survivor` 。新生代采用的是复制算法，新对象首先会被分配到 `Eden` 区（如果是大对象则会进入老年代），当Eden区没有足够的空间后会触发jvm触发一次 `Minor GC` ，如果一个对象经过一次 `Minor GC` 仍然存活并且能被 `Survivor` 区接受，则该对象将会被移到 `Survivor` 区，并且该对象的年龄加1，当对象年龄到达一定程度（默认为15）则将会被晋升到老年代。
老年代采用标记整理算法(Mark-Compact)，如果老年代没有足够空间后会执行一次 Full GC。

### 7.实现单例设计模式(懒汉、饿汉)

```java
/**
 * 饿汉式
 */
public class Singleton {
  private final static Singleton instance = new Singleton();
  private Singleton() {}
  public static Singleton getInstance() {
    return instance;
  }
}
```

```java
/**
 * 懒汉式
 */
public class Singleton {
  private static Singleton instance;
  private Singleton() {}
  public static Singleton getInstance() {
    if (instance == null) {
      instance = new Singleton();
    }
    return instance;
  }
}
```

```java

/**
 * 双检锁机制懒汉式
 */
public class Singleton {
  private static Singleton instance;
  private Singleton() {}
  public static Singleton getInstance() {
    if (instance == null) {
      synchronized (Singleton.class) {
        if (instance == null) {
          instance = new Singleton();
        }
      }
    }
    return instance;
  }
}
```

```java
/**
 * 静态内部类单例模式
 */
public class Singleton {
  private Singleton() {}
  public static Singleton getInstance() {
    return Inner.singletonHolder;
  }
  private static class Inner {
    private final static Singleton singletonHolder = new Singleton();
  }
}

```

### 8.简述 BIO、NIO  和 AIO 的区别


同步和异步的区别是：同步是直接参与IO读写操作。异步是所有操作都交给了操作系统，等待IO操作完成后，操作系统会发一个通知。
阻塞和非阻塞的区别是：阻塞要等待缓冲区中数据准备完成时才进行处理，否则一直阻塞在那里。非阻塞则是当访问缓冲区中的数据时，如果准备完成则直接返回，如果为准备完成也直接返回，然后会定时轮询查询。

BIO：同步阻塞IO，直接参与IO读写操作，需要等待数据缓冲区中数据全部准备完成才进行处理，否则一直阻塞。
NIO：非同步阻塞IO，直接参与IO读写操作，轮询检查缓冲区中数据是否准备完成，如果完成则直接返回，否则则一直轮询。
AIO：异步非阻塞IO，不直接参与IO读写操作，将操作都交给操作系统，等待缓冲区数据准备完成后等待操作系统发出通知。

### 9. ThreadLocal实现原理是什么？

占坑

### 10. Java线程和操作系统的线程是怎么对应的？

占坑

### 11. Java线程是怎样进行调度的？

占坑

### 12. String类能不能被继承？为什么？

String类不能被继承，因为String类被 `final` 关键字修饰，被 `final` 修饰的类不能被继承。

### 13. 集合类中的List和Map的线程安全版本是什么？如何保证线程安全的？

占坑

### 14. 简述JVM的内存模型。JVM内存是如何对应到操作系统的？

占坑

### 15. Java中垃圾回收机制中如何判断对象需要回收？


垃圾回收器判断对象是否需要回收主要有两种方法，引用计数算法和可达性分析算法

#### 引用计数法
* 给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值加1
* 当引用失效时，计数器值就减1
* 任何时刻计数器为0的对象就是不可能再被使用的

引用计数法的优势是实现简单，判断效率高，但是如果存在循环引用则会失效。

#### 可达性分析算法

通过一系列称为 `GC Roots` 的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径称之为"引用链"，当一个对象到GC Roots没有任何的引用链相连时(从GC Roots到这个对象不可达)时，证明此对象是不可用的

可作为 `GC Roots` 的对象有

* 虚拟机栈(栈帧中的本地变量表)中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 本地方法栈中JNI(Native方法)引用的对象

可达性分析算法优势是更加精确和严谨，可以分析出循环相互引用的情况，但缺点是实现较复杂，分析过程需要GC停顿。

### 16. synchronize 关键字底层是如何实现的？它与Lock相比优缺点是什么？

占坑

### 17. volatile 关键字解决了什么问题，它的实现原理是什么？

volatile 是 Java虚拟机提供的轻量级同步机制，主要是用来解决可见性和有序性问题。

* **lock指令来保证有序性** ：对volatile修饰的变量，执行写操作的话，JVM会发送一条lock前缀指令给CPU，CPU在计算完之后会立即将这个值写回主内存，同时因为有MESI缓存一致性协议，所以各个CPU都会对总线进行嗅探，自己本地缓存中的数据是否被别人修改。如果发现别人修改了某个缓存的数据，那么CPU就会将自己本地缓存的数据过期掉，然后这个CPU上执行的线程在读取那个变量的时候，就会从主内存重新加载最新的数据。
* **内存屏障用来保证有序性** ：对于volatile修改变量的读写操作，都会加入内存屏障。每个volatile写操作前面，加StoreStore屏障，禁止上面的普通写和重排；每个volatile写操作后面，加StoreLoad屏障，禁止跟下面的volatile读/写重排。每个volatile读操作前面，加LoadLoad屏障，禁止下面的普通读和voaltile读重排；每个volatile读操作后面，加LoadStore屏障，禁止下面的普通写和volatile读重排。

### 18. HashMap 与 ConcurrentHashMap 的实现原理是怎么样的？

占坑

### 19. ConcurrentHashMap 是如何保证线程安全的？

占坑

### 20. 常见的GC回收算法有哪些？

#### 引用计数法

简单但是速度很慢，缺陷是不能处理循环引用的情况。 

原理：此对象有一个引用，既增加一个计数器，删除一个引用减少一个计数器，垃圾回收时，只回收计数器为0的对象，此算法最致命的是无法处理循环引用的情况

#### 标记-清除算法

标记清除算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。

之所以说他是最基础的收集算法，是因为后续的收集算法都是基于这种思路并且对其不足进行改进而得到的。

它的主要不足有两个：
1. 一个是效率问题，标记和清除两个过程的效率都不高
2. 另一个是空间问题，标记清除后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后再程序运行过程中需要分配较大对象时，无法找到足够的连续的内存而不得不提前触发另一次垃圾收集动作。


#### 复制算法
为了解决效率问题，一种称为“复制”（Copying）的收集算法出现了，它将可用的内存按照容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。这样使得每次都是对整个半区进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况。只要移动堆指针，按照顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为原来的一半。


#### 标记-整理算法
复制收集算法在对象存活率较高的时候，就要进行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要额外的空间进行分配担保，以应对被使用的内存中所有对象都是100%存活的极端情况，所以在老年代一般不能直接选用这种算法。根据老年代的特点。有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与”标记-清除”算法一样，但是后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界意外的内存。


#### 分代收集算法
这种算法并没有什么新的思路，只是根据对象的存活周期的不同，将内存划分为几块。一般是把java堆分成新生代和老年代，这样就可以根绝各个年代的特点采取最适当的收集算法。在新生代中，每次垃圾回收时都发现大批对象的死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高，没有额外空间对它进行分配担保，就必须使用”标记-清理”或者“标记-整理”算法来进行回收。




