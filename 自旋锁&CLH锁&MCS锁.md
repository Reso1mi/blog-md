---
title:
  自旋锁，CLH锁，MCS锁
tags:
  [多线程,并发编程]
categories:
  [并发]
date: 2019/8/10
cover: http://static.imlgw.top/blog/20190810/eUpUP2rgLLol.jpg?imageslim
---

- [自旋锁](#自旋锁)
  - [简单自旋锁](#简单自旋锁)
  - [Ticket Lock](#ticket-lock)
- [CLH锁](#clh锁)
- [MCS锁](#mcs锁)
- [总结](#总结)
- [参考](#参考)

## 自旋锁

自旋锁(spin lock)是一个典型的对临界资源的互斥手段，自旋锁是基于CAS原语的，所以它是轻量级的同步操作，它的名称来源于它的特性。自旋锁是指当一个线程尝试获取某个锁时，如果该锁已被其他线程占用，就一直循环检测锁是否被释放，而不是进入线程挂起或睡眠状态。由于自旋锁只不进行线程状态的改变（挂起线程），所以当线程竞争不激烈时，它的响应速度极快（因为`避免了线程调度的上下文切换`）。自旋锁适用于锁保护的临界区很小的情况，线程竞争不激烈的场景下。如果线程之间竞争激烈或者临界区的操作特别耗时，那么线程的自旋操作就会耗费大量的cpu资源，所以这种情况下性能就会下降明显。

###  简单自旋锁

```java
public class SimpleSpinLock {
    private AtomicReference<Thread> owner = new AtomicReference<Thread>();

    public void lock() {
        Thread currentThread = Thread.currentThread();
        // 如果锁未被占用，则设置当前线程为锁的拥有者
        // 后面解锁的就只能是当前线程
        while (!owner.compareAndSet(null, currentThread)) {
        }
    }

    public void unlock() {
        Thread currentThread = Thread.currentThread();
        // 只有锁的拥有者才能释放锁
        owner.compareAndSet(currentThread, null);
    }
}
```

**缺点**

- CAS操作需要硬件的配合（现代处理器大多都支持）

- 保证各个CPU的缓存（L1、L2、L3、跨CPU Socket、主存）的数据一致性，通讯开销很大，在多处理器系统上更严重（这是由于Atomic的volatile变量导致的，同时这也是必须的）

- 没法保证公平性，不保证等待进程/线程按照FIFO顺序获得锁。

### Ticket Lock

```java
public class TicketLock {
    private AtomicInteger serviceNum = new AtomicInteger(); // 服务号
    private AtomicInteger ticketNum = new AtomicInteger(); // 排队号

    public int lock() {
        // 首先原子性地获得一个排队号
        int myTicketNum = ticketNum.getAndIncrement();

        // 只要当前服务号不是自己的就不断轮询
        while (serviceNum.get() != myTicketNum) {
        }
        return myTicketNum;
    }

    public void unlock(int myTicket) {
        // 解锁后只有拥有该线程的下一个排队号线程才能加锁,保证了公平性,不会有插队的情况
        int next = myTicket + 1;
        // 只有当前线程拥有者才能释放锁
        serviceNum.compareAndSet(myTicket, next);
    }
}
```

**缺点**

Ticket Lock 虽然解决了公平性的问题，但是多处理器系统上，每个进程/线程占用的处理器都在读写同一个变量serviceNum ，每次读写操作都必须在多个处理器缓存之间进行缓存同步，这会导致繁重的系统总线和内存的流量，大大降低系统整体的性能。

## CLH锁

CLH的发明人是：Craig，Landin and Hagersten，三个人的名字合称

CLH锁是一种基于隐式链表（节点里面没有next指针）的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，它不断轮询前驱的状态，如果发现前驱释放了锁就结束自旋。

```java
public class CLHLock {
    public static class CLHNode {
        private volatile boolean isLocked = true; // 默认是在等待锁
    }

    private volatile CLHNode tail ;

    private static final AtomicReferenceFieldUpdater<CLHLock, CLHNode> UPDATER = AtomicReferenceFieldUpdater
            . newUpdater(CLHLock.class, CLHNode .class , "tail" );

    public void lock(CLHNode currentThreadNode) {
        //获取之前的尾结点, 然后将自己设置为尾节点
        CLHNode preNode = UPDATER.getAndSet( this, currentThreadNode);
        if(preNode != null) {//已有线程占用了锁，进入自旋
            while(preNode.isLocked ) {
            }
        }
    }

    public void unlock(CLHNode currentThreadNode) {
        // 如果队列里只有当前线程，则释放对当前线程的引用（for GC）。
        // 尝试设置尾节点为自己, 传入的期望值是自己,成功就代表队列中只有它一个
        if (!UPDATER.compareAndSet(this, currentThreadNode, null)) {
            // 还有后续线程
            currentThreadNode.isLocked = false ;// 改变状态，让后续线程结束自旋
        }
    }
}
```

这里用到了`原子字段更新器`，让tail变量可以具有CAS的功能，具体可以参考之前的文章[CAS与原子变量](http://imlgw.top/2019/04/22/cas-yu-yuan-zi-bian-liang/#%E5%AD%97%E6%AE%B5%E6%9B%B4%E6%96%B0%E5%99%A8)

**缺点**

先说一下`NUMA`和`SMP`两种处理器结构
SMP(Symmetric Multi-Processor)，即对称多处理器结构，指服务器中多个CPU对称工作，每个CPU访问内存地址所需时间相同。其主要特征是共享，包含对CPU，内存，I/O等进行共享。SMP的优点是`能够保证内存一致性`，缺点是这些共享的资源很可能成为性能瓶颈，随着CPU数量的增加，每个CPU都要访问相同的内存资源，可能导致内存访问冲突，可能会导致CPU资源的浪费。常用的PC机就属于这种。

NUMA(Non-Uniform Memory Access)非一致存储访问，将CPU分为CPU模块，每个CPU模块由多个CPU组成，并且具有独立的本地内存、I/O槽口等，模块之间可以通过互联模块相互访问，`访问本地内存的速度将远远高于访问远地内存(系统内其它节点的内存)的速度`，这也是非一致存储访问NUMA的由来。NUMA优点是可以较好地解决原来SMP系统的扩展问题，缺点是由于访问远地内存的延时远远超过本地内存，因此当CPU数量增加时，系统性能无法线性增加。

**CLH锁的缺点是在NUMA系统结构下性能很差，在这种系统结构下，每个线程有自己的内存，如果前趋结点的内存位置比较远，自旋判断前趋结点的locked域，性能将大打折扣，在SMP架构下能够保证内存一致性所以自旋判断较快**

## MCS锁

MCS Spinlock是一种基于显式链表（节点里面拥有next指针）的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，由直接前驱负责通知其结束自旋（与CLH自旋锁不同的地方，不在轮询前驱的状态，而是由前驱主动通知），从而极大地减少了不必要的处理器缓存同步的次数，降低了总线和内存的开销。而MCS是在自己的结点的locked域上自旋等待。正因为如此，它解决了CLH在NUMA系统架构中获取locked域状态内存过远的问题。

```java
/**
 * @author imlgw.top
 * @date 2019/8/10 21:49
 */
public class MCSLock {
    public static class MCSNode {
        //持有后继者的引用
        volatile MCSNode next;
        // 默认是在等待锁
        volatile boolean block = true;
    }

    volatile MCSNode tail;// 指向最后一个申请锁的MCSNode

    private static final AtomicReferenceFieldUpdater<MCSLock, MCSNode> UPDATER = AtomicReferenceFieldUpdater
            .newUpdater(MCSLock.class, MCSNode.class, "tail");

    public void lock(MCSNode currentThreadMcsNode) {
        //更新tail为最新加入的线程节点，并取出之前的节点（也就是当前节点的前驱）
        MCSNode predecessor = UPDATER.getAndSet(this, currentThreadMcsNode);
        if (predecessor != null) {
            //连接在tail的尾部
            predecessor.next = currentThreadMcsNode;
            //轮询自己的isLocked属性
            while (currentThreadMcsNode.block) {

            }
        } else {
            //前驱节点为空直接获取锁,自己是第一个
            currentThreadMcsNode.block = false;
        }
    }

    public void unlock(MCSNode currentThreadMcsNode) {
        if (currentThreadMcsNode.block) {
            return;
        }
        //判断是不是只有一个线程
        if (currentThreadMcsNode.next == null) {
            //CAS 将tail设置为空
            if (UPDATER.compareAndSet(this, currentThreadMcsNode, null)) {
                // 设置成功返回，没有其他线程等待锁
                return;
            } else {
                //CAS更新tail失败,有线程抢先一步执行lock更新了tail
                //但是可能还没有连接在 之前的tail(当前节点)后
                while (currentThreadMcsNode.next == null) {
                    //等待 predecessor.next = currentThreadMcsNode执行
                    //否则后面会报NPE
                }
            }
        }
        //修改后继者的isLocked,通知后继者结束自旋
        currentThreadMcsNode.next.block = false;
        currentThreadMcsNode.next = null;// for GC
    }
}
```

## 总结

传统的`Spin lock` 和 `Ticket Lock`都在同一个共享变量上竞争（例如SimpleSpinLock中的owner、Ticket Lock中的serviceNum），这样对给CPU保证缓存一致性带来的压力比较大，每次读写都需要同步到所有的线程，而MCS和CLH最大的优化点在于把上述同一个点上的竞争分散到队列的每个节点中去了。

## 参考

[自旋锁、排队自旋锁、MCS锁、CLH锁](https://coderbee.net/index.php/concurrent/20131115/577)