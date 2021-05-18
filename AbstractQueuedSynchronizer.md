---
title:
  AQS源码解析（上）
tags:
  [多线程,并发编程]
categories:
  [并发]
cover: http://static.imlgw.top/blog/20190924/fjWv53purgaU.jpg?imageslim
date: 2019/9/24
---

## AbstractQueuedSynchronized

`AbstractQueuedSynchronized` 简称AQS，这个类是整个并发包的基础工具类， ReentrantLock、CountDownLatch、Semaphore、FutureTask 等并发工具类底层都是通过它来实现的

AQS定义了两种资源共享的方式：

- Exclusive：独占式，只有一个线程能获取资源并执行，比如ReentrantLock。
- Share：共享式，多个线程获取资源，多个线程可以同时执行，比如CountDownLatch，ReentrantReadWriteLock的ReadLock等

### AQS结构

#### 属性

主要的就是这三个volatile修饰的Node对象，还有一些对应的偏移量(用于CAS的)

```java
/**
 * Head of the wait queue, lazily initialized.  Except for
 * initialization, it is modified only via method setHead.  Note:
 * If head exists, its waitStatus is guaranteed not to be
 * CANCELLED.
 * 头节点，可以理解为当前持有锁的节点
 * 在分析的过程中不要将它算作队列的一部分！它只是一个空节点
 */
private transient volatile Node head;

/**
 * Tail of the wait queue, lazily initialized.  Modified only via
 * method enq to add new wait node.
 * 尾节点
 */
private transient volatile Node tail;

/**
 * The synchronization state.
 * 同步状态，0代表没有被占用，1代表被一个线程占用，>1 代表被同一个线程多次占用（可重入）
 */
private volatile int state;
```

#### Node节点

```java
static final class Node {
    /** Marker to indicate a node is waiting in shared mode */
    //共享模式
    static final Node SHARED = new Node();
    /** Marker to indicate a node is waiting in exclusive mode */
    //独占模式
    static final Node EXCLUSIVE = null;

    /** waitStatus value to indicate thread has cancelled */
    //取消抢锁
    static final int CANCELLED =  1;
    
    /** waitStatus value to indicate successor's thread needs unparking */
    //代表当前节点的后续节点需要被unparking，也就是说后继节点状态是parking
    static final int SIGNAL    = -1;
    
    /** waitStatus value to indicate thread is waiting on condition */
    //在condition上等待
    static final int CONDITION = -2;
    
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    static final int PROPAGATE = -3;

	//上面的那些状态
    volatile int waitStatus;

    //前驱节点    
    volatile Node prev;
    //后继节点
    volatile Node next;

    /**
     * The thread that enqueued this node.  Initialized on
     * construction and nulled out after use.
     */
    volatile Thread thread;

    /**
     * Link to next node waiting on condition, or the special
     * value SHARED.  Because condition queues are accessed only
     * when holding in exclusive mode, we just need a simple
     * linked queue to hold nodes while they are waiting on
     * conditions. They are then transferred to the queue to
     * re-acquire. And because conditions can only be exclusive,
     * we save a field by using special value to indicate shared
     * mode.
     */
    Node nextWaiter;

    /**
     * Returns true if node is waiting in shared mode.
     */
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    //返回前驱节点
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

## ReentrantLock分析

我们知道ReentrantLock内部有两个锁，一个是公平锁(FairSync)🔒，一个是非公平锁(NonFairSync)🔒，这两个锁都是独占锁，两者实现的差异其实并不大，我们先从`公平锁`开始说起。

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }
    
    /**
     * Fair version of tryAcquire.  Don't grant access unless
     * recursive call or no waiters or is first.
     */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

### Lock()

🔔 **这里我们为了模拟真实的情况，我们假设有两个线程`Thread0` 和`Thread1` 过来执行了`Lock()` 方法，且`Thread0` 比`Thread1` 要先执行。**

`Lock()`方法中调用了父类的`acquire(1)`

#### acquire()

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

首先尝试tryAcquire(1)，这个tryLock()在AQS中没有具体实现是交给子类去实现的，所以这里就会调用FairSync的，tryAquire(1)

#### tryAquire()

```java
protected final boolean tryAcquire(int acquires) {
    //获取当前线程
    final Thread current = Thread.currentThread();
    //获取同步状态
    int c = getState();
    if (c == 0) { //0代表还没有线程占用
        //判断有没有前驱节点（除head节点外），没有则进行CAS设置同步状态获取锁
        if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) { 
            //设置当前线程为独占线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //到这里就说明已经有线程占用了，所以下面是为了重入
    else if (current == getExclusiveOwnerThread()) {
        //这里不用担心并发的问题，因为是独占锁，同时只有一个线程会到达这里
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

#### hasQueuedPredecessors()

公平锁和非公平锁的tryAcquire方法区别就在这里，这个方法就是判断有没有前驱节点(不包含头节点head，也就是)存在，有的话为了保证公平性就是需要等待，返回true。

```java
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    // 头不等于尾，并且队列的第一个节点所持有线程非当前线程返回true
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

💡 到这里我们分析下`Thread0` 和`Thread1` 的执行情况

🔸 首先`Thread0`先执行了`tryAcquire(1)`  没有任何阻碍，执行成功直接retrurn

🔸 `Thread1` 此时有多种情况：

- 还没有获取state ，`Thread0`执行完后获取State==1 ，由于是独占锁直接return false ，获取锁失败。

-  已经`getState()==0`了，执行`hasQueuedPredecessors` 方法，注意，此时head和tail都还没有初始化，都还是null（官方的注释中也提到head和tail是 lazily initialized ）所以这里会直接 return false，然后继续执行CAS，由于前面`Thread0` 已经将state设置为了 1 ，所以这里CAS肯定失败了，最终`Thread1`的tryAcquire失败返回false。

🔸 既然`tryAcquire()` 失败了，那`Thread1` 就会转头继续执行后面的方法

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

首先执行的就是`AddWaiter()` 

#### addWaiter()

这个方法的作用就是将Thread和mode包装成Node然后添加到链表尾部然后返回这个Node

```java
private Node addWaiter(Node mode) {
    //将当前线程和模式封装进Node
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    //如果尾节点不为空
    if (pred != null) {
        //将node连接在当前tail后面
        node.prev = pred;
        //cas设置当前tail为node
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 其实前面看似会有并发的问题其实并没有
    // 上面抢锁失败的线程会直接进入enq方法自旋重新设置，直到成功
    enq(node);
    return node;
}
```

💡 根据前面分析head和tail都还没有初始化都还是null，所以这里会直接进入 `enq()`方法

####  enq()

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

🔸 可以看到这里是一个循环，因为head和tail都还是空的所以这里进入第一个循环，CAS设置一个空的Node()为头节点head，然后将tail也指向这个head，到这里head和tail才算是初始化完成了(lazily initialized )。

🔸 循环，进入else，这里就将当前node连接到tail后面并且利用CAS自旋设置tail为当前node也就是包含`Thread1` 的node，然后return 当前节点的前驱节点(这里返回值并没有用到)。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

下一步就是执行`acquireQueued()`

❓ **为什么不直接在构造器里面就初始化头节点head？而要采用懒加载的方式？**

> CLH queues need a dummy header node to get started. Butwe don't create them on **construction**, because it would be wasted  effort if there is **never contention**. Instead, the nodeis constructed and head and tail pointers are set up **on first contention**.

以上摘自**Doug Lea** 大师的注释解释，根据我们的上面的分析，其实我们也看到了，第一个线程`Thread0` 过来的时候并没有去初始化head，后面的线程`Thread1`过来的时候 有了竞争才初始化了这个头节点head，如果直接初始化这个head，然后又没有锁竞争，这个head节点就被浪费了。 大师就是大师，太强了 😮

#### acquireQueued()

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            //获取当前节点的前置节点
            final Node p = node.predecessor();
            //如果前置节点是head就尝试去获取锁
            if (p == head && tryAcquire(arg)) {
                //设置头节点为当前节点
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        //发生异常取消抢锁
        if (failed)
            cancelAcquire(node);
    }
}
```

💡 因为前置节点是head，所以这里作为队列第一个可以去尝试获取锁，可以看到这里是个死循环，会一直尝试获取锁，其实类似与CAS的自旋，但是相比CAS自旋又有很大不同，它并不会一直自旋，详细可以继续往下看。

🔸 前面传递过来的node前继节点正好就是head，所以执行tryAcquire() 但是由于`Thread0` 还没有释放锁所以这里仍然失败。

🔸 进入第二个if 执行 `shouldParkAfterFailedAcquire(p,node)`

#### shouldParkAfterFailedAcquire()

看名字就知道是干啥的了，获取失败是否Park？

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //前置节点的状态
    int ws = pred.waitStatus;
    
    if (ws == Node.SIGNAL)
        //前置节点状态为-1，代表当前节点的后续节点需要被挂起
        /*
         * This node has already set status asking a release
         * to signal it, so it can safely park.
         */
        return true;
    if (ws > 0) {
        // 前驱节点 waitStatus大于0，说明前驱节点取消了排队。
        // 这里需要知道这点：进入阻塞队列排队的线程会被挂起，而唤醒的操作是由前驱节点完成的。
        // 所以下面这块代码说的是将当前节点的prev指向waitStatus<=0的节点，
        // 简单说，就是为了找个正常的前驱节点，因为你还得依赖它来唤醒呢，如果前驱节点取消了排队，就无法唤醒你了
        // 同时这个操作也会将那些 ws>0 的节点移除掉
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        /*
         * waitStatus must be 0 or PROPAGATE.  Indicate that we
         * need a signal, but don't park yet.  Caller will need to
         * retry to make sure it cannot acquire before parking.
         */
        //设置前驱节点状态为 -1
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

注意这个里面回剔除不正常的节点，为下面的唤醒操作考虑

💡 分析`Thread1`，我们先看看当前队列的状态

![mark](http://static.imlgw.top/blog/20190809/jOs2WA3R7lGR.png?imageslim)

🔸 因为前面的操作并没有对state进行操作，所以这里会直接进入最后的else，设置前驱节点的状态为 `SIGNAL`

然后renturn  false回到acquireQueued的内循环

🔸 再次尝试获取锁，`Thread0` 仍然没有释放锁，失败，再次进入shouldParkAfterFailedAcquire，这一次由于已经将前继节点的状态设置为`SIGNAL` 所以直接return true，进入后面的 `parkAndCheckInterrupt()` 方法

**此时状态变为**

![mark](http://static.imlgw.top/blog/20190809/pQ5NOkN5mQTY.png?imageslim)

#### parkAndCheckInterrupt()

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

到这里我们的线程`Thread1`就会被阻塞住，也不会继续自旋获取锁了。

> LockSupport.park()实际上调用的是Unsafe提供的指令属于`线程阻塞原语`，可以理解为二元信号量（只有一个permit），这个方法也会响应Interrupt  [参考](https://segmentfault.com/a/1190000008420938)

🔔 **假设此时又有一个线程`Thread2`过来了，并且`Thread0` 依然没有释放锁**

🔸 tryAcquire失败

🔸addWaiter()， 将`Thread2`和模式包装成Node添加到队尾（这个时候就不会进入enq了，因为tail此时为`Thread1`已经不为空了）然后返回包含`Thread2`的节点，队列状态变为：

![mark](http://static.imlgw.top/blog/20190809/yniIvx1sgpXa.png?imageslim)

🔸acquireQueued()，根据上面的状态图，这里前置节点并不是head，直接进入shouldParkAfterFailedAcquire()

🔸shouldParkAfterFailedAcquire()，明显前驱节点状态并不是`SIGNAL` 而是0，所以直接利用CAS设置前驱节点为`SIGNAL`  状态变为：

![mark](http://static.imlgw.top/blog/20190809/R7GmOQ4V8ESm.png?imageslim)

回到 `acquireQueued()` 继续自旋

🔸 前置节点不是head，调用shouldParkAfterFailedAcquire(NodeT1，mode)，成功，调用parkAndCheckInterrupt阻塞，至此，`Thread1`，`Thread2` 都在这里park住了。

### unLock()

🔔 继续上面的模拟，此时`Thread0`释放锁 ，调用`unlock()` 方法，unlock()会调用AQS中的`release(1)`方法

#### release()

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        //获取头节点
        Node h = head;
        //头节点不为空，并且头节点的waitStatus不是0
        if (h != null && h.waitStatus != 0)
            //unpark后继节点
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

首先执行`tryRelease(1)` ，和`tryAcquire()` 一样，这个方法最后是交给子类去实现的

#### tryRelease()

```java
protected final boolean tryRelease(int releases) {
    //同步状态减1
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        //解锁线程不是当前线程，解铃还须系铃人
        throw new IllegalMonitorStateException();
    //和上面一样这里不用担心并发的问题，因为是独占锁，同时只有一个线程会到达这里
    boolean free = false;
    //不为0则代表被重入了，需要多次release直到0才会释放锁
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

tryRelease()成功继续执行后面的语句，看一下当前AQS队列的状态

![mark](http://static.imlgw.top/blog/20190809/R7GmOQ4V8ESm.png?imageslim)

🔸 头节点head ! =null 并且head.waitState不是0，执行unparkSuccessor().

#### unparkSuccessor()

```java
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        //如果当前节点的ws小于0就设置为0，允许失败
        //其实是独占锁的话这里肯定不会失败，因为只有一个线程
        //release执行完之后，这个节点就会被移除掉。然后被GC
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    //获取后继节点
    Node s = node.next;
    //后继节点为空，或者已经撤销了，取消抢锁（1>0）
    if (s == null || s.waitStatus > 0) {
        s = null;
        //从后往前遍历找到正数第一个waitStatus<0的
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        //然后释放它
        LockSupport.unpark(s.thread);
}
```

🔸 `Thread1` 被`unpark()` 继续执行。

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

返回`Thread1`的 中断标志位，并复原为false，结束`parkAndCheckInterrupt()`方法，再次回到`acquireQueued()` 的循环中执行第一个if

```java
for (;;) {
    final Node p = node.predecessor();
    if (p == head && tryAcquire(arg)) {
        setHead(node);
        p.next = null; // help GC
        failed = false;
        return interrupted; //返回中断状态
    }
    if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
        interrupted = true;
}
```

🔸 由于`Thread0` 已经释放锁，同步状态已经变为0，`Thread1` 可以直接`tryAcquire`获取到锁，然后设置头节点为当前节点，将之前的head节点**移除**，返回中断状态，由于之前park期间没有被中断直接`return false`，acquire成功！！！ `Thread1` 获得锁！！！

❓ **为什么要从后往前遍历？**

> 这里看了一些博客介绍，大概有两个说法，一个是在`enq()` 方法里面，先设置的`node.prev = pred;`再执行的CAS最后执行的`t.next = node;` CAS成功后next也许还没有设置成功，从前往后遍历有可能找不到这个刚加入的节点；其次，在`cancelAcquire(node);` 的最后一步有一个`node.next=node`的操作，如果这个时候从前往后遍历会导致死循环。

❓ **从后往前遍历找到最前面第一个waitStatus<0的节点，这个操作如果返回的是个中间节点怎么办？**

> 不要怕，我们继续执行，首先它是个中间节点而且是公平锁，它有前驱节点，unpark后肯定获取不到锁(公平锁需要检测是否有前驱节点)，然后执行`shouldParkAfterFailedAcquire()`，还记得这个方法里面的一个操作么？？如果前驱节点状态>0，他就会清除这些不正常的节点，返回false，不park自旋，下一次循环这个节点在获取锁就可以获取到了，妙哉！！！

注意**setHead**是这样的

```java
private void setHead(Node node) {
    head = node;
    node.thread = null; //设置线程为null
    node.prev = null; //设置前驱为空
}
```

此时队列状态变为：

![mark](http://static.imlgw.top/blog/20190809/wFDlbQTu417I.png?imageslim)

再往后就是重复前面的过程啦。

### 非公平锁公平锁区别

上面是介绍的公平锁，所谓的公平就是先来后到FIFO。

我们来看一下非公平锁的lock和nonfairTryAcquire()的实现，这两个锁的区别其实就是这两个方法。

#### lock()

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

可以看到非公平锁`lock()`的时候，不管三七二十一先CAS试一下能不能获取到锁，获取到就直接返回

#### nonfairTryAcquire()

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

对比公平锁的实现，会发现少了`hasQueuedPredecessors()` 这个方法，所以如果前面`lock()` 的时候没有CAS成功，到这里后如果之前持有锁的线程释放了锁，它又会再次尝试CAS获取锁，这里其实就体现了非公平锁的特点，**先等待锁的线程不一定能先获取到锁，中间允许有人`"插队"`**，如果这一次还是失败了，就会和公平锁一样老老实实去等待队列中排队

一般而言，非公平锁的性能会比公平锁好，而非公平锁可能会导致排在后面的线程饥饿

### 流程图

![mark](http://static.imlgw.top/blog/20190810/6FEWGydOmVzm.png?imageslim)

## 参考

**《Java并发编程之美》**

[一行一行源码分析清楚AbstractQueuedSynchronizer](https://javadoop.com/2017/06/16/AbstractQueuedSynchronizer/)

[AbstractQueuedSynchronizer源码剖析（六）- 深刻解析与模拟线程竞争资源](https://blog.csdn.net/pfnie/article/details/53191892)

[[浅谈Java并发编程系列（八）—— LockSupport原理剖析]](https://segmentfault.com/a/1190000008420938)

[Java同步器——AQS学习](https://brightloong.github.io/2018/06/21/Java%E5%90%8C%E6%AD%A5%E5%99%A8%E2%80%94%E2%80%94AQS%E5%AD%A6%E4%B9%A0/)

