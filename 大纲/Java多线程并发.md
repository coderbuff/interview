# Java多线程并发

## 1. synchronized的实现原理以及锁优化？ 

Java中每个对象有一个监视器（monitor），synchronized关键字就是使用指令“monitorenter”、“monitorexit”获取对象监视器锁和释放监视器锁。 

**关于synchronized的锁优化，需理解偏向锁、轻量级锁、重量级锁**

## 2. synchronized修饰代码块、普通方法、静态方法有什么区别。 

修饰代码块获取的是指定的对象监视器。 

修饰普通代码块并未显示使用monitorenter指令，而是通过标志位ACC_SYNCHRONIZED标志位来判断是否由synchronized修饰，获取的是该实例对象的监视器。 

修饰静态方法同普通方法，不同的是获取的是这个类的监视器。 

## 3. volatile 关键字及其实现原理？ 

1. 内存可见性； 

2. 不可重排序。 

由于Java内存模型的原因，Java中提供了主内存和线程的工作内存，针对普通变量的赋值和取值是和线程工作内存进行交互，而volatile修饰后则是会从主内存中刷新，保证工作内存和主内存数据一致。 

read、load、use；assign、store、save。 

重排序指的是JVM在对其进行优化时，会在不影响逻辑结果的情况下会对执行顺序有所改变。加入volatile关键字修饰后，在其指令中会形成内存屏障，在该关键字之前的操作不允许重排序。 

## 4. Java 的信号灯 

Semaphore。使用信号灯Semaphore可控制线程访问资源的个数，通过acquire获取一个许可，没有就等待，通过release释放一个许可。内部通过AQS（AbstractQueuedSynchronizer）实现，控制数赋值给AQS中的state值，成功获得许可则状态-1，如果为0，线程此时进入AQS的等待队列中，创建Semaphore时可指定它的公平性，默认非公平。 

## 5. 怎么实现所有线程在等待某个事件的发生才会去执行？ 

使用CountDownLatch等待某一个指定的事件发生后，才让多个等待的线程继续执行。 

## 6. CountDownLatch和CyclicBarrier的用法，以及相互之间的差别？ 

CountDownLatch是闭锁，等待事件发生的同步工具类。采用减计数方式，为0时释放所有线程。计数器设为1，当某个事件未到来时，线程await阻塞，某个事件发生后调用countDown方法计数器-1=0，此时线程被唤醒继续执行。计数器不可重置。 

CyclicBarrier是栅栏，采用加计数方式。调用await计数器+1，当计数达到某个设置值时，释放所有等待的线程，用于等待线程的全部执行。计数器可置0。 

## 7. synchronized 和 lock 有什么区别？ 

synchronized是一个关键字，jvm实现；lock是一个类。 

synchronized不需要显示释放锁，退出同步代码块或者同步方法、抛出异常时会释放锁；lock需要显示释放。 

synchronized是阻塞式的霍曲锁；lock可以采取非阻塞方式。 

synchronized可重入、不可中断、非公平；lock可重入、可中断、可公平。 

## 8. CAS？CAS 有什么缺陷，如何解决？ 

compare and swap。比较和替换，使用一个期望值和当前值进行比较，如果当前变量的值和我们期望的值相等，就使用一个新值替换当前变量的值。 
缺陷，只对值有效，而不会判断当前值是否经历了几次修改，加入版本控制编号（此处不理解可参考Mysql的乐观锁实现）

## 9. 介绍ConcurrenHashMap？ 

线程安全的HashMap，与Hashtable在整个散列表上加锁不同的是，ConcurrentHashMap采用的是分段锁。将整个散列表分为几个部分，在不同部分加锁，称之为分段锁，key散列到不同的段可以并行存储互不影响，只有散列到同一个段上的时候才会加锁互斥。段的个数会根据设置的concurrentLevel来确定，concurrentLevel默认=16，段的个数会大于或等于concurrentLevel最小的2次幂。 

put的主要逻辑也是：1.定位segment并确保segment已经初始化；2.调用segment的put方法。 

## 10. 线程池的种类，区别和使用场景？ 

JDK为我们提供了4种线程池： 

newFixedThreadPool：固定线程池，根据传入的参数，线程池的线程数量是固定的； 

corePoolSize = maximumPoolSize = nThread。采用无界阻塞队列。LinkedBlockingQueue。 

任务提交到线程池，使用池中的线程，如果池中满了的话则进入阻塞队列。 

适用于执行长期的任务。 

newSingleThreadPool：只包含一个线程的线程池； 

corePoolSize = maximumPoolSize = 1。 

适用于一个任务一个任务执行的场景。 

newCachedTheradPool：可创建无限量的线程池，如果提交任务的速度大于任务处理的速度，则会无限量的创建线程； 

其corePoolSize核心线程池为0，任务直接提到到同步队列，执行任务时会从maximumPoolSize线程池中寻找可用的线程，如果没有则创建线程。如果提交任务的速度大于任务处理的速度，则会无限量的创建线程；如果有线程的空闲时间超过指定大小，则线程会被销毁。 

适用于执行短期异步的小程序，或则负载较轻的小程序。执行时间长的任务会不断创建线程。 

newScheduledThreadPool：可以定时执行任务的线程池。 

corePoolSize为传递进来的参数，maximumPoolSize为Integer.MAX_VALUE。dWorkQueue() 一个按超时时间升序排序的队列。 

适用于周期性执行任务的场景 

## 10. 分析线程池的实现原理和线程的调度过程？ 

1. 当线程池小于corePoolSize时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程。 

2. 当线程池达到corePoolSize时，新提交任务将被放入workQueue中，等待线程池中任务调度执行 

3. 当workQueue已满，且maximumPoolSize>corePoolSize时，新提交任务会创建新线程执行任务 

4. 当提交任务数超过maximumPoolSize时，新提交任务由RejectedExecutionHandler处理 

5. 当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程 

6. 当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭 

## 11. 线程池如何调优，最大数目如何确认？ 

根据业务场景，限制最大线程数以及最小线程数，包括工作的队列的选择。最大数目最好不超过cpu核心数。 

## 12. ThreadLocal原理，用的时候需要注意什么？ 

ThreadLocal本地变量，各线程直接并不需要共享此变量。 

在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals并未实现Map接口但也是初始化一个16个大小的Entry数组，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。 

ThreadLocal可能导致内存泄漏。key被保存到了WeakReference对象中。使用线程池时，线程可能并不会被销毁，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。 

## 13. Condition接口及其实现原理 

Condition接口提供了类似Object的监视器方法，与Lock配合可以实现等待/通知模式。Condition定义了等待/通知两种类型的方法，当前线程调用这些方法时，需要提前获取到Condition对象关联的锁。Condition对象是由Lock对象（调用Lock对象的newCondition()方法）创建出来的，换句话说，Condition是依赖Lock对象的。一般都会将Condition对象作为成员变量。当调用await()方法后，当前线程会释放锁并在此等待，而其他线程调用Condition对象的signal()方法，通知当前线程后，当前线程才从await()方法返回，并且在返回前已经获取了锁。 

ConditionObject是同步器AbstractQueuedSynchronizer的内部类，因为Condition的操作需要获取相关联的锁，所以作为同步器的内部类也较为合理。每个Condition对象都包含着一个队列，该队列是Condition对象实现等待/通知功能的关键。 

## 14. Fork/Join框架的理解 

用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。 

第一步分割任务。首先我们需要有一个fork类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停的分割，直到分割出的子任务足够小。 

第二步执行任务并合并结果。分割的子任务分别放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。 

Fork/Join使用两个类来完成以上两件事情： 

RecursiveAction：用于没有返回结果的任务。 

RecursiveTask ：用于有返回结果的任务。 

ForkJoinPool ：ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。 

## 15. 阻塞队列以及各个阻塞队列的特性 

ArrayBlockingQueue ：有界阻塞队列，一旦指定了队列的长度，则队列的大小不能被改变  

LinkedBlockingQueue ：可以通过构造方法设置capacity来使得阻塞队列是有界的，也可以不设置，则为无界队列  
