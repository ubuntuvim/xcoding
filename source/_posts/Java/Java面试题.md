---
title: Java面试题
tag:
	- java
---



### 面试问题汇总

1. SpringBoot常用注解？

    最主要注解`@SpringBootApplication`，这个注解其实是包含了`ComponentScan`、`@Configuration`、`EnableAutoConfiguration`三个注解。

    `@RestController`，`@Service`、`@ResponseBody`，`@RequestBody`

    `@PathVariable`用户获取路径参数，`@RequestParam`用户获取查询参数。

    `@ConfigurationProperties`用户读取配置信息并注入到bean的属性上。

2. Oracle索引原理？

    默认索引类型是B树，Oracle会维护索引列值和rowid值映射关系表。主要分为根、茎、叶三个部分，索引列值放在叶子节点上，茎存放叶子节点相关信息。

3. Springcloud组件有哪些？

    eureka服务注册发现，ribbon客户端负载均衡，hystrix熔断器，zuul（getway）服务网关，config（Apollo）分布式配置。

4. 线程池的拒绝策略有哪些？

    AbortPolicy：丢弃任务并抛出拒绝异常RejectExcutionException（默认策略）

    DiscardPolicy：丢弃任务并且不会抛出异常

    DiscardOldestPolicy：丢弃队列最前面的任务，执行后面的任务

    CallerRunsPolicy：有调用线程池处理，也就是当触发拒绝策略时，只要线程池没有关闭，就由提交任务的当前线程处理。



### Java基础



1. [HashMap的源码，实现原理，JDK8中对HashMap做了怎样的优化](https://blog.csdn.net/weixin_37356262/article/details/80543218)

    > JDK8中长度超过8链表转为红黑树，提高查找效率，避免并发情况下覆盖问题。

2. [HaspMap扩容是怎样扩容的，为什么都是2的N次幂的大小](https://blog.csdn.net/Apeopl/article/details/88935422)

    > 获取下标时使用&运算，2的N次幂数据很高效并且hash值比较均匀。

3. [HashMap，HashTable，ConcurrentHashMap的区别](https://www.cnblogs.com/heyonggang/p/9112731.html)

    > HashTable底层数组+链表实现，无论key还是value都**不能为null**，线程**安全**
    >
    > HashMap底层数组+链表（JDK8长度超过8转为红黑树）key和Value可以为null，线程不安全
    >
    > ConcurrentHashMap是线程安全的HashMap

4. [极高并发下HashTable和ConcurrentHashMap哪个性能更好，为什么，如何实现的](https://www.cnblogs.com/heyonggang/p/9112731.html)

    > 后者，HashTable是直接对整个table加锁，ConcurrentHashMap是分段加锁，分为16个table，可以同时支持16个线程操作。

5. HashMap在高并发下如果没有处理线程安全会有怎样的安全隐患，具体表现是什么。

    > 并发修改问题、数据覆盖问题、数据丢失问题、循环链表问题（JDK7之前）。

6. [java中四种修饰符的限制范围](https://blog.csdn.net/weixin_37356262/article/details/80554821)

    > public所有包、类都可以访问
    >
    > protected自身、子类可以访问
    >
    > private自身可以访问
    >
    > default同包中的类可以访问

7. [Object类中的方法](https://www.jianshu.com/p/51d9ba549172)

    > equals、hashCode、clone、wait、notify、notifyAll、toString、getClass、finalize

8. 接口和抽象类的区别，注意JDK8的接口可以有实现。

    > JDK8之后几乎差别不大，除了一点，接口支持多继承，抽象类不支持。

9. [动态代理的两种方式，以及区别](https://blog.csdn.net/weixin_36759405/article/details/82770422)

    > JDK动态代理：利用反射机制生成代理接口的匿名类，在具体方法调用前先调用InvokeHandler，但是只能代理接口。
    >
    > CGLIB动态代理：利用ASM加载代理类字节码，修改字节码生成一个子类。但不能代理final修饰的类。

10. Java序列化的方式。

    实现序列号接口Serializable，或者通过JSON实现

11. [传值和传引用的区别，Java是怎么样的，有没有传值引用](https://www.zhihu.com/question/31203609)

    简单讲只有传值，如果是对象类型传的值是对象地址，如果是简单类型传的值是原址的拷贝。

12. [一个ArrayList在循环过程中删除，会不会出问题，为什么](https://juejin.cn/post/6844903672481054733)

    不管是在单线程还是多线程情况下，都是有可能出现并发修改异常（`ConcurrentModificationException`）

13. @transactional注解在什么情况下会失效，为什么。

    在非public方法上使用时、作用在接口上，但是接口却是用CGLIB也会导致失效。

14. Java 集合类框架的基本接口有哪些？

    List、Map、Set、Collection

15. HashSet 和 TreeSet 有什么区别？

    都可以去重，HashSet无序，TreeSet有序。

16. HashSet 的底层实现是什么?

    HashMap

17. LinkedHashMap 的实现原理?

    底层也是通过HashMap实现，只不过相比HashMap来说LinkedHashMap是有序的（插入顺序和访问顺序）。

18. 为什么集合类没有实现 Cloneable 和 Serializable 接口？

    具体实现类都是有实现的。只是在集合类接口上没有实现，考虑到接口的职责应该单一。

19. 数组 (Array) 和列表 (ArrayList) 有什么区别？什么时候应该使用 Array 而不是 ArrayList？

20. Java 集合类框架的最佳实践有哪些？

    ArrayList,HashMap,HashSet这些不就是咯

21. Set 里的元素是不能重复的，那么用什么方法来区分重复与否呢？是用 == 还是 equals()？它们有何区别？

    使用equals判断，

    equals判断对象的内容是否相等

    ==判断基本类型的值和引用是否相等

22. Comparable 和 Comparator 接口是干什么的？列出它们的区别

    Comparable通常是在集合内部定义排序，Comparator是在外部定义排序。

23. Collection 和 Collections 的区别。

    一个是集合接口，一个是封装好的集合工具包

### JVM与调优

1. JVM的内存结构。

2. JVM方法栈的工作过程，方法栈和本地方法栈有什么区别。

3. JVM的栈中引用如何和堆中的对象产生关联。

4. 可以了解一下逃逸分析技术。

5. GC的常见算法

6. CMS以及G1的垃圾回收过程，CMS的各个阶段哪两个是Stop the world的，CMS会不会产生碎片，G1的优势。

7. 双亲委派模型的过程以及优势。

8. 常用的JVM调优参数。

9. 对象什么时候进入老年代？

10. 什么是内存溢出， 内存泄露？ 他们的区别是什么？

11. 引起类加载操作的行为有哪些？

    Class.forName，反射，new，调用静态方法或者静态属性，

12. 介绍一下 JVM 提供的常用工具

    jconsole,jstat,jmap,jvisualvm,

13. Full GC 、 Major GC 、Minor GC 之间区别？

14. 什么时候触发 Full GC ？

15. 什么情况下会出现栈溢出

16. 说一下强引用、软引用、弱引用、虚引用以及他们之间和 gc 的关系

17. Eden 和 Survivor 的比例分配是什么情况？为什么？

    Eden是8/10，两个survivor分别是1/10，原因：jdk通过统计发现，大部分的对象生命周期都是很短的（大部分都是局部变量），经过一次MinorGC之后通常只剩下10%左右。所以只需要设置10%的survivor。

18. 什么是分布式垃圾回收（DGC）？它是如何工作的？

19. 串行（serial）收集器和吞吐量（throughput）收集器的区别是什么？

    serial是单线程收集，收集垃圾时会发生STW，导致用户线程被暂停，所以吞吐量很低。

20. 在 Java 中，对象什么时候可以被垃圾回收？

    没有任何引用后会被回收。

      

### Java并发



### Spring

1. 谈谈对 Spring IoC 的理解？
2. 谈谈对 Spring DI 的理解？
3. BeanFactory 接口和 ApplicationContext 接口不同点是什么？
4. 请介绍你熟悉的 Spring 核心类，并说明有什么作用？
5. 介绍一下 Spring 的事务的了解？
6. 介绍一下 Spring 的事务实现方式？
7. Spring 配置 Bean 实例化有哪些方式？
8. Bean 注入属性有哪几种方式
9. 在 Spring 中如何实现时间处理？
10. Spring 中如何更高效的使用 JDBC ？
11. 请介绍一下设计模式在 Spring 框架中的使用？
12. IoC 控制反转设计原理？
13. Spring 的生命周期？
14. Spring 如何处理线程并发问题？
15. 核心容器（应用上下文）模块的理解？
16. 为什么说 Spring 是一个容器？
17. Spring 的优点？
18. Spring 框架中的单例 Beans 是线程安全的么？
19. Spring 框架中有哪些不同类型的事件？
20. IoC 的优点是什么？



