# JVM

## 1. 详细JVM内存模型？ 

JVM内存模型一共分为：程序计数器、本地方法栈、虚拟机栈、方法区、堆。 

## 2. 什么情况会出现内存溢出？内存泄露？ 

内存溢出： 

堆内存溢出，会抛出heap space oom，此时堆内存不足以分配新的对象实例； 

方法区内存溢出，会抛出permgen space oom，此时class对象太多或者jsp太多，或者字符串常量过多，方法区不足以分配新的内存空间； 

虚拟机栈内存溢出，单线程下抛出stackoverflow，多线程抛出oom:unable create native thread，程序中线程过多，可创建线程数小，减小虚拟机栈的大小。 

内存泄露： 

未能及时GC对象，这部分对象既不能回收又不能使用。长生命周期的对象持有短生命周期对象的引用。尽量使用局部变量，或者在对象使用完过后赋值null。 

## 3. Java线程栈 

每个线程有一个线程栈，包括主线程。创建一个新的线程，即会产生一个新的线程栈。具有多个线程栈时，会并行执行。 

## 4. JVM 年轻代到年老代的晋升过程的判断条件是什么？ 

对象分配在年轻代上，每经历一次minor gc其年龄就会+1，达到一定的阈值时会晋升到老年代。可以通过MaxTenuringThreshold设置阈值。 

## 5. FullGC频繁，怎么排查？ 

JVM参数加上XX:HeapDumpBeforeFullGC，也就是在FullGC前记录堆内存的情况。dump文件显示程序中有大量的大对象，原因是程序启动后将数据库中的数据全部加载进了内存中，导致出现堆的内存较小无法继续分配持续FullGC，并出现了停止响应等问题。 

## 6. JVM垃圾回收机制，何时触发MinorGC等操作？ 

分代垃圾回收机制：不同的对象生命周期不同，针对不同的对象采用不同的GC方式。 

一共分为：年轻代、老年代、永久代。 

年轻代：新创建的对象会分配在年轻代。 

老年代：经过N次年轻代的minor gc仍然存活的对象会进入老年代；或者大对象的创建会直接进入老年代。 

永久代：字符串常量、静态文件。 

新生代的gc称为minor gc，只会在新生代上产生gc；老年代的gc称为full gc会发生在整个堆上。 

触发条件：堆内存不足；程序较为空闲时会触发gc线程执行。 

## 7. JVM 中一次完整的 GC 流程（从 ygc 到 fgc）是怎样的 

对象优先在年轻代中分配，若没有足够的空间则进行minor gc，大对象直接进入老年代，长期存活的对象也直接进入老年代。对象如果在新生代中的一次minor gc后存活，则其年龄+1，如果到了阈值15则进入老年代，阈值可通过MaxTenuringThreshold设置。 

各种回收器，各自优缺点，重点CMS、G1 

串行收集器：顾名思义会使用串行的方式进行gc，会暂定所有线程，单线程。 

并行收集器：同上也会暂停所有线程，不过会使用多线程的方式进行gc。 

CMS：并发标记清除。发生在老年代。一共经历4个过程：初始标记、并发标记、重新标记、并发清除，在初始标记和重新标记阶段会暂停线程，不过时间较短，采用的是标记-清除算法。 

G1：jdk7默认的gc收集器。发生在新生代以及老年代。 

## 7. 各种回收算法。 

标记-清除：标记出需要被gc的对象并直接清除。会造成空间不连续，碎片化。 

标记-整理：标记并清除需要被gc的对象后会将存活的对象整理为连续的。 

复制：将一个内存区域分成两块，其中一块进行gc时，将存活的对象复制放入另一块空间。 
