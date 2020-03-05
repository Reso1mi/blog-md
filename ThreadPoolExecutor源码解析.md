---
title:
  ThreadPoolExecutor源码解析
tags:
  [多线程,并发编程]
categories:
	[并发]
date: 2019/7/30
cover: http://static.imlgw.top/blog/20190806/wdUz2J3lFXcL.jpg?imageslim
top: True
---

## 深入ThreadPoolExecutor源码

### 类结构

![mark](http://static.imlgw.top/blog/20190802/wTkHh1NbyM8K.png?imageslim)

这里主要要说的是 `ThreadPoolExecutor`类

### 线程池状态

打开源码映入眼帘的就是这几个字段和方法，对应的就是线程池的一些运行状态和相关方法

```java
//控制线程池中数量和状态的字段,用AtomicInteger保存
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
//count bit顾名思义就是 workerCount的位数，这里是29
private static final int COUNT_BITS = Integer.SIZE - 3;
//1<<29 -1 == 1111....1111(29个1) 线程数(workerCount)上限 大约5亿
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
//高三位存放状态，相应的低29位就是workerCount
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
// 拆解ctl获取状态和数量
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
// 拼接状态和数量得到ctl
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

#### 状态转换过程

![mark](http://static.imlgw.top/blog/20190802/EWlphYbCMVjv.png?imageslim)

💡 **RUNNING** ：能接受新提交的任务，并且也能处理阻塞队列中的任务；

💡 **SHUTDOWN**：关闭状态，不再接受新提交的任务，但却可以继续处理阻塞队列中已保存的任务。在线程池处于 `RUNNING` 状态时，调用 `shutdown()`方法会使线程池进入到该状态。（`finalize()` 方法在执行过程中也会调用shutdown()方法进入该状态）

💡 **STOP**：不能接受新任务，也不处理队列中的任务，会中断正在处理任务的线程。在线程池处于 `RUNNING` 或 `SHUTDOWN` 状态时，调用 `shutdownNow()` 方法会使线程池进入到该状态

💡 **TIDYING**：如果所有的任务都已终止了，workerCount (有效线程数) 为0，线程池进入该状态后会调用 terminated() 方法进入TERMINATED 状态

💡 **TERMINATED**：在terminated() 方法执行完后进入该状态，默认terminated()方法中什么也没有做。

进入`TERMINATED`的条件如下：

- 线程池不是RUNNING状态；
- 线程池状态不是TIDYING状态或TERMINATED状态；
- 如果线程池状态是SHUTDOWN并且workerQueue为空；
- workerCount为0；
- 设置TIDYING状态成功。

### 成员变量

再往下，就会看见一些很重要的成员变量

```java
//任务缓存队列，存放待执行的任务
private final BlockingQueue<Runnable> workQueue; 
//可重入锁，线程池主要的锁
private final ReentrantLock mainLock = new ReentrantLock();
//线程集合(线程池的workers集合)
private final HashSet<Worker> workers = new HashSet<Worker>();
//对应的条件变量
private final Condition termination = mainLock.newCondition(); 
//用来记录线程池中曾经出现过的最大线程数，和线程池容量没有关系
private int largestPoolSize;
//用来记录已经执行完毕的任务个数
private long completedTaskCount; 
//工厂方法，用来创建线程
private volatile ThreadFactory threadFactory; 
//拒绝策略
private volatile RejectedExecutionHandler handler; 
//线程闲置时候的最大存活时间
private volatile long keepAliveTime; 
//是否允许核心线程闲置的时候超时
private volatile boolean allowCoreThreadTimeOut; 
//核心线程数
private volatile int corePoolSize; 
//最大线程数
private volatile int maximumPoolSize; 
//默认的拒绝策略：AbortPolicy直接拒绝并抛异常
private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();
```

### execute()

 **源码分析**

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();

    //获取线程池的状态和线程数量
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) { //workerCount小于核心线程数量
        //将任务添加到workers中，第二个参数代表是否根据corePoolSize来添加线程，false则根据maxPoolSize
        if (addWorker(command, true))
            return;
        //添加失败，重新获取ctl
        c = ctl.get();
    }
    
    //上面添加到workers中失败，有可能是核心线程不够用了或者线程池不是运行状态
    //如果线程池是Running状态 尝试添加任务到阻塞队列中
    if (isRunning(c) && workQueue.offer(command)) {
        //添加到等待队列成功，重新获取ctl
        int recheck = ctl.get();
        //如果线程池不是Running状态就从等待队列中remove这个任务
        if (! isRunning(recheck) && remove(command))
            reject(command); //采用拒绝策略拒绝该任务
        else if (workerCountOf(recheck) == 0)
            //线程池Running但是没有线程
            //创建一个线程但是不传入Runnable(已经在阻塞队列中了)
            addWorker(null, false);
    }
    //两种情况
    //1. 线程池不是Running状态并且command不为null，addWorker会直接false然后拒绝这个任务
    //2. 添加到workQueue(阻塞队列)失败，也就是 queue满了，可能是需要扩容了
    //   所以后面的参数是false，添加失败也会直接拒绝
    else if (!addWorker(command, false))
        reject(command);
}
```

这里要理解addWorker的第二个参数，true则代表当前线程池的**上界**仍然是corePoolSize，后面的addWorker会根据上界来判断是否增加线程，false则代表**上界**是maximumPoolSize，这一点在后面的分析中会看到。

**Executor大致执行流程**

![mark](http://static.imlgw.top/blog/20190803/j3xQrmOkc3Kl.png?imageslim)

### addWorker()

`addWorker()` 的作用就是创建线程(Worker)并且添加到Workers集合中，然后启动线程

**源码分析**

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // rs>=SHUTDOWN说明不是RUNNING状态
        if (rs >= SHUTDOWN && !(rs == SHUTDOWN && firstTask == null &&!workQueue.isEmpty()))
            return false;
        
        for (;;) {
            int wc = workerCountOf(c);
            //如果大于可允许的最大线程数，或者大于当前的线程池的上界，直接false
            //上面传入的第二个参数的作用体现出来了，为true上界则是corePoolSize
            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            //利用CAS自增 增大workCount线程数，成功后就跳出循环
            if (compareAndIncrementWorkerCount(c))
                break retry;
            //CAS失败，重新获取ctl
            c = ctl.get();  // Re-read ctl
            //判断还是不是Running状态(能到这里说明rs==Running状态)，不是的话就跳出去重新来过
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
            // CAS失败并且还是Running状态，继续自旋尝试自增
        }
    }
    //线程是否启动，以及线程是否添加
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        //创建一个Worker(对线程的封装)
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());
				//是RUNNING状态 或者 是SHUTDOWN状态且没有提交任务(SHUTDOWN状态还可以执行阻塞队列的任务)
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    //添加到工作集中
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s; //记录最大值
                    workerAdded = true; //添加成功
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                //启动线程，执行Worker的run方法
                t.start(); 
                workerStarted = true; //启动成功
            }
        }
    } finally {
        if (! workerStarted)
            //启动失败，回滚workers并尝试关闭线程池
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

🔸 首先判断当前线程池的状态是否适合继续addWorker，分析这里的if条件，RUNNING不会false ，STOP，TIDYING，TERMINATED直接false，SHUTDOWN状态如果firstTask为空 阻塞队列中还有任务的时候不会false，其他情况都false。

🔸 获取当前的workerCount判断是否超过了当前的上界，这里就用到了第二个参数

🔸 然后利用CAS自旋增加workerCount

🔸 创建Worker对象，获取mainLock并加锁，因为workers是HashSet并不是线程安全的

🔸 再次获取线程池转台并判断是否合法，合法就添加到workers中，然后在finally块中释放锁

🔸 根据前面的workerAdded 判断是否启动线程

🔸 在最终的finally块中根据是否启动成功来决定是否回滚

### addWorkerFailed()

启动失败，回滚之前的添加操作

```java
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    //获取锁
    mainLock.lock();
    try {
        if (w != null)
            //从workers中移除
            workers.remove(w);
        //减少workerCount
        decrementWorkerCount();
        //尝试关闭线程池
        tryTerminate();
    } finally {
        mainLock.unlock();
    }
}
```

### Worker类

封装了线程对象，线程池维护的就是这些worker

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable
{
    /**
     * This class will never be serialized, but we provide a
     * serialVersionUID to suppress a javac warning.
     */
    private static final long serialVersionUID = 6138294804551838833L;

    /** Thread this worker is running in.  Null if factory fails. */
    final Thread thread;
    /** Initial task to run.  Possibly null. */
    Runnable firstTask; //传入的任务
    /** Per-thread task counter */
    volatile long completedTasks;

    /**
     * Creates with given first task and thread from ThreadFactory.
     * @param firstTask the first task (null if none)
     */
    Worker(Runnable firstTask) {
        setState(-1); // 设置状态为 -1
        this.firstTask = firstTask;
        //根据工厂方法创建线程
        //将Worker传递进去，作为Thread的参数
        //new Thread(Worker worker);
        this.thread = getThreadFactory().newThread(this);
    }

    /** Delegates main run loop to outer runWorker  */
    public void run() {
        runWorker(this);
    }

    // Lock methods
    //
    // The value 0 represents the unlocked state.
    // The value 1 represents the locked state.
    // 是否获取到了锁
    protected boolean isHeldExclusively() {
        return getState() != 0; 
    }

    protected boolean tryAcquire(int unused) {
        //利用CAS设置state
        //很明显这里是个不可重入的独占锁，具体可以对比ReentrantLock的实现方法
        if (compareAndSetState(0, 1)) {
            //继承自AbstractOwnableSynchronizer
            //保存当前的持有锁的线程
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }
    
    //释放锁设置state=0
    protected boolean tryRelease(int unused) {
        //设置独占锁线程为空
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }

    public void lock()        { acquire(1); } //最终会调用tryAcquire(1);
    public boolean tryLock()  { return tryAcquire(1); }
    public void unlock()      { release(1); } //最终会调用tryRelease(1);
    public boolean isLocked() { return isHeldExclusively(); }

    //打断已经启动的线程
    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

这里可以看到Worker继承了AQS并且实现了Runnable接口，然后借助AQS实现了一个`独占的不可重入的锁`，其实这也是很巧妙的一点（这里我和博客上的理解有点出入）。

到这里大家肯定会有疑问，为什么这里要实现AQS然后实现一个锁？既然要又为什么要实现一个不可重入的，而不直接使用`ReentrantLock` 那不是更加方便么？？除此之外还有一个小细节就是构造器里面为什么`setState(-1)`  这样不就获取不到锁了么？？

其实这是为了后面`shutdown`的时候`interruptIdleWorkers`能判断出线程是否在工作，从而打断那些空闲的线程。如果使用可重入锁的话就无法通过`tryLock()` 来判断线程是否在工作。而`setState(-1)` 则是为了防止在任务没有开始前被打断

### runWorker()

在上面`AddWorker()`最后添加成功后会启动Worker线程，而在worker线程中run方法又会调用一个`runWorker()`方法，这里就是具体执行任务的地方

```java
final void runWorker(Worker w) {
    //当前执行线程
    Thread wt = Thread.currentThread();
    //拿到任务
    Runnable task = w.firstTask; 
    w.firstTask = null;
    //前面构造worker的时候设置了state=-1
    //设置state为0，tryRelease(1)
    //其实这样是为了在这之前不会被打断（还没有开始执行任务）
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true; //是否因为异常而退出？
    try {
        //task为空就从阻塞队列中拿任务
        while (task != null || (task = getTask()) != null) {
            //加锁，但是这个锁不会和其他的Worker互斥，这个锁只是用来判断worker是否在工作
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                //打断当前执行线程wt
                wt.interrupt();
            try {
                //空方法交给子类去实现
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run(); //执行任务
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    //空方法交给子类去实现
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++; //完成任务数++
                w.unlock();
            }
        }
        completedAbruptly = false; //下面的收尾工作会根据这个判断是否调整线程数
    } finally {
        processWorkerExit(w, completedAbruptly); //收尾工作
    }
}
```

🔸 首先获取了参数传递进来的worker携带的任务

🔸 然后执行了w.unlock()，实际上这里就对应了上面Worker类构造器中的setState(-1)，正因为前面设置了state为 -1 所以在unlock()之前，获得锁的CAS操作肯定都会失败，当然这也是为了在任务启动前不会被打断，所以在这里unlock()就又将state设置为了0 也就表示可以通过CAS获得锁了，也可以被shutdown打断了。

🔸 然后这个线程就会执行worker中的任务，如果worker中任务为空就会从阻塞队列中获取任务

🔸 获取到任务后进入循环先进行 lock() 操作，这就代表已经开始执行任务了，这个时候shutdown就无法发送中断信号中断这个线程执行 (注意这个lock并不会和其他的worker互斥，因为每个Worker都是新new出来的，完全不相关的，他们的state状态都是独立的)

🔸 `runStateAtLeast(ctl.get(), STOP)` 返回`ctl.get() >= STOP`  ，判断线程池是否正在关闭如果是就打断该线程，如果不是需要确保线程不是Interrupt状态

`If pool is stopping, ensure thread is interrupted;  if not, ensure thread is not interrupted.`

🔸`beforeExecute()` 和`afterExecute()` 都是空方法交给子类去实现的。

🔸 到finally块里面就代表这个工作线程已经快要结束了，`processWorkerExit()`  就是处理一些"后事"的

### getTask()

这个方法就是用来从阻塞队列中获取任务的，如果 `getTask()` 返回null，在`runWorker()` 里面接收到`null` 后就会**跳出循环**进而执行`finally` 块里面的`processWorkerExit()` 。而这也就意味着这个线程`执行结束`了，不会在执行任务了，剩下的就是等待JVM回收这个线程。

```java
private Runnable getTask() {
    //是否超时？和keepAliveTime相关
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        // 如果线程池状态为SHUTDOWN并且任务队列没任务 或者 线程状态>=STOP
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        // 是否允许线程超时 允许核心线程超时(主动超时)？ 或者wc大于了核心线程数(被动超时)
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
		//此时需要回收多余的线程
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c)) //cas减少线程数
                return null; //返回null
            continue; //cas失败 继续循环
        }

        try {
            //根据timed判断是限时获取还是直接获取
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take(); //阻塞的获取
            if (r != null)
                return r;
            //没获取到，超时了
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

🔸 获取线程池状态，如果线程池状态为SHUTDOWN并且任务队列没任务 或者 线程状态>=STOP 就通过CAS自旋减少线程数，然后返回null

🔸 判断是否允许当前线程获取任务超时，如果允许核心线程超时就代表所有线程都会超时限制，又或者是当前线程数超过了核心线程数，也就是经过了扩容，所以核心线程之外的线程都是有超时限制的。

🔸 如果 ① wc超过最大线程数  ②没超过最大线程数，但是超时了并且此时wc>1(留一个处理任务)③没超过最大线程数，但是超时了并且阻塞队列为空，此时需要回收多余的线程

🔸 根据timed选取从阻塞队列中获取任务的方式，要么是限时获取的poll或者一直阻塞的take，获取到了之后返回

### processWorkerExit()

看名字就知道是一个收尾的方法，执行线程结束后的一些必要收尾工作。

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        //如果是因为异常而退出则 getTask() 没有机会去调整线程数所以需要在这里调整
        decrementWorkerCount();  // wc--

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        //完成任务总数+=该worker完成的任务数
        completedTaskCount += w.completedTasks;
        //移除出线程集workers
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }
    
    //尝试结束线程池
    tryTerminate();
    
    //获取ctl
    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {
        //非异常结束
        if (!completedAbruptly) {
            //最小值根据是否允许核心线程超时来判断
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && ! workQueue.isEmpty())
                min = 1; //如果最小为0并且任务队列不为空则保证线程池中至少有一个线程执行这些任务
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        //1.异常结束，新增加一个线程到线程池,相当于替换了异常的线程
        //2.非异常结束，工作线程小于核心线程，增加线程，确保在不允许核心线程超时的情况下线程数不小于corePoolSize
        addWorker(null, false);
    }
}
```

`runStateLessThan`  c < STOP 返回true就说明是`SHUTDOWN` 或者`RUNNING` ，到这里该工作线程（Worker）的生命周期就结束了。

### 工作线程的生命周期

![mark](http://static.imlgw.top/blog/20190803/NbjiMxDqJlu4.png?imageslim)

### tryTerminate()

在上面的处理收尾工作的`processWorkerExit()` 的时候中间调用了一个 `tryTerminate()` 方法

```java
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        //如果是在RUNNING状态 直接return
        //如果已经是TIDYING或者TERMINATED状态 直接return
        //如果是SHUTDOWN状态并且阻塞队列中还有任务 直接return
        //return就说明线程池还不到关闭的时候
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
        //到这里就说明要么是STOP状态，要么是SHUTDOWN状态阻塞队列也没任务了
        if (workerCountOf(c) != 0) { // Eligible to terminate
            //中断一个空闲的worker线程
            interruptIdleWorkers(ONLY_ONE);
            return;
        }
		
        //wc==0 没有线程存活了
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //CAS自旋 修改线程池状态为TIDYING
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated(); //空方法,交给子类去实现的
                } finally {
                    //设置状态为 TERMINATED
                    ctl.set(ctlOf(TERMINATED, 0));
                    //唤醒termination上awiat的线程
                    termination.signalAll();
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}
```

这里最后的`termination.signalAll()` 实际上是唤醒的`awaitTermination()` 方法阻塞的线程

```java
public boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (;;) {
            if (runStateAtLeast(ctl.get(), TERMINATED))
                return true;
            if (nanos <= 0)
                return false;
            //限时等待
            nanos = termination.awaitNanos(nanos);
        }
    } finally {
        mainLock.unlock();
    }
}
```

### shutdown()

```java
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        //CAS自旋设置线程池状态为 SHUTDOWN
        advanceRunState(SHUTDOWN);
        //打断idle(空闲)的worker
        interruptIdleWorkers(); 
        onShutdown(); // hook for ScheduledThreadPoolExecutor
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
}
```

这里关键的就是这个打断空闲线程的操作。

### interruptIdleWorkers()

打断空闲的worker，这里的空闲线程其实指的就是在阻塞队列上获取不到任务而阻塞的线程

经过上面的分析我们知道Worker会不断的从阻塞队列中去拿任务也就是 `getTask()` 方法，如果阻塞队列为空就会阻塞住 直到有任务提交到阻塞队列中，或者执行线程被中断。

这里我们是要SHUTDOWN那阻塞队列中肯定是不会再有任务提交，所以`take()` 会阻塞住，所以我们就只能通过打断执行线程的方式来打断`take()` 操作，否则会一直阻塞，线程池无法关闭。

```java
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock(); //加锁，因为Wokers是HashSet是线程不安全的
    try {
        for (Worker w : workers) {
            Thread t = w.thread;
            //没有被打断并且没有在工作(空闲)
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    //打断Worker里面的线程
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            //只打断一个就直接break
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
```

🔸首先获取了mainLock保证了workers的线程安全避免产生并发修改异常。

🔸然后遍历workers，打断那些没有被打断并且没有工作的线程，那这里怎么知道线程是不是在工作呢？

别忘了前面提到的Worker类借助AQS实现了一个`不可重入`的`lock` 方法，而在worker执行任务的时候会执行 `lock` 加锁，所以在这里`tryLock()` 如果返回`true`则说明 并没有在工作可以打断，反之如果正在工作`tryLock()` 不可重入，无法获取到自己持有的锁返回false，所以线程肯定是在工作状态所以不应该打断，这些线程会在执行完任务后自行了断，因为线程池状态已经设置为`SHUTDOWN` 当然前提是这些任务是**可终止的**

🔸打断后释放`tryLock()`获取到的锁

🔸如果只打断一个就直接break，否则就继续下一轮循环

🔸释放mianLock

> 瞎猜: 这里可以通过getState判断线程状态但是有可能在执行任务的过中阻塞

### shutdownNow()

和上面的shutdown()一样都是用来关闭线程池的但是看方法名就知道这个比较粗暴，shutdownNow立刻马上关闭，一点情面都不给😂，虽然说是打断所有的线程但是毕竟使用的是`Interrupt` ，也许别人正在执行的线程根本就不会理你😂，所以在提交任务的时候要对任务进行正确的interrupt响应，或者确保线程不会一直阻塞否则线程池就无法正常关闭

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        //CAS自旋设置状态为STOP
        advanceRunState(STOP);
        //打断所有的线程包括正在工作的线程
        interruptWorkers();
        //排干阻塞队列，因为已经STOP了不会再执行队列里面的任务了
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    //尝试关闭线程池
    tryTerminate();
    return tasks;
}
```

### interruptWorkers()

```java
private void interruptWorkers() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers)
            //打断所有启动的线程
            w.interruptIfStarted();
    } finally {
        mainLock.unlock();
    }
}
```

打断了所有的启动的线程，即使他们可能不会响应这个Interrupted信号，但是由于线程池状态已经变为了STOP，所以他们也活不长了(当然前提是执行的任务是可以结束的)，在下一次获取任务的时候就会直接return 。

## 细节是魔鬼

在群里面看见的问题，为什么线程池要用 线程不安全的 `HashSet` 然后设置了一个 `mainLock` 控制并发，而不是直接使用线程安全的并发集合？

这一点其实在源码的注释中已经说的很清楚了，之前一直没有注意到

```java
    /**
     * Lock held on access to workers set and related bookkeeping.
     * While we could use a concurrent set of some sort, it turns out
     * to be generally preferable to use a lock. Among the reasons is
     * that this serializes interruptIdleWorkers, which avoids
     * unnecessary interrupt storms, especially during shutdown.
     * Otherwise exiting threads would concurrently interrupt those
     * that have not yet interrupted. It also simplifies some of the
     * associated statistics bookkeeping of largestPoolSize etc. We
     * also hold mainLock on shutdown and shutdownNow, for the sake of
     * ensuring workers set is stable while separately checking
     * permission to interrupt and actually interrupting.
     */
    private final ReentrantLock mainLock = new ReentrantLock();

```

一开始还挺抗拒的，感觉有的地方有的看不懂（英语渣渣留下眼泪）然后去stackoverflower上看了一下找到了一个答案  [Using ReentrantLock in ThreadPoolExecutor to ensure thread-safe workers](https://stackoverflow.com/questions/31942117/using-reentrantlock-in-threadpoolexecutor-to-ensure-thread-safe-workers) 

大概总结一下就是两点

1. 避免 “中断风暴” ，如果是用的显式锁那么如果有10个线程同时去执行shutdown方法，那么10个线程会排队去执行`interruptIdleWorkers` ，如果使用并发安全的队列的话，在10个线程同时去执行shutdown方法，那么肯定会同时去 `interruptIdleWorkers` 也就是所谓的中断风暴

   > [中断风暴(维基百科)](https://en.wikipedia.org/wiki/Interrupt_storm) 中文几乎搜不到相关资料，其实就是字面意思，而太多的中断请求会的影响系统的整体性能，不得不说大师就是大师，细节之处则更能体现水平，我等也只能膜拜了

2. 另一点就是为了方便统计线程池的一些信息比如 `largestPoolSize` 等，这点也好理解，使用并发的Set只能保这个set的并发安全，而对于其他的一些线程池的相关的信息统计起来就比较麻烦，可能又需要另外的加锁，所以索性就直接搞一个全局的锁，一举两得

## ThreadPoolExecutor使用

### 构造方法

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

其实从中也可以看出对各个参数的一些限制。

**corePoolSize：**核心池的大小，这个参数与后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了`prestartAllCoreThreads()`或者`prestartCoreThread()`方法，从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。`默认情况下`，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中

**maximumPoolSize：**线程池最大线程数，它表示在线程池中最多允许创建多少个线程

**keepAliveTime**：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当`线程池中的线程数大于corePoolSize时`，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize：即当线程池中的线程数大于`corePoolSize`时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize，但是如果调用了`allowCoreThreadTimeOut(boolean)`方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0，这些内容其实在上面的深入源码中都有过分析。

**unit：**参数keepAliveTime的时间单位

**workQueue**：一个阻塞队列，用来存储等待执行的任务，这个参数的选择会对线程池的运行过程产生重大影响

**threadFactory**：线程工厂，主要用来创建线程(根据传进来的Runnable/Callable)

**handler**：表示当拒绝处理任务时的策略，有以下四种取值

- **ThreadPoolExecutor.AbortPolicy** 丢弃任务并抛出RejectedExecutionException异常。 
- **ThreadPoolExecutor.DiscardPolicy** 也是丢弃任务，但是不抛出异常。 
- **ThreadPoolExecutor.DiscardOldestPolicy** 丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
- **ThreadPoolExecutor.CallerRunsPolicy**  由调用线程处理该任务 

### 线程池的关闭

`shutdown()`  `shutdownNow()` ，上面已经分析过了，就不再过多介绍了。

### 工厂方法

上面的构造器中一共有7个参数，可见要构造一个线程池并非那么容易，所以jdk 在`Executors` 类中为我们提供了一些工厂方法，可以直接构造一些特定的线程池。

**newCachedThreadPool()**

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

可以看到`core`为0，最大值为`Integer.MAX_VALUE`，任务队列使用的`SynchronousQueue` ，这个队列是一个很奇葩的阻塞队列，实际上它不是一个真正的队列，每个删除操作都要等待插入操作，反之每个插入操作也都要等待删除动作，只有两个都准备好的时候才不会阻塞，所以它内部不会为队列元素维护空间，也就是说并不会缓存任务，一旦提交了(put)任务，要么就由空闲线程去执行(take)，要么创建一条新线程去执行(take)。

**newFixedThreadPool()**

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

可以看到，最大值和core都是 `nThread` ，也就是最多`nThread`个线程，阻塞队列采用 `LinkedBlockQueue` 

**newSingleThreadExecutor()**

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

等价于`newFixedThreadPool(1)` ，但是这里的返回值经过了一层包装 返回的不再是`ThreadPoolExecutor` 也就是不会再有那些扩展的monitor方法

> A wrapper class that exposes only the ExecutorService methods  of an ExecutorService implementation.
>

类似的方法其实还有一些，像`newWorkStealingPool` 等，感兴趣可以自己去查一查。

其实这里阿里巴巴Java开发规范并不建议使用工厂方法创建线程

![mark](http://static.imlgw.top/blog/20190806/lMVPdlPqEPNO.png?imageslim)

所以建议还是通过构造器的方式去创建线程，这样也更加灵活更加可控。