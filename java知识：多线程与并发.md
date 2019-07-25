# java知识：多线程与并发

### 1、进程与线程

​		**进程是资源分配的最小单位，线程是cpu调度的最小单位。线程也被称为轻量级进程。**

- 所有与进程相关的资源，都被记录在PCB中

- 进程是抢占处理及的调度单位；线程属于某个进程，共享其资源

  **一个 Java 程序的运行是 main 线程和多个其他线程同时运行**。

（1）**从 JVM 角度说进程和线程之间的关系**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1563161014609.png"  style="zoom:50%">

​		从上图可以看出：一个进程中可以有多个线程，多个线程共享进程的**堆**和**方法区 (JDK1.8 之后的元空间)资源，但是每个线程有自己的程序计数器**、**虚拟机栈** 和 **本地方法栈**。

### 2、Thread中的start和run方法的区别

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1563172280028.png"  style="zoom:50%">

- 调用start()方法会创建一个新的子线程并启动

- run()方法只是Thread的一个普通方法的调用，还是在主线程里执行。

### 3、Thread和Runnable是什么关系？

​		Thread是实现了Runnable接口的类，是的run支持多线程。

​		因java类的单一继承原则，推荐多使用Runnable接口

### 4、如何给run()方法传参？

- 构造函数传参
- 成员变量传参
- 回调函数传参

### 5、如何实现处理线程的返回值？

​		实现的方式主要有三种：

- **主线程等待法**

```java
/*
private String value;
public void run() {
    try {
        Thread.currentThread().sleep(5000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    value = "we have data now";
}*/

CycleWait cw = new CycleWait();
Thread t = new Thread(cw);
t.start();
  while (cw.value == null){
          Thread.currentThread().sleep(100);//如果value一直为空，则线程一直sleep
        }
```

- **使用Thread类的join()阻塞当前线程，以等待子线程处理完毕**

```java
t.join();
```

- **通过Callable接口实现：通过FutureTask Or 线程池获取**

### 6、线程的状态？

- 新建（NEW）：创建后尚未启动的线程的状态

- 运行（Runnable）：包含Running和Ready

- 无限期等待（Waiting）：不会被分配CPU执行时间，需要显式被唤醒

  > 没有设置Timeout参数的Object.wait()方法。
  >
  > 没有设置Timeout参数的Thread.join()方法。
  >
  > LockSupport.park()方法。

- 限期等待（Timed Waiting）：在一定时间后会由系统自动唤醒

  > Thread.sleep()方法。
  >
  > 设置了Timeout参数的Object.wait()方法。
  >
  > 设置了Timeout参数的Thread.join()方法。
  >
  > LockSupport.parkNanos()方法。
  >
  > LockSupport.parkUntil()方法。

- 阻塞（blocked）：等待获取排它锁

- 结束：已终止线程的状态，线程已经结束执行

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1563628836226.png"  style="zoom:50%">

### 7、sleep和wait

- sleep是Thread类的方法，wait是Object类中定义的方法

- sleep方法可以在任何地方使用

- wait方法只能在synchronized方法或者synchronized块中使用

  **最本质的区别**

- Thread.sleep只会让出CPU，不会导致锁行为的改变（不会释放锁）
- Object.wait不仅让出CPU，还会释放已经占有的同步资源锁

### 8、notify和notifyAll的区别

- notifyAll会让所有处于等待池的线程全部进入锁池去竞争获取锁的机会
- notify会随机选取一个处于等待池中的线程进入锁池去竞争获取锁的机会。

### 9、yield函数

当调用Thread.yield()函数时，会给线程调度器一个当前线程愿意让出CPU使用的暗示，但是线程调度器可能会忽略这个暗示

### 10、中断函数interrupt()

- 已经被抛弃的方法

  > 通过调用stop()方法停止线程

- 目前使用的方法

  > **调用interrupt()，通知线程应该中断了**
  >
  > 1、如果线程处于被阻塞状态，那么线程将立即退出被阻塞状态，并抛出一个InterruptedException异常
  >
  > 2、如果线程处于正常活动状态，那么会将该线程的中断标志设置为true。被设置中断标志的线程将继续正常运行，不受影响
  >
  > **需要被调用的线程配合中断**
  >
  > 1、在正常运行任务时，经常检查本线程的中断标志位，如果被设置了中断标志就自行停止线程。
  >
  > 2、如果线程处于正常活动状态，那么会将该线程的中断标志设置为true。被设置中断标志的线程将继续正常运行，不受影响

### 11、[synchronized](http://www.mamicode.com/info-detail-1770568.html)

**线程安全问题的主要诱因：**

- 存在共享数据（也称临界资源）

- 存在多条线程共同操作这些共享数据

  **解决问题的根本办法：**

  ​		同一时刻有且只有一个线程在操作共享数据，其他线程必须等到该线程处理完数据后再对贡献数据进行操作。

**互斥锁的特性：**

​		**互斥性**：即在同一时间只允许一个线程持有某个对象锁，通过这种特性来实现多线程的协调机制，这样同一时间只有一个线程对需要同步的代码块（复合操作）进行访问。互斥性也称为操作的原子性。

​		**可见性**：必须确保在锁被释放之前，对共享变量所做的修改，对于随后获得该锁的另一个线程是可见的（即在获得锁时应该获得最新共享变量的值），否则另一个线程可能是在本地缓存的某个副本上继续操作，从而引起不一致。（一致性？？？paxos？？？raft？？？）

**根据获取锁的分类：获取对象锁和获取类锁**

- 获取对象锁的两种用法

  > （1）同步代码块（synchronized(this)，synchronized(类实例对象)），锁是小括号()中的实例对象。
  >
  > （2）同步非静态方法（synchronized method），锁是当前对象的实例对象。

- 获取类锁的两种用法

  > 
  >
  > （1）同步代码块（synchronized(类.class)），锁是小括号()中的类对象（Class对象）。
  >
  > （2）同步静态方法（synchronized static method）,锁是当前对象的类对象（Class对象）

**对象锁和类锁的总结：**

​		（1）有线程访问对象的同步代码块时，另外的线程可以访问该对象的非同步代码块；

​		（2）若锁住的是同一个对象，一个线程在访问对象的同步代码块时，另一个访问对象的同步代码块的线程会被阻塞；

​		（3）若锁住的是同一个对象，一个线程在访问对象的同步方法时，另一个访问对象的同步方法的线程会被阻塞；

​		（4）若锁住的是同一个对象，一个线程在访问对象的同步代码块时，另一个访问对象的同步方法的线程会被阻塞；，反之亦然；

​		（5）同一个类的不同对象的对象锁互不干扰；

​		（6）类锁由于也是一种特殊的对象锁，因此表现和上述1、2、3、4一致，而由于一个类只有一把对象锁，所以同一个类的不同对象使用类锁将会是同步的；

​		（7）类锁和对象锁互不干扰。

### 12、synchronized的底层实现原理

（1）实现synchronized的基础

- **java对象头**
- **Monitor**

（2）对象在内存中的布局

- **对象头**

- 实例数据

- 对齐填充

**对象头的结构：**

java的对象头由以下三部分组成：

> 1、Mark Word
>
> 2、指向类的指针
>
> 3、数组长度（只有数组对象才有）

**Mark Word**
		Mark Word记录了对象和锁有关的信息，当这个对象被synchronized关键字当成同步锁时，围绕这个锁的一系列操作都和Mark Word有关。

​		Mark Word在32位JVM中的长度是32bit，在64位JVM中长度是64bit。

​		Mark Word在不同的锁状态下存储的内容不同，在32位JVM中是这么存的：

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1563801542838.png"  style="zoom:50%">

JVM一般是这样使用锁和Mark Word的：

> 1，当没有被当成锁时，这就是一个普通的对象，Mark Word记录对象的HashCode，锁标志位是01，是否偏向锁那一位是0。
>
> 2，当对象被当做同步锁并有一个线程A抢到了锁时，锁标志位还是01，但是否偏向锁那一位改成1，前23bit记录抢到锁的线程id，表示进入偏向锁状态。
>
> 3，当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步锁的代码。
>
> 4，当线程B试图获得这个锁时，JVM发现同步锁处于偏向状态，但是Mark Word中的线程id记录的不是B，那么线程B会先用CAS操作试图获得锁，这里的获得锁操作是有可能成功的，因为线程A一般不会自动释放偏向锁。如果抢锁成功，就把Mark Word里的线程id改为线程B的id，代表线程B获得了这个偏向锁，可以执行同步锁代码。如果抢锁失败，则继续执行步骤5。
>
> 5，偏向锁状态抢锁失败，代表当前锁有一定的竞争，偏向锁将升级为轻量级锁。JVM会在当前线程的线程栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时在对象锁Mark Word中保存指向这片空间的指针。上述两个保存操作都是CAS操作，如果保存成功，代表线程抢到了同步锁，就把Mark Word中的锁标志位改成00，可以执行同步锁代码。如果保存失败，表示抢锁失败，竞争太激烈，继续执行步骤6。
>
> 6，轻量级锁抢锁失败，JVM会使用自旋锁，自旋锁不是一个锁状态，只是代表不断的重试，尝试抢锁。从JDK1.7开始，自旋锁默认启用，自旋次数由JVM决定。如果抢锁成功则执行同步锁代码，如果失败则继续执行步骤7。
>
> 7，自旋锁重试之后如果抢锁依然失败，同步锁会升级至重量级锁，锁标志位改为10。在这个状态下，未抢到锁的线程都会被阻塞。

**Monitor（管程）**：**每个java对象天生自带了一把看不见的锁**

​		Monitor锁的竞争、获取与释放

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1563866821421.png"  style="zoom:50%">

### 13、自旋锁

- 许多情况下，共享数据的所状态持续时间较短，切换线程不值得。
- 通过让线程执行忙循环等待锁的释放，不让出cpu。
- 缺点：若锁被其他线程长时间占用，会带来许多性能上的开销·

### 14、自适应自旋锁

- 自旋的次数不再固定
- 由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定

### 15、锁消除

JIT编译时，对运行上下文进行扫描，去除不可能存在竞争的锁。

### 16、锁粗化

通过扩大锁的范围，避免反复的加锁解锁

###  17、[synchronized的四种状态](<https://blog.csdn.net/lengxiao1993/article/details/81568130>)

无锁、偏向锁、轻量级锁、重量级锁

**锁膨胀方向：无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁**

**偏向锁：减少同一线程获取锁的代价**

> 大多数情况下，锁不存在多线程竞争，总是由同一线程多次获得

​		***核心思想：***

​		如果一个线程获得了锁，那么锁就进入了偏向模式，此时Mark Word的结构也变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作，即获取锁的过程只需要检查Mark Word的所标记位为偏向锁以及当前线程ID等于Mark Word的ThreadID即可，这样就省去了大量有关锁申请的操作。

### 18、轻量级锁

​		轻量级锁是由偏向锁升级来的，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁争用的时候，偏向锁就会升级为轻量级锁。

​		**适用场景：**线程交替执行同步块

​		若存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁

### 19、[锁的内存语义](https://blog.csdn.net/bohu83/article/details/80798930)

​	当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。

​    当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须要从主内存中去读取共享变量。

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1563871848782.png"  style="zoom:50%">



<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1563872055190.png"  style="zoom:50%">

​															**开销从上到下递增**

### 20、ReenTrantLock

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

```java
public class LockExample {

    private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock(); // 确保释放锁，从而避免发生死锁。
        }
    }
}
```

```java
public static void main(String[] args) {
    LockExample lockExample = new LockExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> lockExample.func());
    executorService.execute(() -> lockExample.func());
}
```

**（1） 锁的实现**

​		synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

**（2） 性能**

​		新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

**（3） 等待可中断**

​		当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

​		ReentrantLock 可中断，而 synchronized 不行。

**（4） 公平锁**

​		公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

​		synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**（5） 锁绑定多个条件**

​		一个 ReentrantLock 可以同时绑定多个 Condition 对象。

### 21、[线程池](https://www.jianshu.com/p/50fffbf21b39)

**（1）为什么要用线程池？**

​		线程池提供了一种限制和管理资源（包括执行一个任务）。 每个线程池还维护一些基本统计信息，例如已完成任务的数量。

​		这里借用《Java并发编程的艺术》提到的来说一下使用线程池的好处：

- **降低资源消耗。** 通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度。** 当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- **提高线程的可管理性。** 线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

**（2）实现Runnable接口和Callable接口的区别**

​		如果想让线程池执行任务的话需要实现的Runnable接口或Callable接口。 Runnable接口或Callable接口实现类都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行。两者的区别在于 Runnable 接口不会返回结果但是 Callable 接口可以返回结果。

​		**备注：** 工具类`Executors`可以实现`Runnable`对象和`Callable`对象之间的相互转换。（`Executors.callable（Runnable task）`或`Executors.callable（Runnable task，Object resule）`）。

**（3）执行execute()方法和submit()方法的区别是什么呢？**

​		1) **execute() 方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**

​		2) **submit() 方法用于提交需要返回值的任务。线程池会返回一个Future类型的对象，通过这个Future对象可以判断任务是否执行成功**，并且可以通过future的get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用 `get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

**（4）如何创建线程池** 

​		《阿里巴巴Java开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

> Executors 返回线程池对象的弊端如下：
>
> - **FixedThreadPool 和 SingleThreadExecutor** ： 允许请求的队列长度为 Integer.MAX_VALUE ，可能堆积大量的请求，从而导致OOM。
> - **CachedThreadPool 和 ScheduledThreadPool** ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致OOM。

**方式一：通过构造方法实现** 

![ThreadPoolExecutoræé æ¹æ³](https://camo.githubusercontent.com/c1a87ea139bc0379f5c98484416594843ff29d6d/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f546872656164506f6f6c4578656375746f722545362539452538342545392538302541302545362539362542392545362542332539352e706e67)

**方式二：通过Executor 框架的工具类Executors来实现** 我们可以创建三种类型的ThreadPoolExecutor：

- **FixedThreadPool** ： 该方法返回一个固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。
- **SingleThreadExecutor：** 方法返回一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。
- **CachedThreadPool：** 该方法返回一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。