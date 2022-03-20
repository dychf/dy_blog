---
title: Java并发
tags: Java 并发 多线程
date: 2022-02-15
---

Java并发

## 并发和并行的区别

- **并发：** 同一时间段，多个任务都在执行 (单位时间内不一定同时执行)；
- **并行：** 单位时间内，多个任务同时执行。

## Java的进程与线程

进程是程序的一次执行过程，是**系统运行程序的基本单位**，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

**总结： 线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。线程执行开销小，但不利于资源的管理和保护；而进程正相反。**

## 线程和进程的区别

    线程具有许多传统进程所具有的特征，故又称为轻型进程(Light—Weight Process)或进程元；而把传统的进程称为重型进程(Heavy—Weight Process)，它相当于只有一个线程的任务。在引入了线程的操作系统中，通常一个进程都有若干个线程，至少包含一个线程。

**根本区别**：进程是操作系统资源分配的基本单位，而线程是处理器任务调度和执行的基本单位

**资源开销**：每个进程都有独立的代码和数据空间（程序上下文），程序之间的切换会有较大的开销；线程可以看做轻量级的进程，同一类线程共享代码和数据空间，每个线程都有自己独立的运行栈和程序计数器（PC），线程之间切换的开销小。

**包含关系**：如果一个进程内有多个线程，则执行过程不是一条线的，而是多条线（线程）共同完成的；线程是进程的一部分，所以线程也被称为轻权进程或者轻量级进程。

**内存分配**：同一进程的线程共享本进程的地址空间和资源，而进程之间的地址空间和资源是相互独立的。

**影响关系**：一个进程崩溃后，在保护模式下不会对其他进程产生影响，但是一个线程崩溃整个进程都死掉。所以多进程要比多线程健壮。

**执行过程**：每个独立的进程有程序运行的入口. 顺序执行序列和程序出口。但是线程不能独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制，两者均可并发执行

## 创建线程的四种方式

- 继承 Thread 类；
- 实现 Runnable 接口；
- 实现 Callable 接口；
- 使用线程池例如用Executor框架；
- 使用匿名内部类方式。

```java
public class CreateRunnable {
    public static void main(String[] args) {
        //创建多线程创建开始
        Thread thread = new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("i:" + i);
                }
            }
        });
        thread.start();
    }
}
```

### runnable 和 callable 有什么区别

**相同点：**

- 都是接口
- 都可以编写多线程程序
- 都采用Thread.start()启动线程

**主要区别：**

- Runnable 接口 run 方法无返回值；Callable 接口 call 方法有返回值，是个泛型，和 Future 、 FutureTask 配合可以用来获取异步执行的结果
- Runnable 接口 run 方法只能抛出运行时异常，且无法捕获处理；Callable 接口 call 方法允许抛出异常，可以获取异常信息 注：Callalbe接口支持返回执行结果，需要调用 FutureTask.get() 得到，此方法会阻塞主进程的继续往下执行，如果不调用不会阻塞。

### Future 和 FutureTask 有什么区别

    Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。总的来说Future提供了三种功能：

- 判断任务是否完成
- 取消任务
- 获取返回结果

    Future是一个接口，是无法生成一个实例的，所以又有了FutureTask。FutureTask实现了RunnableFuture接口，RunnableFuture接口又实现了Runnable接口和Future接口。所以FutureTask既可以被当做Runnable来执行，也可以被当做Future来获取Callable的返回结果。

## 线程的生命周期和状态

![Java 线程的状态 ](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

### 上下文切换

线程在执行过程中会有自己的运行条件和状态（也称上下文），比如上文所说到过的程序计数器，栈信息等。当出现如下情况的时候，线程会从占用 CPU 状态中退出。

- 主动让出 CPU，比如调用了 `sleep()`, `wait()` 等。
- 时间片用完，因为操作系统要防止一个线程或者进程长时间占用CPU导致其他线程或者进程饿死。
- 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞。
- 被终止或结束运行

这其中前三种都会发生线程切换，线程切换意味着需要保存当前线程的上下文，留待线程下次占用 CPU 的时候恢复现场。并加载下一个将要占用 CPU 的线程上下文。这就是所谓的 **上下文切换**。

上下文切换是现代操作系统的基本功能，因其每次需要保存信息恢复信息，这将会占用 CPU，内存等系统资源进行处理，也就意味着效率会有一定损耗，如果频繁切换就会造成整体效率低下。

## 线程死锁及其预防

线程死锁描述的是这样一种情况：多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。

死锁必须具备以下四个条件：

1. **互斥条件**：该资源任意一个时刻只由一个线程占用。
2. **请求与保持条件**：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. **不剥夺条件**：线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. **循环等待条件**：若干进程之间形成一种头尾相接的循环等待资源关系。

**如何预防死锁？** 破坏死锁的产生的必要条件即可：

1. **破坏请求与保持条件** ：一次性申请所有的资源。
2. **破坏不剥夺条件** ：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
3. **破坏循环等待条件** ：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

## 请说出与线程同步以及线程调度相关的方法

- wait()：使一个线程处于等待（阻塞）状态，并且释放所持有的对象的锁；
- sleep()：使一个正在运行的线程处于睡眠状态，是一个静态方法，调用此方法要处理 InterruptedException 异常；
  - 如果sleep()不是静态的话就可以sleep别的线程，造成多线程问题
- notify()：唤醒一个处于等待状态的线程，当然在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由 JVM 确定唤醒哪个线程，而且与优先级无关；
- notifyAll()：唤醒所有处于等待状态的线程，该方法并不是将对象的锁给所有线程，而是让它们竞争，只有获得锁的线程才能进入就绪状态；

### sleep() 和 wait() 有什么区别？

```
两者都可以暂停线程的执行
```

- 类的不同：sleep() 是 Thread线程类的静态方法，wait() 是 Object类的方法。
- 是否释放锁：sleep() 不释放锁；wait() 释放锁。
- 用途不同：Wait 通常被用于线程间交互/通信，sleep 通常被用于暂停执行。
- 用法不同：wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify() 或者 notifyAll() 方法。sleep() 方法执行完成后，线程会自动苏醒。或者可以使用wait(long timeout)超时后线程会自动苏醒。

### Java 中 interrupted 和 isInterrupted 方法的区别？

- interrupt：用于中断线程。调用该方法的线程的状态为将被置为”中断”状态。

  注意：线程中断仅仅是置线程的中断状态位，不会停止线程。需要用户自己去监视线程的状态为并做处理。支持线程中断的方法（也就是线程中断后会抛出 interruptedException 的方法）就是在监视线程的中断状态，一旦线程的中断状态被置为“中断状态”，就会抛出中断异常。
- interrupted：是静态方法，查看当前中断信号是 true 还是 false 并且清除中断信号。如果一个线程被中断了，第一次调用 interrupted 则返回 true，第二次和后面的就返回 false 了。
- isInterrupted：是可以返回当前中断信号是 true 还是 false ，与 interrupt 最大的差别

## synchronized 关键字

**`synchronized` 关键字解决的是多个线程之间访问资源的同步性，`synchronized`关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。**

### 如何使用 **synchronized 关键字**

- **修饰实例方法:** 作用于当前对象实例加锁，进入同步代码前要获得 **当前对象实例的锁**。
- **修饰静态方法:** 也就是给当前类加锁，会作用于类的所有对象实例 ，进入同步代码前要获得 **当前 class 的锁**。因为静态成员不属于任何一个实例对象，是类成员（ _static 表明这是该类的一个静态资源，不管 new 了多少个对象，只有一份_）。所以，如果一个线程 A 调用一个实例对象的非静态 `synchronized` 方法，而线程 B 需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象，**因为访问静态 `synchronized` 方法占用的锁是当前类的锁，而访问非静态 `synchronized` 方法占用的锁是当前实例对象锁**。
- **修饰代码块**：指定加锁对象，对给定对象/类加锁。`synchronized(this|object)` 表示进入同步代码前要获得**给定对象的锁**。`synchronized(类.class)`表示进入同步代码前要获得**当前 class 的锁。**

**总结：**

- `synchronized` 关键字加到 `static` 静态方法和 `synchronized(class)` 代码块上都是是给 Class 类上锁。
- `synchronized` 关键字加到实例方法上是给对象实例上锁。
- 尽量不要使用 `synchronized(String a)` 因为 JVM 中，字符串常量池具有缓存功能！

### synchronized 关键字的底层原理

**`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。**

当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 **对象监视器 `monitor`** 的持有权。

> 在 Java 虚拟机(HotSpot)中，Monitor 是基于 C++实现的，由ObjectMonitor实现的。每个对象中都内置了一个 `ObjectMonitor`对象。
>
> 另外，`wait/notify`等方法也依赖于 `monitor`对象，这就是为什么只有在同步的块或者方法中才能调用 `wait/notify`等方法，否则会抛出 `java.lang.IllegalMonitorStateException`的异常的原因。

在执行 `monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

**`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。JVM 通过该 `ACC_SYNCHRONIZED` 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。**

**两者的本质都是对对象监视器 monitor 的获取。**

### JDK1.6 之后的 synchronized 关键字底层做了哪些优化，可以详细介绍一下这些优化吗？

JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

锁主要存在四种状态，级别从低到高依次是:

1. 无锁状态
2. 偏向锁状态
3. 轻量级锁状态
4. 重量级锁状态

锁可以升级, 但不能降级. 即:

    无锁 -> 偏向锁 -> 轻量级锁（ CAS+自旋） -> 重量级锁（synchronized）是单向的。

### 自旋锁

自旋：竞争锁的线程暂时获取不到锁，并不进入阻塞状态，而是占用cpu自旋一段时间（可以设定一个自旋等待的最大时间），等持有锁的线程释放锁后立即获取锁，**从而避免用户线程和内核的切换带来的资源消耗**。

优点：在锁竞争不激烈的情况下且占用锁时间比较短，**自旋的消耗会小于线程阻塞挂起再唤醒的操作的消耗**（两次线程上下文切换）。

缺点：在锁竞争激烈的情况下，或者持有锁的线程需要长时间占用锁，就不适合用自旋锁，因为自旋锁在得到锁之前一直占用cpu做无用功，如果大量线程都在自旋等待这个锁，等待时间过长，就**会导致线程自旋消耗大于线程上下文切换得消耗**。

### 偏向锁

偏向锁：适用于不存在多线程竞争锁，并且锁总是由同一线程获得的场景，偏向锁的目的是在某个线程获得锁之后，消除这个线程锁重入（CAS）的开销，看起来让这个线程得到了偏护。**它通过消除资源无竞争情况下的同步，进一步提高程序运行性能。**

- CAS （比较并交换），是实现并发算法时用到的技术，Java并发包中的很多类都使用了CAS技术。
- CAS具体包括三个参数：**当前内存值V、旧的预期值A**、**即将更新的值B**，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。
- CAS 有效地说明了“ 我认为位置 V 应该包含值 A，如果真的包含A值，则将 B 放到这个位置，否则，不要更改该位置，只告诉我这个位置现在的值(A)即可。 ”整个比较并交换的操作是**原子操作**。

偏向锁获取：

1. 访问MarkWord中偏向锁的标识是否设置成1，锁标志位是否为01，确认为可偏向状态。
2. 若为可偏向状态，测试线程ID（Markword）是否指向当前线程，若是，进入步骤5，否则进入步骤3。
3. 如果线程ID（Markword）并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将MarkWord中线程ID设置为当前线程ID，然后执行5；如果竞争失败，执行4。
4. 如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。（撤销偏向锁的时候会导致stop the word）
5. 执行同步代码。

释放：

线程不会主动释放偏向锁，只有当有其他线程竞争该锁时才会释放。

适用场景：

始终只有一个线程在执行同步块，在它没有执行完释放锁之前，没有其它线程去执行同步块。

开启关闭：

开启偏向锁：-XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0
关闭偏向锁：-XX:-UseBiasedLocking

### 轻量级锁

当出现有两个线程来竞争锁的话, 那么偏向锁就失效了, 此时锁就会膨胀, 升级为轻量级锁。

加锁：

    线程在执行同步块之前，JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中，官方称为Displaced Mark Word。**然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，则自旋获取锁，当自旋获取锁仍然失败时，表示存在两条或两条以上的线程竞争同一个锁，则轻量级锁会膨胀成重量级锁。**

解锁：

    轻量级解锁时，会使用原子的CAS操作来将Displaced Mark Word替换回到对象头，如果成功，则表示同步过程已完成。如果失败，表示有其他线程尝试过获取该锁，表示当前锁存在竞争，锁就会膨胀成重量级锁。则要在释放锁的同时唤醒被挂起的线程。

### 偏向锁、轻量级锁和重量级锁的区别

|    锁    |                              优点                              |                      缺点                      |              适用场景              |
| :------: | :-------------------------------------------------------------: | :---------------------------------------------: | :--------------------------------: |
|  偏向锁  | 加锁和解锁不需要额外的消耗, 和执行非同步代码方法的性能相差无几. | 如果线程间存在锁竞争, 会带来额外的锁撤销的消耗. |  适用于只有一个线程访问的同步场景  |
| 轻量级锁 |            竞争的线程不会阻塞, 提高了程序的响应速度            |  如果始终得不到锁竞争的线程, 使用自旋会消耗CPU  | 追求响应时间, 同步快执行速度非常快 |
| 重量级锁 |                 线程竞争不适用自旋, 不会消耗CPU                 |             线程堵塞, 响应时间缓慢             | 追求吞吐量, 同步快执行时间速度较长 |

### synchronized与lock的区别

![这里写图片描述](https://img-blog.csdn.net/20180904143958577?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlZmVuZ2xpYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### synchronized 和 ReentrantLock 的区别

- **两者都是可重入锁**

同一个线程每次获取锁，锁的计数器都自增 1，所以要等到锁的计数器下降为 0 时才能释放锁。

- **synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API**

`synchronized` 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 `synchronized` 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。`ReentrantLock` 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。

- **ReentrantLock 比 synchronized 增加了一些高级功能**

相比 `synchronized`，`ReentrantLock`增加了一些高级功能。主要来说主要有三点：

1. **等待可中断** : `ReentrantLock`提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
2. **可实现公平锁** : `ReentrantLock`可以指定是公平锁还是非公平锁。而 `synchronized`只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的 `ReentrantLock(boolean fair)`构造方法来制定是否是公平的。
3. **可实现选择性通知（锁可以绑定多个条件）**: `synchronized`关键字与 `wait()`和 `notify()`/`notifyAll()`方法相结合可以实现等待/通知机制。`ReentrantLock`类当然也可以实现，但是需要借助于 `Condition`接口与 `newCondition()`方法。

> `Condition`是 JDK1.5 之后有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个 `Lock`对象中可以创建多个 `Condition`实例（即对象监视器），**线程对象可以注册在指定的 `Condition`中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用 `notify()/notifyAll()`方法进行通知时，被通知的线程是由 JVM 选择的，用 `ReentrantLock`类结合 `Condition`实例可以实现“选择性通知”** ，这个功能非常重要，而且是 Condition 接口默认提供的。而 `synchronized`关键字就相当于整个 Lock 对象中只有一个 `Condition`实例，所有的线程都注册在个身上。如果执行 `notifyAll()`方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而 `Condition`实例的 `signalAll()`方法只会唤醒注册在该 `Condition`实例中的所有等待线程。

## Java内存模型（JMM）及 volatile 关键字

    Java采用的是共享内存模型。在当前的 Java 内存模型下，线程可以把变量保存**本地内存**（比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成**数据的不一致**。

![JMM(Java内存模型)](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-8/0ac7e663-7db8-4b95-8d8e-7d2b179f67e8.png)

    要解决这个问题，就需要把变量声明为**volatile** ，这就指示 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

    所以，**volatile 关键字除了防止 JVM 的指令重排 ，还有一个重要的作用就是保证变量的可见性。**

### 并发编程的三个重要特性

1. **原子性** : 一个的操作或者多次操作，要么所有的操作全部都得到执行并且不会收到任何因素的干扰而中断，要么所有的操作都执行，要么都不执行。`synchronized` 可以保证代码片段的原子性。
2. **可见性** ：当一个变量对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。`volatile` 关键字可以保证共享变量的可见性。
3. **有序性** ：代码在执行的过程中的先后顺序，Java 在编译器以及运行期间的优化，代码的执行顺序未必就是编写代码时候的顺序。`volatile` 关键字可以禁止指令进行重排序优化。

### 什么是重排序

- 程序执行的顺序按照代码的先后顺序执行。
- 一般来说处理器为了提高程序运行效率，可能会对输入代码进行优化，进行重新排序（重排序），它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。
- 则因为重排序，他还可能执行顺序为（这里标注的是语句的执行顺序） 2-1-3-4，1-3-2-4 但绝不可能 2-1-4-3，因为这打破了依赖关系。
- 显然重排序对单线程运行是不会有任何问题，但是多线程就不一定了，所以我们在多线程编程时就得考虑这个问题了。

#### 重排序实际执行的指令步骤

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/17172bab5818fe21~tplv-t2oaga2asx-watermark.awebp)

1. 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
2. 指令级并行的重排序。现代处理器采用了指令级并行技术（ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
3. 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作可能是在乱序执行。

- 这些重排序对于单线程没问题，但是多线程都可能会导致多线程程序出现内存可见性问题。

### 说说 synchronized 关键字和 volatile 关键字的区别

`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

- **`volatile` 关键字**是线程同步的**轻量级实现**，所以 **`volatile `性能肯定比 `synchronized`关键字要好** 。但是 **`volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块** 。
- **`volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。**
- **`volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。**

## ThreadLocal

### ThreadLocal 简介

    通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。**如果每一个线程都想有自己的专属本地变量该如何解决呢？**可以使用JDK提供的 JDK 中的`ThreadLocal`类解决此问题。 **`ThreadLocal`类就是让每个线程绑定自己的值，可以将 `ThreadLocal`类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据。**

    **如果创建了一个 `ThreadLocal`变量，那么访问这个变量的每个线程都会有这个变量的本地副本，这也是 `ThreadLocal`变量名的由来。他们可以使用 `get（）` 和 `set（）` 方法来获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题。**

    再举个简单的例子：比如有两个人去宝屋收集宝物，这两个共用一个袋子的话肯定会产生争执，但是给他们两个人每个人分配一个袋子的话就不会出现这样的问题。如果把这两个人比作线程的话，那么`ThreadLocal` 就是用来避免这两个线程竞争的。

### ThreadLocal 原理

    从`Thread`类源代码可以看出该类中有一个 `threadLocals` 和 一个 `inheritableThreadLocals` 变量，它们都是 `ThreadLocalMap` 类型的变量，我们可以把 `ThreadLocalMap` 理解为 `ThreadLocal` 类实现的定制化的 `HashMap`。默认情况下这两个变量都是 null，只有当前线程调用 `ThreadLocal` 类的 `set`或 `get`方法时才创建它们，实际上调用这两个方法的时候，我们调用的是 `ThreadLocalMap`类对应的 `get()`、`set()`方法。

    **最终变量放在当前线程的 `ThreadLocalMap` 中，而不是 `ThreadLocal` 中，`ThreadLocal` 可以理解为是 `ThreadLocalMap`的封装，传递了变量值。** `ThrealLocal` 类中可以通过 `Thread.currentThread()`获取到当前线程对象后，直接通过 `getMap(Thread t)`可以访问到该线程的 `ThreadLocalMap`对象。

    **每个 `Thread`中都具备一个 `ThreadLocalMap`，而 `ThreadLocalMap`可以存储以 `ThreadLocal`为 key ，Object 对象为 value 的键值对。**

    比如我们在同一个线程中声明了两个`ThreadLocal` 对象的话，会使用 `Thread`内部都是使用仅有那个 `ThreadLocalMap` 存放数据的，`ThreadLocalMap`的 key 就是 `ThreadLocal`对象，value 就是 `ThreadLocal` 对象调用 `set`方法设置的值。

### ThreadLocal 内存泄露问题

    `ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用，而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收时 key 会被清理掉，而 value 不会被清理掉。这样 `ThreadLocalMap` 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用 `remove()`方法。

## 线程池的使用场景和好处，线程池参数分析

场景：线程池一般用于执行多个不相关联的耗时任务，没有多线程的情况下，任务顺序执行，使用了线程池的话可让多个不相关联的任务同时执行。

优势：

1. **降低资源消耗**：**重用**线程池的**线程**，**避免**因为线程的创建和销毁所带来的**性能开销**；
2. **提高响应速度**：有效**控制**线程池的**最大并发数**，**避免**大量的线程之间因**抢占系统资源而阻塞**；
3. **提高线程的可管理性**：能够**对线程进行简单的管理**，并提供一下特定的操作如：可以提供**定时**、**定期**、**单线程**、**并发数控制**等功能。

### `ThreadPoolExecutor` 3 个最重要的参数：

- `corePoolSize` ：核心线程数线程数定义了最小可以同时运行的线程数量。
- `maximumPoolSize` ：当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- `workQueue`：当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

### `ThreadPoolExecutor`其他常见参数:

1. `keepAliveTime`:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
2. `unit` : `keepAliveTime` 参数的时间单位。
3. `threadFactory` : executor 创建新线程的时候会用到。
4. `handler` : 饱和策略。关于饱和策略下面单独介绍一下。
   1. `ThreadPoolExecutor.AbortPolicy`： 抛出 `RejectedExecutionException`来拒绝新任务的处理。
   2. `ThreadPoolExecutor.CallerRunsPolicy` ： 调用执行自己的线程运行任务，也就是直接在调用 `execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
   3. `ThreadPoolExecutor.DiscardPolicy`不处理新任务，直接丢弃掉。
   4. `ThreadPoolExecutor.DiscardOldestPolicy`： 此策略将丢弃最早的未处理的任务请求。

### 线程池的执行原理

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/17172bac2446a113~tplv-t2oaga2asx-watermark.awebp)

提交一个任务到线程池中，线程池的处理流程如下：

1. 判断线程池里的核心线程是否都在执行任务，如果不是（核心线程空闲或者还有核心线程没有被创建）则创建一个新的工作线程来执行任务。如果核心线程都在执行任务，则进入下个流程。
2. 线程池判断工作队列是否已满，如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。
3. 判断线程池里的线程是否都处于工作状态，如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

### 如何确定线程池的线程数量

- 如果是CPU密集型应用，则线程池大小设置为N+1 （N为CPU总核数）
- 如果是IO密集型应用，则线程池大小设置为2N+1 （N为CPU总核数）
- **线程等待时间(IO)所占比例越高，需要越多线程**。
- **线程CPU时间所占比例越高，需要越少线程**。

一个系统最快的部分是CPU，所以决定一个系统吞吐量上限的是CPU。增强CPU处理能力，可以提高系统吞吐量上限。

但根据短板效应，真实的系统吞吐量并不能单纯根据CPU来计算。

那要提高系统吞吐量，就需要从“系统短板”（比如网络延迟、IO）着手：

1. 尽量提高短板操作的并行化比率，比如多线程下载技术
2. 增强短板能力，比如用NIO替代IO

### 执行 execute()方法和 submit()方法的区别是什么呢？

- **`execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**
- **`submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get(long timeout，TimeUnit unit)`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

### 构建线程池的方法有哪些？

#### 1. newCachedThreadPool

- **特点**：newCachedThreadPool创建一个可缓存线程池，如果当前线程池的长度超过了处理的需要时，它可以灵活的回收空闲的线程，当需要增加时， 它可以灵活的添加新的线程，而不会对池的长度作任何限制
- **缺点**：他虽然可以无线的新建线程，但是容易造成堆外内存溢出，因为它的最大值是在初始化的时候设置为 Integer.MAX_VALUE，一般来说机器都没那么大内存给它不断使用。当然知道可能出问题的点，就可以去重写一个方法限制一下这个最大值
- **总结**：线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

#### 2.newFixedThreadPool

- **特点**：创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。定长线程池的大小最好根据系统资源进行设置。
- **缺点**：线程数量是固定的，但是阻塞队列是无界队列。如果有很多请求积压，阻塞队列越来越长，容易导致OOM（超出内存空间）
- **总结**：请求的挤压一定要和分配的线程池大小匹配，定线程池的大小最好根据系统资源进行设置。如Runtime.getRuntime().availableProcessors()

```
Runtime.getRuntime().availableProcessors()方法是查看电脑CPU核心数量）
```

#### 3.newScheduledThreadPool

- **特点**：创建一个固定长度的线程池，而且支持定时的以及周期性的任务执行，类似于Timer（Timer是Java的一个定时器类）
- **缺点**：由于所有任务都是由同一个线程来调度，因此所有任务都是串行执行的，同一时间只能有一个任务在执行，前一个任务的延迟或异常都将会影响到之后的任务（比如：一个任务出错，以后的任务都无法继续）。

#### 4.newSingleThreadExecutor

- **缺点**：缺点的话，很明显，他是单线程的，高并发业务下有点无力
- **总结**：保证所有任务按照指定顺序执行的，如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它

### 几种常见的线程池

#### FixedThreadPool

    `FixedThreadPool` 被称为可重用固定线程数的线程池。通过相关源代码来看一下实现：

```java
   /**
     * 创建一个可重用固定数量线程的线程池
     */
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
```

    **从上面源代码可以看出新创建的 `FixedThreadPool` 的 `corePoolSize` 和 `maximumPoolSize` 都被设置为 nThreads，这个 nThreads 参数是我们使用的时候自己传递的。**

##### 执行任务过程介绍

`FixedThreadPool` 的 `execute()` 方法运行示意图：

![FixedThreadPool的execute()方法运行示意图](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93/FixedThreadPool.png)

**上图说明：**

1. 如果当前运行的线程数小于 corePoolSize， 如果再来新任务的话，就创建新的线程来执行任务；
2. 当前运行的线程数等于 corePoolSize 后， 如果再来新任务的话，会将任务加入 `LinkedBlockingQueue`；
3. 线程池中的线程执行完手头的任务后，会在循环中反复从 `LinkedBlockingQueue` 中获取任务执行；

##### 为什么不推荐使用 `FixedThreadPool`？

**`FixedThreadPool` 使用无界队列 `LinkedBlockingQueue`（队列的容量为 Integer.MAX_VALUE）作为线程池的工作队列会对线程池带来如下影响 ：**

1. 当线程池中的线程数达到 `corePoolSize` 后，新任务将在无界队列中等待，因此线程池中的线程数不会超过 corePoolSize；
2. 由于使用无界队列时 `maximumPoolSize` 将是一个无效参数，因为不可能存在任务队列满的情况。所以，通过创建 `FixedThreadPool`的源码可以看出创建的 `FixedThreadPool` 的 `corePoolSize` 和 `maximumPoolSize` 被设置为同一个值。
3. 由于 1 和 2，使用无界队列时 `keepAliveTime` 将是一个无效参数；
4. 运行中的 `FixedThreadPool`（未执行 `shutdown()`或 `shutdownNow()`）不会拒绝任务，在任务比较多的时候会导致 OOM（内存溢出）。

#### SingleThreadExecutor

`SingleThreadExecutor` 是只有一个线程的线程池。下面看看**SingleThreadExecutor 的实现：**

```java
 /**
     *返回只有一个线程的线程池
     */
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```

从上面源代码可以看出新创建的 `SingleThreadExecutor` 的 `corePoolSize` 和 `maximumPoolSize` 都被设置为 1.其他参数和 `FixedThreadPool` 相同。

##### 执行任务过程介绍

![SingleThreadExecutor的运行示意图](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93/SingleThreadExecutor.png)

**上图说明;**

1. 如果当前运行的线程数少于 corePoolSize，则创建一个新的线程执行任务；
2. 当前线程池中有一个运行的线程后，将任务加入 `LinkedBlockingQueue`
3. 线程执行完当前的任务后，会在循环中反复从 `LinkedBlockingQueue` 中获取任务来执行；

##### 为什么不推荐使用 `SingleThreadExecutor`？

    `SingleThreadExecutor` 使用无界队列 `LinkedBlockingQueue` 作为线程池的工作队列（队列的容量为 Intger.MAX_VALUE）。`SingleThreadExecutor` 使用无界队列作为线程池的工作队列会对线程池带来的影响与 `FixedThreadPool` 相同。说简单点就是可能会导致 OOM。

#### CachedThreadPool

`CachedThreadPool` 是一个会根据需要创建新线程的线程池。下面通过源码来看看 `CachedThreadPool` 的实现：

```java
    /**
     * 创建一个线程池，根据需要创建新线程，但会在先前构建的线程可用时重用它。
     */
    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
Copy to clipboardErrorCopied
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

    `CachedThreadPool` 的 `corePoolSize` 被设置为空（0），`maximumPoolSize`被设置为 Integer.MAX.VALUE，即它是无界的，这也就意味着如果主线程提交任务的速度高于 `maximumPool` 中线程处理任务的速度时，`CachedThreadPool` 会不断创建新的线程。极端情况下，这样会导致耗尽 cpu 和内存资源。

##### 执行任务过程介绍

![CachedThreadPool的execute()方法的执行示意图](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93/CachedThreadPool-execute.png)

**上图说明：**

1. 首先执行 `SynchronousQueue.offer(Runnable task)` 提交任务到任务队列。如果当前 `maximumPool` 中有闲线程正在执行 `SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)`，那么主线程执行 offer 操作与空闲线程执行的 `poll` 操作配对成功，主线程把任务交给空闲线程执行，`execute()`方法执行完成，否则执行下面的步骤 2；
2. 当初始 `maximumPool` 为空，或者 `maximumPool` 中没有空闲线程时，将没有线程执行 `SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)`。这种情况下，步骤 1 将失败，此时 `CachedThreadPool` 会创建新线程执行任务，execute 方法执行完成。

##### 为什么不推荐使用 `CachedThreadPool`？

`CachedThreadPool`允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程从而导致OOM。

#### ScheduledThreadPoolExecutor

**`ScheduledThreadPoolExecutor` 主要用来在给定的延迟后运行任务，或者定期执行任务。**

**`ScheduledThreadPoolExecutor` 使用的任务队列 `DelayQueue` 封装了一个 `PriorityQueue`，`PriorityQueue` 会对队列中的任务进行排序，执行所需时间短的放在前面先被执行。(`ScheduledFutureTask` 的 `time` 变量小的先执行)，如果执行所需时间相同则先提交的任务将被先执行(`ScheduledFutureTask` 的 `squenceNumber` 变量小的先执行)。**

**`ScheduledThreadPoolExecutor` 和 `Timer` 的比较：**

- `Timer` 对系统时钟的变化敏感，`ScheduledThreadPoolExecutor`不是；
- `Timer` 只有一个执行线程，因此长时间运行的任务可以延迟其他任务。 `ScheduledThreadPoolExecutor` 可以配置任意数量的线程。 此外，如果你想（通过提供 ThreadFactory），你可以完全控制创建的线程;
- 在 `TimerTask` 中抛出的运行时异常会杀死一个线程，从而导致 `Timer` 死机，即计划任务将不再运行。`ScheduledThreadExecutor` 不仅捕获运行时异常，还允许您在需要时处理它们（通过重写 `afterExecute` 方法 `ThreadPoolExecutor`）。抛出异常的任务将被取消，但其他任务将继续运行。

##### 运行机制

![ScheduledThreadPoolExecutor运行机制](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93/ScheduledThreadPoolExecutor%E6%9C%BA%E5%88%B6.png)

**`ScheduledThreadPoolExecutor` 的执行主要分为两大部分：**

1. 当调用 `ScheduledThreadPoolExecutor` 的 **`scheduleAtFixedRate()`** 方法或者 **`scheduleWithFixedDelay()`** 方法时，会向 `ScheduledThreadPoolExecutor` 的 **`DelayQueue`** 添加一个实现了 **`RunnableScheduledFuture`** 接口的 **`ScheduledFutureTask`** 。
2. 线程池中的线程从 `DelayQueue` 中获取 `ScheduledFutureTask`，然后执行任务。

**`ScheduledThreadPoolExecutor` 为了实现周期性的执行任务，对 `ThreadPoolExecutor`做了如下修改：**

- 使用 **`DelayQueue`** 作为任务队列；
- 获取任务的方不同
- 执行周期任务后，增加了额外的处理

##### ScheduledThreadPoolExecutor 执行周期任务的步骤

![ScheduledThreadPoolExecutor执行周期任务的步骤](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93/ScheduledThreadPoolExecutor%E6%89%A7%E8%A1%8C%E5%91%A8%E6%9C%9F%E4%BB%BB%E5%8A%A1%E6%AD%A5%E9%AA%A4.png)

1. 线程 1 从 `DelayQueue` 中获取已到期的 `ScheduledFutureTask（DelayQueue.take()）`。到期任务是指 `ScheduledFutureTask`的 time 大于等于当前系统的时间；
2. 线程 1 执行这个 `ScheduledFutureTask`；
3. 线程 1 修改 `ScheduledFutureTask` 的 time 变量为下次将要被执行的时间；
4. 线程 1 把这个修改之后的 `ScheduledFutureTask` 放回 `DelayQueue` 中（`DelayQueue.add()`)。

## Atomic 原子类

    `Atomic` 翻译成中文是原子的意思。在java中 Atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。所以原子类说简单点就是具有原子/原子操作特征的类。

    并发包`java.util.concurrent` 的原子类都存放在 `java.util.concurrent.atomic`下。

### JUC 包中的原子类是哪 4 类?

**基本类型**

使用原子的方式更新基本类型

- `AtomicInteger`：整形原子类
- `AtomicLong`：长整型原子类
- `AtomicBoolean`：布尔型原子类

**数组类型**

使用原子的方式更新数组里的某个元素

- `AtomicIntegerArray`：整形数组原子类
- `AtomicLongArray`：长整形数组原子类
- `AtomicReferenceArray`：引用类型数组原子类

**引用类型**

- `AtomicReference`：引用类型原子类
- `AtomicStampedReference`：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
- `AtomicMarkableReference` ：原子更新带有标记位的引用类型

**对象的属性修改类型**

- `AtomicIntegerFieldUpdater`：原子更新整形字段的更新器
- `AtomicLongFieldUpdater`：原子更新长整形字段的更新器
- `AtomicReferenceFieldUpdater`：原子更新引用类型字段的更新器

## AQS

    AQS 的全称为（`AbstractQueuedSynchronizer`），在 `java.util.concurrent.locks`包下面。

    AQS 是一个用来构建**锁和同步器**的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的 `ReentrantLock`，`Semaphore`，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`等等皆是基于 AQS 的。当然，我们自己也能利用 AQS 非常轻松容易地构造出符合我们自己需求的同步器。

### AQS 原理分析

#### AQS原理概览

    **AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

![AQS原理图](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/AQS%E5%8E%9F%E7%90%86%E5%9B%BE.png)

    AQS 使用一个 int 成员变量来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。

```
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

    状态信息通过 protected 类型的 getState，setState，compareAndSetState 进行操作。

#### AQS 对资源的共享方式

**AQS 定义两种资源共享方式**

- Exclusive（独占）：只有一个线程能执行，如 `ReentrantLock` 。又可分为公平锁和非公平锁：
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- **Share**（共享）：多个线程可同时执行，如 ` CountDownLatch`、`Semaphore`、 `CyclicBarrier`、`ReadWriteLock` 我们都会在后面讲到。

`ReentrantReadWriteLock` 可以看成是组合式，因为 `ReentrantReadWriteLock` 也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS 已经在顶层实现好了。

#### AQS 底层使用了模板方法模式

同步器的设计是基于模板模式的，如果需要自定义同步器一般是这种方式（模板模式很经典的一个应用）：

1. 使用者继承 `AbstractQueuedSynchronizer` 并重写指定的方法。（这些重写方法很简单，无非是对于共享资源 state 的获取和释放）
2. 将 AQS 组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。

**AQS 使用了模板方法模式，自定义同步器时需要重写下面几个 AQS 提供的模板方法：**

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

默认情况下，每个方法都抛出 `UnsupportedOperationException`。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS 类中的其他方法都是 final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

以 ReentrantLock 为例，state 初始化为 0，表示未锁定状态。A 线程 lock()时，会调用 tryAcquire()独占该锁并将 state+1。此后，其他线程再 tryAcquire()时就会失败，直到 A 线程 unlock()到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证 state 是能回到零态的。

再以 `CountDownLatch` 以例，任务分为 N 个子线程去执行，state 也初始化为 N（注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后 ` countDown()` 一次，state 会 CAS(Compare and Swap)减 1。等到所有子线程都执行完后(即 state=0)，会 unpark()主调用线程，然后主调用线程就会从 `await()` 函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现 `tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如 `ReentrantReadWriteLock`。

### AQS 组件总结

- **`Semaphore`(信号量)-允许多个线程同时访问：** `synchronized` 和 `ReentrantLock` 都是一次只允许一个线程访问某个资源，`Semaphore`(信号量)可以指定多个线程同时访问某个资源。
- **`CountDownLatch `（倒计时器）：** `CountDownLatch` 是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。
- **`CyclicBarrier`(循环栅栏)：** `CyclicBarrier` 和 `CountDownLatch` 非常类似，它也可以实现线程间的技术等待，但是它的功能比 `CountDownLatch` 更加复杂和强大。主要应用场景和 `CountDownLatch` 类似。`CyclicBarrier` 的字面意思是可循环使用（`Cyclic`）的屏障（`Barrier`）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。`CyclicBarrier` 默认的构造方法是 `CyclicBarrier(int parties)`，其参数表示屏障拦截的线程数量，每个线程调用 `await()` 方法告诉 `CyclicBarrier` 我已经到达了屏障，然后当前线程被阻塞。
