---
title:
  Java多线程基础
tags:
  [多线程,并发编程]
categories:
	[并发]
cover: http://static.imlgw.top///20190323/oGikrmykzg5w.jpg?imageslim
date: 2019/4/7 12:00
---
## 1.线程与进程区别

每个正在系统上运行的程序都是一个进程。每个进程包含一到多个线程。线程是一组指令的集合，或者是程序的特殊段，它可以在程序里独立执行。也可以把它理解为代码运行的上下文。所以线程基本上是轻量级的进程，它负责在单个程序里执行多任务。通常由操作系统负责多个线程的调度和执行。

使用线程可以把占据时间长的程序中的任务放到后台去处理，程序的运行速度可能加快，在一些等待的任务实现上如用户输入、文件读写和网络收发数据等，线程就比较有用了。在这种情况下可以释放一些珍贵的资源如内存占用等等。

如果有大量的线程,会影响性能，因为操作系统需要在它们之间切换，更多的线程需要更多的内存空间，线程的中止需要考虑其对程序运行的影响。通常块模型数据是在多个线程间共享的，需要防止线程死锁情况的发生。

总结:进程是所有线程的集合，每一个线程是进程中的一条执行路径。

## 2.为什么要使用多线程？多线程应用场景？

答:主要能体现到多线程提高程序效率。

举例: 迅雷多线程下载、数据库连接池、分批发送短信等。

## 3.线程创建方式

### 继承Thread类重写run方法

```java
class PlayGame extends Thread {
	public void run() {
		for (int i = 0; i < 50; i++) {
			System.out.println("PlayGame" + (i + 1));
		}
	}
	//void say() {}
}

class ListenMusic extends Thread {
	public void run() {
		for (int i = 0; i < 50; i++) {
			System.out.println("ListenMusic" + (i + 1));//底层操作居然是用的StringBuilder
		}
	}
}

public class ThreadDemo {
	public static void main(String[] args)
	{
		Thread music = new ListenMusic();
		Thread pg = new PlayGame();
		music.start(); //三个线程同时运行抢占资源
		try {
			pg.start();
			for (int i = 0; i < 50; i++) {
				System.out.println("    main方法:" + (i + 1));
			}
			pg.start();
		} catch (Exception e) {
			System.out.println("出现了IllegalThreadStateException异常");	  //线程只能启动一次
		}
	}
}

```
这种方法官方也不推荐使用因为Java是单继承的继承了Thread类之后就不能继承其他的类

### 实现Runnable接口

```java
class CreateRunnable implements Runnable {

	@Override
	public void run() {
		for (inti = 0; i< 10; i++) {
			System.out.println("i:" + i);
		}
	}

}

public class ThreadDemo2 {
	public static void main(String[] args) {
		System.out.println("-----多线程创建开始-----");
		// 1.创建一个线程
		CreateRunnable createThread = new CreateRunnable();
		// 2.开始执行线程 注意 开启线程不是调用run方法，而是start方法
		System.out.println("-----多线程创建启动-----");
		Thread thread = new Thread(createThread);
		thread.start();
		System.out.println("-----多线程创建结束-----");
	}
}

```
### 匿名内部类

```java
public class InClass {
	public static void main(String[] args) {
		 Thread thread = new Thread(new Runnable() {
				public void run() {
					for (int i = 0; i< 100; i++) {
						System.out.println("i:" + i);
					}
				}
			});
		thread.start();

		 //lambda表达式还是比较简洁
		 new Thread(()-> {for (int i = 0; i <100; i++) {System.out.println("lambda:" +i);}}).start();
	}
}

```
线程创建方式不只这些还有很多，后面再介绍

### Thread 类中的start() 和 run() 方法有什么区别？

这个问题经常被问到，但还是能从此区分出面试者对Java线程模型的理解程度。`start()`方法被用来启动新创建的线程，而且`start()`内部native方法`start0()`调用了run()方法，这和直接调用`run()`方法的效果不一样。当你调用`run()`方法的时候，只会是在原来的线程中调用，没有新的线程启动，`start()`方法才会启动新线程

## 4.Thread构造函数

| **常用线程api方法**| |
|:--|---|
| start()| 启动线程|
| currentThread()| 获取当前线程对象|
| getID()| 获取当前线程ID   Thread-编号 该编号从0开始|
| getName()| 获取当前线程名称|
| sleep(long mill)| 休眠线程|
| Stop（）| 停止线程,|
| **常用线程构造函数** | |
| Thread（）| 分配一个新的 Thread 对象|
| Thread（String name）| 分配一个新的 Thread对象，具有指定的 name正如其名。|
| Thread（Runnable r）| 分配一个新的 Thread对象|
| Thread（Runable r, String name）| 分配一个新的 Thread对象，具有指定的 name正如其名。|
| Thread(ThreadGroup group, Runnable target) | 分配一个新的 Thread对象，如果不传`ThreadGroup`默认加入当前线程的`ThreadGroup`中 |
| Thread(ThreadGroup group, Runnable target, String name) | 分配一个新的 `Thread`对象，使其具有  `target`作为其运行对象，具有指定的 `name`作为其名称，属于  `group`引用的线程组。 |
| Thread(ThreadGroup group, Runnable target, String name,  long stackSize) | 分配一个新的 `Thread`对象，以便它具有  `target`作为其运行对象，将指定的 `name`正如其名，以及属于该线程组由称作  `group` ，并具有指定的 *堆栈大小* |

### Thread构造方法的一些细节

直接上源码

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }

    this.name = name;

    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    if (g == null) {
        /* Determine if it's an applet or not */

        /* If there is a security manager, ask the security manager
               what to do. */
        if (security != null) {
            g = security.getThreadGroup();
        }

        /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
        if (g == null) {
            g = parent.getThreadGroup();
        }
    }

    /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
    g.checkAccess();

    /*
         * Do we have the required permissions?
         */
    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }

    g.addUnstarted();
    this.group = g;
    //这里继承了父类的一些属性
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();

    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    this.inheritedAccessControlContext =
        acc != null ? acc : AccessController.getContext();
    //传入的Runnable接口
    this.target = target;
    setPriority(priority);
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
        ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    tid = nextThreadID();
}
```

① 创建的线程会继承父线程的一些属性，是否是守护线程 ，和优先级。

②使用Thread(Runnable r)创建线程时`this.target = target`传入的`Runnable`接口在这里赋值然后会在`run方法`中被调用。

③`stacksize`这个参数会影响`虚拟机栈`的大小，这个值越大能存放的`栈帧`就越多，可达到的递归深度越深，但是这个参数不一定有效，有的平台可能并没有效果，具体的`JVM`底层的知识等后面学到再来细化

④一种奇怪的写法，这里只会执行重写`Thread`的`run`方法，这里从源码上可以看出来，传递`Runnable`接口其实是在Thread的run方法中调用了`target`的`run`方法，如果同时再`继承`Thread类，重写`run`方法，调用的就不再是Thread类的run方法，而是匿名Thread子类重写的run方法

```java
Thread t = new Thread(() -> {
    System.out.println("Runnable");
}) {
    @Override
    public void run() {
        System.out.println("Thread");
    }
};
```

④线程`tid`通过`threadSeqNumber`从0自增的来，main线程是第`10`个线程因为会有一些守护线程会在main启动前启动.

## 5.守护线程

Java中有两种线程，一种是用户线程，另一种是守护线程。

用户线程是指用户自定义创建的线程，主线程停止，用户线程不会停止

- 守护线程顾名思义当父线程结束时，守护线程也会被停止。
- JVM只有在最后一个非守护线程结束后才会退出
- 在线程start前`setDaemon(true)`方法设置为守护线程，否则就会报错
- 父线程是守护线程，子线程默认是守护线程。

```java
class Daemon implements Runnable {
    public void run() {
        for(int i=0;i<500;i++) {
            System.out.println(Thread.currentThread().getName()+i);
        }
    }
}

public class DaemonDemo {
    public static void main(String[]args) {
        System.out.println(Thread.currentThread().getPriority());
        Thread r1=new Thread(new Daemon(),"后台线程");     //设置该线程为后台线程
        r1.setDaemon(true);                        //前台线程挂掉后，后台线程就会挂掉
        for(int i=0;i<50;i++) {                   
            System.out.println("main"+i);
            if(i==10) {
                r1.start();
            }
        }
        System.out.println("主线程执行完毕");
    }
}
```

### 应用场景

`心跳检测`，在通信过程中会需要判断对方是否在线，会需要创建一条线程去做这些事情，但是如果这样会导致`主线程停止工作`了，但是检测心跳的线程仍然在继续工作，JVM就无法停下来，显然这样时不合理的，这时就可以把检测心跳的线程设置为`守护线程`，这样当它主线程停止工作时它的守护线程也会随之停止。

```java
//心跳检测MOCK
public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            Thread inThread = new Thread(() -> {
                while (true) {
                    System.out.println("start heart check");
                    try {
                        Thread.sleep(1_00);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
            //将子线程设置为守护线程
            inThread.setDaemon(true);
            inThread.start();
            try {
                Thread.sleep(1_000);
                System.err.println("Thread finish done...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        t.start();
        //主线程(main)-->t-->inThread
    }
```

## 6.线程的状态

线程从创建、运行到结束总是处于下面五个状态之一：新建状态、就绪状态、运行状态、阻塞状态及死亡状态。

![mark](http://static.imlgw.top///20181226/WXPYz3SiidMT.png?imageslim)

- 新建状态
  当用new操作符创建一个线程时， 例如new Thread(r)，线程还没有开始运行，此时线程处在新建状态。 当一个线程处于新生状态时，程序还没有开始运行线程中的代码

- 就绪状态
  一个新创建的线程并不自动开始运行，要执行线程，必须调用线程的start()方法。当线程对象调用start()方法即启动了线程，start()方法创建线程运行的系统资源，并调度线程运行run()方法。当start()方法返回后，线程就处于就绪状态。
  处于就绪状态的线程并不一定立即运行run()方法，线程还必须同其他线程竞争CPU时间，只有获得CPU时间才可以运行线程。因为在单CPU的计算机系统中，不可能同时运行多个线程，一个时刻仅有一个线程处于运行状态。因此此时可能有多个线程处于就绪状态。对多个处于就绪状态的线程是由`Java`运行时系统的线程调度程序(*thread scheduler*)来调度的。

- 运行状态
  当线程获得CPU时间后，它才进入运行状态，真正开始执行run()方法.

- 阻塞状态
  线程运行过程中，可能由于各种原因进入阻塞状态:
  1>线程通过调用sleep方法进入睡眠状态；
  2>线程调用一个在I/O上被阻塞的操作，即该操作在输入输出操作完成之前不会返回到它的调用者；
  3>线程试图得到一个锁，而该锁正被其他线程持有；
  4>线程在等待某个触发条件；

- 死亡状态

  有两个原因会导致线程死亡：

  1) run方法正常退出而自然死亡，

  2) 一个未捕获的异常终止了run方法而使线程猝死。 为了确定线程在当前是否存活着（就是要么是可运行的，要么是被阻塞了），需要使用`isAlive`方法。如果是可运行或被阻塞，这个方法返回true； 如果线程仍旧是new状态且不是可运行的， 或者线程死亡了，则返回false.

## 7.join()方法

### 源码解析

```java
 public final synchronized void join(long millis) throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
```

①`thread.join()`没有参数会默认调用`join(0)`然后回轮询检查调用`join()`的线程也就是是`thread`线程是否`isAlive()`如果thread依然存活就回释放`当前线程`的CPU执行权，然后继续轮询，知道`thread`进入终止状态。

②`join(long millis)` 参数的作用就是当前线程最多等待时间，限时等待，避免无止境的等待。

③一个线程自己调用自己的`join`方法该线程就回一直`wait`下去因为他自己要一直等自己😄

### 应用场景

>  多线程同时采集数据，最后将统计的总时间等信息存到数据库中，如果不jion就无法统一结束的时间

```java
public class ThreadJoin3 {
    public static void main(String[] args) throws InterruptedException {
        long l1 = System.currentTimeMillis();
        Thread t0 = new Thread(new CaptureMachine("M0", 1000));
        Thread t1 = new Thread(new CaptureMachine("M1", 2000));
        Thread t2 = new Thread(new CaptureMachine("M2", 4000));
        t0.start();
        t1.start();
        t2.start();
        //让主线程等待子线程结束然后统计最后总体结束的时间
        t0.join();
        t1.join();
        t2.join();
        long l = System.currentTimeMillis();
        System.out.println("end save begin timestamp:" + l1 + "end timestamp" + l);
    }
}

class CaptureMachine implements Runnable {
    private String machineId;

    private long spentTime;

    public CaptureMachine(String machineId, long spentTime) {
        this.machineId = machineId;
        this.spentTime = spentTime;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(spentTime);
            System.out.println(machineId + " capture done");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## 8.优先级

现代操作系统基本采用时分的形式调度运行的线程，线程分配得到的时间片的多少决定了线程使用处理器资源的多少，也对应了线程优先级这个概念。在JAVA线程中，通过一个int priority来控制优先级，范围为1-10，其中10最高，默认值为5。下面是Demo（基于1.8）中关于priority的一些量和方法。

```java
public class ThreadSimpleAPI2 {
    public static void main(String[] args) {
        Thread t0 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                Optional.of(Thread.currentThread().getName() + "-index" + i).ifPresent(System.out::println);
            }
        });
        t0.setPriority(Thread.MAX_PRIORITY);
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                Optional.of(Thread.currentThread().getName() + "-index" + i).ifPresent(System.out::println);
            }
        });
        t1.setPriority(Thread.NORM_PRIORITY);
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                Optional.of(Thread.currentThread().getName() + "-index" + i).ifPresent(System.out::println);
            }
        });
        t2.setPriority(Thread.MIN_PRIORITY);
        t0.start();
        t1.start();
        t2.start();
    }
}
```

## 9.Interrupt方法 

### 看看源码

```java
public void interrupt() {
        if (this != Thread.currentThread())
            checkAccess();

        synchronized (blockerLock) {
            Interruptible b = blocker;
            if (b != null) {
                interrupt0();           // Just to set the interrupt flag
                b.interrupt(this);
                return;
            }
        }
        interrupt0();
}
```

在知乎上看见一个好的回答：首先，一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己`自行停止`。所以，`Thread.stop()` ,`Thread.suspend()`,`Thread.resume()`都已经被废弃了。而`Thread.interrupt()`的作用也不是中断线程，_而是通知线程该结束了_ 具体中断还是继续运行还是由被通知的线程自己处理。具体来说，当对一个线程调用`interrupt()`时

①如果线程处于阻塞状态(sleep,wait,join等)，那么线程将~~立即退出被阻塞状态~~并抛出一个异常(2019.8.10 fix)

> 这里其实是有点问题的，在有同步锁存在的情况下，并不一定会立即退出被阻塞的状态，即使抛出异常也要等到再次拿到锁之后才能抛出，同时也不是所有的阻塞操作都会响应中断信号，比如IO操作之类的都不会响应中断信号

**验证Demo**

```java
public class WaitNotify {

    public static void main(String[] args) {

        Object object = new Object();

        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (object) {
                    System.out.println("线程1 获取到监视器锁");
                    try {
                        object.wait();
                        System.out.println("线程1 恢复啦。我为什么这么久才恢复，因为notify方法虽然早就发生了，可是我还要获取锁才能继续执行。");
                    } catch (InterruptedException e) {
                        System.out.println("线程1 wait方法抛出了InterruptedException异常，即使是异常，我也是要获取到监视器锁了才会抛出");
                    }
                }
            }
        }, "线程1");
        thread1.start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (object) {
                    System.out.println("线程2 拿到了监视器锁。为什么呢，因为线程1 在 wait 方法的时候会自动释放锁");
                    System.out.println("线程2 设置线程1 中断");
                    thread1.interrupt();
                    System.out.println("线程2 执行完了 中断，先休息3秒再说。");
                    try {
                        Thread.sleep(3000);
                        System.out.println("线程2 休息完啦。注意了，调sleep方法和wait方法不一样，不会释放监视器锁");
                    } catch (InterruptedException e) {

                    }
                    System.out.println("线程2 休息够了，结束操作");
                }
            }
        }, "线程2").start();
    }
}
```

②如果线程处于正常活动状态，那么会将该线程的`中断标志位`设置为true，仅此而已，被设置中断标志的线程将继续正常运行，不受影响。

③对已经结束的线程调用`interupt`没有任何效果

上面只是简单的分析，其实情况还是很复杂的，后面再来总结

**具体的小案例**

```java
public class ThreadInterrup2 {
    public static void main(String[] args) {
        Thread main=Thread.currentThread();

        Thread t=new Thread(()->{
            while (true){

            }
        });
        t.start();
	
        Thread t2 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //t.interrupt(); 这里打断的是t线程但是阻塞的是main线程所以打断不了，捕获不到异常
            main.interrupt();
            System.out.println("打断 main 线程");
        });
        t2.start();

        try {
            //这里阻塞的是main线程
            t.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

看懂这个小案例应该就理解interrupt了。

### 如何优雅的结束线程

**1. 使用“开关”**

```java
public class ThreadCloseGraceful {
    public static void main(String[] args) throws InterruptedException {
        Worker worker=new Worker();
        worker.start();
        Thread.sleep(10000); //等待10s
        worker.shutdown();
    }
}

class Worker extends Thread{
    //优雅的停止线程-----开关
    private volatile boolean start = true;

    @Override
    public void run() {
        while (start){

        }
    }

    public void shutdown(){
        this.start=false;
    }
}
```

> 为了及时的感知到开关的变化 start需要声明为 volatile（后面讲Volatile的时候会说到）

**2. 轮询中断标志位**

```java
public class ThreadCloseGraceful2 {
    public static void main(String[] args) throws InterruptedException {
        Worker2 worker2 = new Worker2();
        worker2.start();
        Thread.sleep(5000);
        worker2.interrupt();
    }
}

class Worker2 extends Thread{
    //优雅的停止线程2-----打断
    @Override
    public void run() {
        while (true){
            /*try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
                break; //return 会直接退出
            }*/
            //代码有可能在执行isInterrupted之前就Block了
            if(isInterrupted()){
                break;
            }
        }
        //-----------
    }
}
```

**3. 利用守护线程**

上面的代码已经提到了，有可能在轮询`开关` 或者 轮询`中断标志位`之前就堵塞了，这时也不能一直等该下去所以就需要强制结束线程的方法，`（当然不会是stop）` 这里就可以利用守护线程的特性去完成这件事

```java
public class ThreadService {
    private  Thread executeThread;

    private  volatile  boolean finished=false;

    public void execute(Runnable task){
        executeThread =new Thread(()->{
            //子线程
            Thread t=new Thread(()->{
                task.run();
            });
            t.setDaemon(true);
            t.start();

            try {
                t.join(); //这里阻塞的是executeThread
                finished=true;
                //到这里说明executeThread已经不阻塞了,子线程已经执行完了，没有超时
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println("TLE，executeThread execution was interrupted");
            }
        });
        executeThread.start();
    }

    public void shutdown(long mills){
        long base=System.currentTimeMillis();
        while (!finished){ //轮询检查标志位，看是否已经结束
            if(System.currentTimeMillis()-base>=mills){
                //超时了没有完成
                System.out.println("TLE, will end it now");
                executeThread.interrupt(); //打断executeThread
                break;
            }
            //没超时
            try {
                executeThread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println("executeThread was interrupted when shutdown");
            }
        }
        //标志位复原
        finished=false;
    }
}
```

将待执行的`task`传递到`ThreadService`中然后会创建一个`executeThread`的线程，在线程中创建一个子线程去执行`task`的`run`方法然后将子线程设置为`executeThread`的守护线程然后`join`阻塞`executeThread`线程，同时会调用`ThreadService`的`shutdown`方法传入一个最长等待时间然后`轮询标志位`检查是否结束，如果超时就会打断`executeThread`进而结束`executeThread`的子线程也就是`task`

### Thread.interrupted()

这个方法和`isInterrupt()`类似但是他会清除中断标志位为`false`方便之后的中断操作而且这个是`静态方法`，所以你用线程实例去调用这个方法没有任何意义，它这里是用来判断**当前执行线程**是否 `interrupt` ，并且设置中断标志位为`false`

```html
/**
 * Tests whether the current thread has been interrupted.  The
 * <i>interrupted status</i> of the thread is cleared by this method.  In
 * other words, if this method were to be called twice in succession, the
 * second call would return false (unless the current thread were
 * interrupted again, after the first call had cleared its interrupted
 * status and before the second call had examined it).
 *
 * <p>A thread interruption ignored because a thread was not alive
 * at the time of the interrupt will be reflected by this method
 * returning false.
 *
 * @return  <code>true</code> if the current thread has been interrupted;
 *          <code>false</code> otherwise.
 * @see #isInterrupted()
 * @revised 6.0
 */
```

同时在抛出`InterruptedException` 之后中断状态也会被自动清除为false

> if any thread has interrupted the current thread. The <i>interrupted status</i> of the current thread is cleared when this exception is thrown.     ----Thread.sleep注释

## 10.Yield方法

Thread.yield()方法的作用：暂停当前正在执行的线程，并执行其他线程。（可能没有效果）
yield()让当前正在运行的线程回到可运行状态，以允许具有相同优先级的其他线程获得运行的机会。因此，使用yield()的目的是让具有相同优先级的线程之间能够适当的轮换执行。但是，实际中无法保证yield()达到让步的目的，因为，让步的线程可能被线程调度程序再次选中。
结论：大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。

## 11.Synchronized关键字 

### 线程安全问题

```java
public class TicketRunnable implements Runnable {
    
    private final static int MAX_NO = 500;

    private int index = 1;

    @Override
    public void run() {
        while (true) {
                if (index > MAX_NO) {
                    return;
                }
                try {
                    Thread.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "第：" + index++);
        }
    }
}
```

用这个创建多个`Thread`然后运行

> 二号窗口第：497
> 三号窗口第：499
> 一号窗口第：498
> 一号窗口第：500
> 二号窗口第：`502`
> 三号窗口第：`501`

可以看出打印出了501，502明显不对为什么会出现这种问题呢？其实仔细想想也很容易理解

![mark](http://static.imlgw.top///20190326/zSGNeiCW2zc1.png?imageslim)

当两个线程如图所示的情况，2号线程`index=500`然后`index++`然后 1号线程读取到`index`的值就会产生这个问题。

### 同步代码块

**解决线程安全问题**

```java
public class TicketRunnable implements Runnable {

    private final static int MAX_NO = 500;
    
    private int index = 1;

    private final Object MONITOR = new Object();

    @Override
    public void run() {
        while (true) {
            synchronized (MONITOR) {
                //单线程
                if (index > MAX_NO) {
                    return;
                }
                try {
                    Thread.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "第：" + index++);
            }
        }
    }
}
```

利用`MONITOR`对象作为同步锁同步之后的部分就相当于单线程，`MONITOR`锁对象一般设置为`final`的避免在执行过程中对`MONITOR`对象进行改变而产生无法预料的后果

### 同步方法

```java
public class SynchronizeRunnable implements Runnable {

    private final static int MAX_NO = 500;

    private int index = 1;

    @Override
    public  void run() {
        //this锁
        while (true) {
            if (ticket()) {
                return;
            }

        }
    }

    private synchronized boolean ticket(){
        //synchronized (this) {
            //1.getFiled
            if (index > MAX_NO) {
                return true;
            }
            try {
                Thread.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // getFiled index
            // index=index+1
            // putFiled index
            //同步代码块就是保护共享数据index, MAX_NO不是,他是只读数据
            System.out.println(Thread.currentThread().getName() + "第：" + index++);
            return false;
       // }
    }
}
```

默认加的是`this`锁

**证明this锁的存在**

```java
public class SynchronizedThis {
    public static void main(String[] args) {
        ThisLock thisLock = new ThisLock();
        Thread thread = new Thread(() -> thisLock.m1(), "Thread0");
        thread.start();
        Thread thread1 = new Thread(() -> thisLock.m2(), "Thread1");
        thread1.start();
        Thread thread2 = new Thread(() -> thisLock.m3(), "Thread2");
        thread2.start();
    }
}

class ThisLock {

    private final Object LOCK = new Object();

    public synchronized void m1() {
        System.out.println(Thread.currentThread().getName());
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void m2() {
        System.out.println(Thread.currentThread().getName());
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void m3() {
        synchronized (LOCK) {
            System.out.println(Thread.currentThread().getName());
            try {
                Thread.sleep(10_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

运行上面的代码会发现`Thread0`和`Thread2`会先打印出来因为他们不是同一个锁不用等待对方，而`Thread1`会等待一段时间后才会执行因为它需要等待`Thread0`释放锁而这个锁只能是`this锁`

**证明class锁的存在**

```java
public class SynchronizedClass {
    public static void main(String[] args) {
        Thread thread0 = new Thread(() -> ClassLock.m1(), "Thread0");
        thread0.start();
        Thread thread1 = new Thread(() -> ClassLock.m2(), "Thread1");
        thread1.start();
        Thread thread2 = new Thread(() -> ClassLock.m3(), "Thread2");
        thread2.start();
    }
}

class ClassLock {
    static {
        synchronized (ClassLock.class) {
            System.out.println("static" + Thread.currentThread().getName());
            try {
                Thread.sleep(10_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static synchronized void m1() {
        System.out.println(Thread.currentThread().getName());
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static synchronized void m2() {
        System.out.println(Thread.currentThread().getName());
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void m3() {
        System.out.println(Thread.currentThread().getName());
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

运行上面的代码会发现最开始只有一个线程会去执行静态代码快，在静态代码快执行完之后`Thread2`会和另一个线程一起执行，说明一开始`Thread2`回和其中一个线程抢`Class锁`执行静态代码块（静态代码块只会执行一次）

### 死锁

> 一个线程可以持有多个锁，而这样就可能会导致`死锁`的产生

**死锁示例**

```java
public class Service1 {
    private  final Object LOCK=new Object();

    private Service2 service2;

    public  void s1(){
        synchronized (LOCK){
            System.out.println("s1==============");
        }
    }

    public void s2(){
        synchronized (LOCK){
            System.out.println("s2==============");
            service2.m2();
        }
    }

    public void setService2(Service2 service2) {
        this.service2 = service2;
    }
}
---------------------------------------------------
public class Service2 {
    private Service1 service1;

    public Service2(Service1 service1) {
        this.service1 = service1;
    }

    private  final  Object LOCK=new Object();

    public  void  m1(){
        synchronized (LOCK){
            System.out.println("m1");
            service1.s1();
        }
    }

    public  void  m2(){
        synchronized (LOCK){
            System.out.println("m2");
        }
    }
}
-------------------------------------------------
public class DeadLockTest {
    public static void main(String[] args) {
        Service1 service1=new Service1();
        Service2 service2 =new Service2(service1);
        service1.setService2(service2);

        new Thread(()->{
            while (true){
                service2.m1();
            }
        }).start();

        new Thread(()->{
            while (true){
                service1.s2();
            }
        }).start();
    }
}
```

执行上面代码就会发现在运行一段时间后两个线程都`阻塞`了，这就是`死锁`

- `jps`&`jstack`  分析死锁

![mark](http://static.imlgw.top///20190328/BG9eU9Rq8ae0.png?imageslim)

两个线程都需要对方手上的的锁，陷入僵持状态，就会产生死锁，也可以使用`jconsole`图形化界面来分析

## 12.线程间通讯

在Java平台中，Object.wait()/notify() 等方法可用于实现线程的等待和通知，wait将当前线程暂停生命周期变为 **WAITING** ，而notify() 则可以唤醒一个被暂停的线程从而实现通知，一般来说wait() 代码模板类似下面

```java
synchronized(someObj){
    while(保护条件不成立){
        //等待
        someObj.wait();
    }
    //保护条件满足
    doAction();
}
```

而`notify()` 对应代码模板如下

```java
synchronized(someObj){
    //更新等待线程的保护条件设计的共享变量
    updateSharedDate();
    //唤醒其他线程
    someObj,notify();
}
```

### 生产者消费者模型

**错误示例**

```java
public class ProduceConsumerVersion1 {
    public static void main(String[] args) {
        ProduceConsumerVersion1 pc = new ProduceConsumerVersion1();
        new Thread(() -> {
            while (true) {
                pc.produce();
            }
        }, "Produce").start();
        new Thread(() -> {
            while (true) {
                pc.consumer();
            }
        }, "Consumer").start();
    }

    private int i = 0;

    private final Object LOCK = new Object();

    public void produce() {
        synchronized (LOCK) {
            System.out.println("Produce->" + (i++));
        }
    }

    public void consumer() {
        synchronized (LOCK) {
            System.out.println("Consumer->" + (i));
        }
    }
}
```

这种模型当生产者和消费者启动后会发现两个线程无法协作，生产者不断生产，消费者`不消费`或者`重复消费`

**单生产者&单消费者**

```java
public class ProduceConsumerVersion2 {
    public static void main(String[] args) {
        ProduceConsumerVersion2 pc = new ProduceConsumerVersion2();
        Stream.of("Produce1").forEach(n -> {
            new Thread(() -> {
                while (true) {
                    pc.produce();
                }
            }, n).start();
        });
        Stream.of("Consumer1").forEach(n -> {
            new Thread(() -> {
                while (true) {
                    pc.consumer();
                }
            }, n).start();
        });
    }

    private int i = 0;

    private final Object LOCK = new Object();

    private volatile boolean isProduced = false;

    public void produce() {
        synchronized (LOCK) {
            if (isProduced) {
                //已经生产了
                try {
                    LOCK.wait();//等待消费者唤醒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "-->" + (++i));//produce
            isProduced = true;
            LOCK.notify();
        }
    }

    public void consumer() {
        synchronized (LOCK) {
            if (!isProduced) {
                try {
                    LOCK.wait();//等待生产者唤醒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "-->" + (i));//consumer
            isProduced = false;
            LOCK.notify();
        }
    }
}
```

**wait()：** 当前线程`释放锁`进入该锁对象的`等待队列` 

> Causes the current thread to wait until another thread invokes the
>
> {@link java.lang.Object#notify()} method or the
>
> {@link java.lang.Object#notifyAll()} method for this object.
>
> In other words, this method behaves exactly as if it simply
>
> performs the call {@code `wait(0)`}.
>
> The current thread `must own this object's monitor.` The thread
>
> `releases` ownership of this monitor and waits until another thread
>
> notifies threads waiting on this object's monitor to wake up
>
> either through a call to the {@code notify} method or the
>
> {@code notifyAll} method. The thread then waits until it can
>
> re-obtain ownership of the monitor and resumes execution.

**wait(long timeout)：**`wait()`的重载方法很容易想到是干啥的✔

> Causes the current thread to wait until either another thread invokes the
>
> {@link java.lang.Object#notify()} method or the
>
> {@link java.lang.Object#notifyAll()} method for this object, or a
>
> `specified amount of time has elapsed.`
>
> The current thread must own this object's monitor.
>
> @throws  `IllegalArgumentException`  if the value of timeout is negative.

**notify()：** 唤醒该`锁对象`的`等待队列`的线程，唤醒方法不同的虚拟机实现不同有的可能是`FCFS`有的可能是`SJF` 等等....所以唤醒的是那个线程是无法确定的

> Wakes up a single thread that is `waiting on this object's`
>
> `monitor`. If any threads are waiting on this object, one of them
>
> is chosen to be awakened. `The choice is arbitrary and occurs at`
>
> `the discretion of the implementation`. A thread waits on an object's
>
> monitor by calling one of the {@code wait} methods.
>
> The awakened thread will not be able to proceed until the current
>
> thread `relinquishes` the lock on this object. The awakened thread will
>
> compete in the usual manner with any other threads that might be
>
> actively competing to synchronize on this object; for example, the
>
> awakened thread enjoys no reliable privilege or disadvantage in being
>
> the next thread to lock this object.

生产者生产一个消费者消费一个，没毛病，但是上面的代码仅仅适用于`单生产者&消费者`对于多个生产者消费者就会有线程安全问题，具体问题如下

**测试多消费者&生产者**

沿用上面single p&c的代码，测试多消费者和生产者

```java
    public static void main(String[] args) {
        ProduceConsumerVersion3 pc = new ProduceConsumerVersion3();
        Stream.of("Produce1", "Produce2", "Produce3", "Produce4").forEach(n -> {
            new Thread(() -> {
                while (true) {
                    pc.produce();
                }
            }, n).start();
        });
        Stream.of("Consumer1", "Consumer2", "Consumer3").forEach(n -> {
            new Thread(() -> {
                while (true) {
                    pc.consumer();
                }
            }, n).start();
        });
    }
```

运行会发现程序进入`”死锁“`状态，用`jps&jstack`分析

![mark](http://static.imlgw.top///20190328/pzXBY0ertEVV.png?imageslim)

程序并没有发现死锁❎，这就是多生产者多消费者会产生的`假死`状态，实际上是所有的线程都进入了`wait()`状态都放弃了`CPU`的执行权

**假死原因分析**

```java
Produce1-->1	notify C1 wait
Consumer1-->1	notify P1 wait
Produce1-->2    notify C2 wait
Consumer2-->2   notify P2 wait
Produce2-->3	notify P1 wait ---> Produce1-->wait
```

上面是其中一种情况，大致分析下：前两次生产消费都正常一个`消费者`唤醒一个`生产者`，前两次执行完之后`P1 C1 C2`都进入`wait`状态然后第三次生产的时候`P2`唤醒了一个不该唤醒的人😂 唤醒了`P1`然后`wait`了，`P1`醒来后发现已经生产了然后也`wait`去了，至此所有的线程全部进入`wait`状态就造成了假死。这个问题记得大一的时候还问过老师当时特别纠结为啥会死锁，现在看看其实也没啥，主要就是`notify`唤醒的线程是不确定的，是由`JVM`决定的每种`JDK`的实现也不太一样，无法保证消费者一定唤醒生产者，反之亦然。

**多生产者&多消费者**

```java
import java.util.stream.Stream;
public class ProduceConsumerVersion3 {
    public static void main(String[] args) {
        ProduceConsumerVersion3 pc = new ProduceConsumerVersion3();
        Stream.of("Produce1", "Produce2", "Produce3", "Produce4").forEach(n -> {
            new Thread(() -> {
                while (true) {
                    pc.produce();
                }
            }, n).start();
        });
        Stream.of("Consumer1", "Consumer2", "Consumer3").forEach(n -> {
            new Thread(() -> {
                while (true) {
                    pc.consumer();
                }
            }, n).start();
        });
    }

    private int i = 0;

    private final Object LOCK = new Object();

    private volatile boolean isProduced = false;

    public void produce() {
        synchronized (LOCK) {
            while (isProduced) {
                //已经生产了
                try {
                    LOCK.wait(); //加入LOCK锁的wait队列
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "-->" + (++i));//produce
            isProduced = true;
            LOCK.notifyAll();
        }
    }

    public void consumer() {
        synchronized (LOCK) {
            while (!isProduced) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "-->" + (i));//consumer
            isProduced = false;
            LOCK.notifyAll();
        }
    }
}
```

**notifyAll()：**唤醒当前锁对象等待队列上的`所有`线程

> Wakes up `all threads` that are waiting on `this object's monitor`. A
>
> thread waits on an object's monitor by calling one of the
>
> {@code wait} methods.
>
> The awakened threads will not be able to proceed until the current
>
> thread relinquishes the lock on this object. The awakened threads
>
> will `compete in the usual manner` with any other threads that might
>
> be actively competing to synchronize on this object; for example,
>
> the awakened threads enjoy `no reliable privilege or disadvantage` in
>
> being the next thread to lock this object.

① 为了解决上面的`假死`问题这里使用了`notifyAll()`来唤醒`等待队列`的线程，看名字就知道这个方法会唤醒所有的线程，那么上面的假死问题就自然解决了。

② 还有一点不同的是这里判断生产状态时用的是`while`而不是`if`为什么不用`if`? 其实也很好理解如果有多个生产者或者消费者同时在`等待队列`中，然后其中一个抢到锁后执行，执行完生产后唤醒了所有等待的线程，假设唤醒的是`生产者`的话，因为是`if语句`控制的被唤醒的生产者抢到锁之后就直接顺着执行下去了，就直接去生产了，就会造成`重复的生产`当然用`else`语句貌似可以解决这个问题，但是那会影响效率（个人感觉），而且很别扭（被唤醒了直接退出？？？）. 所以这里用`while`来进行`二次检测`避免这种情况，这种while循环也被称为`自旋锁` 这一块后面的文章会再详细的讲。

### 为什么wait和notify必须在同步方法或同步块中调用？ 

这是`阿里巴巴`的一道面试题

① 首先从语法层面讲，如果不在同步方法和同步代码块中调用，也就是说没有加锁，自然就不用谈是不是`锁对象的持有者` ，就会报`IllegalMonitorStateException`.

> @throws  `IllegalMonitorStateException`  if the current thread is not
>
> the owner of the object's monitor.

②设想下如果不加锁可以直接调用，就会产生所谓的`竞态条件`，假设`wait()`,`notify()`,`notifyAll()`方法不需要加锁就能够被调用。此时消费者线程调用`wait()`正在进入状态变量的等待队列(译者注:可能还未进入)。在同一时刻，生产者线程调用`notify()`方法打算向消费者线程通知状态改变。那么此时消费者线程将错过这个通知并一直阻塞。因此，对象的`wait()`,`notify()`,`notifyAll()`方法必须在该对象的同步方法或同步代码块中被互斥地调用。

> 计算的正确性取决于多个线程的交替执行时序时就会产生竞态条件

### wait()后被唤醒会怎么样？

上面说到被唤醒后会去抢锁，但是这里有人可能会有疑问，去抢锁会不会回到同步的起点去争抢锁，然后把wait前的逻辑再执行一遍？这里肯定事不会的，确实是要抢锁但是会有记录会继续顺着wait方法走下去。

### notify和中断的一个很有意思的现象

```java
public class NotifyInter {

    volatile int a = 0;

    public static void main(String[] args) {

        Object object = new Object();

        NotifyInter waitNotify = new NotifyInter();

        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {

                synchronized (object) {
                    System.out.println("线程1 获取到监视器锁");
                    try {
                        object.wait();
                        System.out.println("线程1 正常恢复啦。但是 isInterrupt = "+ Thread.currentThread().isInterrupted());
                    } catch (InterruptedException e) {
                        System.out.println("线程1 wait方法抛出了InterruptedException异常");
                    }
                }
            }
        }, "线程1");
        thread1.start();

        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {

                synchronized (object) {
                    System.out.println("线程2 获取到监视器锁");
                    try {
                        object.wait();
                        System.out.println("线程2 正常恢复啦。");
                    } catch (InterruptedException e) {
                        System.out.println("线程2 wait方法抛出了InterruptedException异常");
                    }
                }
            }
        }, "线程2");
        thread2.start();

        // 这里让 thread1 和 thread2 先起来，然后再起后面的 thread3
       try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (object) {
                    System.out.println("线程3 拿到了监视器锁。");
                    System.out.println("线程3 设置线程1中断");
                    thread1.interrupt(); // 1
                    //waitNotify.a = 1; // 这行是为了禁止上下的两行中断和notify代码重排序
                    System.out.println("线程3 调用notify");
                    object.notify(); //2
                    System.out.println("线程3 调用完notify后，休息一会");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                    }
                    System.out.println("线程3 休息够了，结束同步代码块");
                }
            }
        }, "线程3").start();
    }
}
```

多执行几次可能就可能会发生如下情况，线程1被打断后居然正常的返回了！！！！线程2被阻塞住了

```java
线程1 获取到监视器锁
线程2 获取到监视器锁
线程3 拿到了监视器锁。
线程3 设置线程1中断
线程3 调用notify
线程3 调用完notify后，休息一会
线程3 休息够了，结束同步代码块
线程1 正常恢复啦。但是 isInterrupt = true
```

其实这里主要问题就是 `notify()` 和`interrupt()` 执行顺序的问题

- 如果先被打断，那么后续的notify会这个线程无效，依然会抛出异常，如果这是该锁实例上仍然有其他线程处于wait状态，那么这个notify会唤醒其中的一个，不能虚发，具体可以参考 [Java语言规范文档17.2.4](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.2.4)

  > The above specifications allow us to determine several properties having to do with the interaction of waits, notification, and interruption.
  >
  > If a thread is both notified and interrupted while waiting, it may either:
  >
  > - return normally from `wait`, while still having a pending interrupt (in other words, a call to `Thread.interrupted` would return true)
  > - return from `wait` by throwing an `InterruptedException`
  >
  > The thread may not reset its interrupt status and return normally from the call to `wait`.
  >
  > Similarly, notifications cannot be lost due to interrupts. Assume that a set *s* of threads is in the wait set of an object *m*, and another thread performs a `notify` on *m*. Then either:
  >
  > - at least one thread in *s* must return normally from `wait`, or
  > - all of the threads in *s* must exit `wait` by throwing `InterruptedException`
  >
  > Note that if a thread is both interrupted and woken via `notify`, and that thread returns from `wait` by throwing an `InterruptedException`, then some other thread in the wait set must be notified.

- 如果先被notify()，那么线程会从wait中醒来，然后中断，设置中断标志位为 true，但不会在这个wait上抛出异常，而会影响后面的阻塞操作，具体可以看下面的Demo

  ```java
  public class NotifyInter {
      volatile int a = 0;
      public static void main(String[] args) {
          Object object = new Object();
          NotifyInter waitNotify = new NotifyInter();
          Thread thread1 = new Thread(new Runnable() {
              @Override
              public void run() {
                  synchronized (object) {
                      System.out.println("线程1 获取到监视器锁");
                      try {
                          object.wait();
                          System.out.println("线程1 正常恢复啦。但是 isInterrupt = "+ Thread.currentThread().isInterrupted());
                      } catch (InterruptedException e) {
                          System.out.println("线程1 wait方法抛出了InterruptedException异常");
                      }
  
                      try {
                          TimeUnit.SECONDS.sleep(3);
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                          System.out.println("在sleep中被中断");
                      }
                  }
              }
          }, "线程1");
          thread1.start();
          // 这里让 thread1 和 thread2 先起来，然后再起后面的 thread3
         try {
              Thread.sleep(1000);
          } catch (InterruptedException e) {
          }
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  synchronized (object) {
                      System.out.println("线程3 拿到了监视器锁。");
                      System.out.println("线程3 设置线程1中断");
                      object.notify(); //2
                      waitNotify.a = 1; // 这行是为了禁止上下的两行中断和notify代码重排序
                      thread1.interrupt(); // 1
                      System.out.println("线程3 调用notify");
                      System.out.println("线程3 调用完notify后，休息一会");
                      try {
                          Thread.sleep(1000);
                      } catch (InterruptedException e) {
                      }
                      System.out.println("线程3 休息够了，结束同步代码块");
                  }
              }
          }, "线程3").start();
      }
  }
  ```

### wait()和sleep()的区别

这也是一道面试常问的题

① 首先`sleep()`是线程Thread的静态方法，`wait()`是`Object`类的实例方法

②`sleep()`不会释放锁对象，`wait()`会释放锁对象，这一点比较重要

③ 承接第二点，`wait()`会释放锁，但是要是你没有锁呢？其实就是上面语法层面说到的，所以调用`wait()`必须要`持有`锁对象否则就会报`IllegalMonitorStateException`

④`sleep()`不需要被唤醒`timeout`后会自动醒来，而`wait()`需要被其他线程唤醒（`wait(long time)`除外）

### 线程通讯综合案例

控制同一时间执行同一方法线程的数量

```java
public class ControlThreadNum {

    private static final LinkedList THREADS = new LinkedList<>();

    private final static int MAX_THREAD = 5;

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        List<Thread> worker = new ArrayList();
        //创建了十个线程，但是控制每次最多同时运行的只有5个
        Arrays.asList("M1", "M2", "M3", "M4", "M5", "M6", "M7", "M8", "M9", "M10").stream().map(ControlThreadNum::captureThread).forEach(t -> {
            t.start();
            worker.add(t);
        });
        //main线程等待worker的线程都执行完
        worker.stream().forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        Optional.of("All capture is done").ifPresent(System.out::println);
        Optional.of(System.currentTimeMillis() - start).ifPresent(System.out::println);
    }

    private static Thread captureThread(String name) {
        return new Thread(() -> {
            Optional.of("Thread " + Thread.currentThread().getName() + "  is begin").ifPresent(System.out::println);
            synchronized (THREADS) {
                while (THREADS.size() >= MAX_THREAD) {
                    try {
                        THREADS.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                //THREADS 只是用来控制数量&锁 元素是什么并不重要
                THREADS.addLast(1);
            }
            //到这里是并行
            Optional.of("Thread " + Thread.currentThread().getName() + "  is running").ifPresent(System.out::println);
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (THREADS) {
                Optional.of("Thread " + Thread.currentThread().getName() + "  is end").ifPresent(System.out::println);
                THREADS.removeLast();
                THREADS.notifyAll();
            }
        }, name);
    }
}
```

这里一开始一共创建了10个线程，但是执行的时候会控制`running`的个数小于5个，`runnning`线程个数用一个`LinkList`记录，若`size()>=5`就进入`wait`然后如果有线程`end`就会`notifyAll`唤醒进入等待状态的线程。

```java
Thread M2  is begin
Thread M3  is begin
Thread M2  is running
Thread M1  is begin
Thread M1  is running
Thread M3  is running
Thread M4  is begin
Thread M4  is running
Thread M5  is begin
Thread M6  is begin
Thread M5  is running
Thread M7  is begin
Thread M8  is begin
Thread M9  is begin
Thread M10  is begin
Thread M3  is `end`
Thread M10  is running
Thread M5  is `end`
Thread M1  is `end`
Thread M2  is `end`
Thread M4  is `end`
Thread M6  is running
Thread M7  is running
Thread M9  is running
Thread M8  is running
Thread M10  is end
Thread M8  is end
Thread M9  is end
Thread M7  is end
Thread M6  is end
All capture is done
20124
```

这样就是其实就是为了提高效率，线程并不是越多越好，线程创建太多，就会达到瓶颈，效率反而会降低，因为时间都消耗在了`线程切换`上了，当然这是在没有`线程池`的情况下，后面用`线程池`就不会这么麻烦了。

### wait()/notify的开销及问题

**过早唤醒** 

 比如生产者消费者问题中生产者生产后唤醒了生产者，其实就是过早唤醒了，过早唤醒使得那些本来无须被唤醒的等待线程也被唤醒了，从而造成资源浪费。这就好比你在人群里大喊一声“美女”，便会有许多自我感觉良好的女性回头一样——尽管你要喊的仅仅是其中某一个人，但大家却都以为你是在喊自己。过早唤醒问题可以利用JDK
1.5引入的`java.util.concurrent.locks.Condition`接口来解决，后面的文章会讲到。

**信号丢失**

信号丢失（Missed Signal）问题。如果等待线程在执行`Object.wait()`前没有先判断保护条件是否已然成立，那么有可能出现这种情形——通知线程在该等待线程进人临界区之前就已经更新了相关共享变量，使得相应的保护条件成立并进行了通知，但是此时等待线程还没有被暂停，自然也就无所谓唤醒了。这就可能造成等待线程直接执行`Object.wait()`而被暂停的时候，该线程由于没有其他线程进行通知而一直处于等待状态。这种现象就相当于等待线程错过了一个本来“发送”给它的“信号”，因此被称为信号丢失（Missed Signal）。只要将对保护条件的判断和`Object.wait()`调用放在一个循环语句之中就可以避免上述场景的信号丢失。信号丢失的另外一个表现是在应该调用`Object.notifyAll()` 的地方却调用了`Object.notify()`。比如，对于使用同一个保护条件的多个等待线程，如果通知线程在侦测到这个保护条件成立后调用的是`Object.notify()`，那么这些等待线程最多只有一个线程能够被唤醒，甚至一个也没有被唤醒——被唤醒的线程是`Object.notify()`所属对象上使用其他保护条件的一个等待线程！也就是说，尽管通知线程在调用`Object.notify()`前可能考虑（判断）了某个特定的保护条件是否成立，但是`Object.notify()`本身在其唤醒线程时是不考虑任何保护条件的！这就可能使得通知线程执行`Object.notify()`进行的通知对于使用相应保护条件的等待线程来说丢失了。这种情形下，避免信号丢失的一个方法是在必要的时候使用`Object.notifyAll()`来通知。总的来说，信号丢失本质上是一种代码错误，而不是Java标准库API自身的问题。

**欺骗性唤醒**

由于莫名其妙的原因，线程有可能在没有调用过`notify()`和`notifyAll()`的情况下醒来。这就是所谓的假唤醒（spurious wakeups），无端端地醒过来了，然而此时可能保护条件并没有成立。这个问题的解决同样是讲 保护条件和wait放在临界区内同一个循环体内就可以了。

**上下文切换**

​	首先，等待线程执行`Object.wait()`至少会导致该线程对相应对象内部锁的两次申请与释放。通知线程在执行`Object.notify()/notifyAll()`时需要持有相应对象的内部锁，因此`Object.notify()/notifyAll()`调用会导致一次锁的申请。而锁的申请与释放可能导致上下文切换。

​	其次，等待线程从被暂停到唤醒这个过程本身就会导致上下文切换。

​	再次，被唤醒的等待线程在继续运行时需要再次申请相应对象的内部锁，此时等待线程可能需要和相应对象的入口集中的其他线程以及其他新来的活跃线程（即申请相应的内部锁且处于RUNNABLE状态的线程）争用相应的内部锁，而这又可能导致上下文切换。
最后，过早唤醒问题也会导致额外的上下文切换，这是因为被过早唤醒的线程仍然需要继续等待，即再次经历被暂停和唤醒的过程。

[更多参考](http://ifeve.com/thread-signaling/)

## 13.手写一个BooleanLock

`Synchronized`的缺点其实很明显，当多个线程竞争锁的时候，当一个线程抢到锁后其他的线程只能傻傻的等着，这样会影响效率，所以这里可以自己简单手写一个限制等待时间的锁。

### LOCK接口

```java
public interface Lock {

    class TimeOutException extends Exception {
        public TimeOutException(String message) {
            super(message);
        }
    }

    void lock() throws InterruptedException;

    void lock(long time) throws InterruptedException,TimeOutException;

    void unLock() throws InterruptedException;

    Collection<Thread> getBlockThread();
}
```

定义了一个`TimeOutException`

### BooleanLock实现类

```java
public class BooleanLock implements Lock {

    //false indicated free
    private boolean initValue;
    //加锁的线程
    private Thread lockedThread;

    private Collection<Thread> blockThreadCollection = new ArrayList<>();

    public BooleanLock() {
        this.initValue = false;
    }

    @Override
    public synchronized void lock() throws InterruptedException {
        while (initValue) {
            blockThreadCollection.add(Thread.currentThread());
            System.out.println(Thread.currentThread().getName() + " is wait");
            this.wait();
        }
        blockThreadCollection.remove(Thread.currentThread());
        this.initValue = true;
        this.lockedThread = Thread.currentThread();
    }

    @Override
    public synchronized void lock(long time) throws InterruptedException, TimeOutException {
        if (time <= 0) lock();
        long remainTime=time;
        long endTime=System.currentTimeMillis()+time;
        while (initValue){
            if(remainTime<=0){
                throw new TimeOutException("time is out");
            }
            blockThreadCollection.add(Thread.currentThread());
            this.wait(time);
            remainTime=endTime-System.currentTimeMillis();
        }
        this.initValue=true;
        this.lockedThread=Thread.currentThread();
        blockThreadCollection.remove(Thread.currentThread());
    }

    @Override
    public synchronized void unLock() {
        //判断是不是加锁的线程
        if (lockedThread == Thread.currentThread()) {
            this.initValue = false;
            this.notifyAll();
            Optional.of(Thread.currentThread().getName() + "  release the lock monitor").ifPresent(System.out::println);
        }
    }

    @Override
    public Collection<Thread> getBlockThread() {
        return Collections.unmodifiableCollection(blockThreadCollection);
    }
}
```

### 测试BooleanLock的效果

```java
public class LockTest {
    public static void main(String[] args) {
        final BooleanLock booleanLock = new BooleanLock();
        Stream.of("t0", "t1", "t2").forEach(name -> {
            new Thread(() -> {
                try {
                    booleanLock.lock(10);
                    Optional.of(Thread.currentThread().getName() + " get the lock").ifPresent(System.out::println);
                    doSomething();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (Lock.TimeOutException e) {
                    System.out.println(Thread.currentThread().getName()+" Time out");
                    e.printStackTrace();
                } finally {
                    booleanLock.unLock();
                }
            }, name).start();
        });
        //main线程释放锁，不应该，谁加的锁应该由谁去释放锁
        //booleanLock.unLock();
    }

    private static void doSomething() {
        Optional.of(Thread.currentThread().getName() + " is working...").ifPresent(System.out::println);
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

这里设置在抢不到锁的时候，只等待`10ms`，然后`doSomething`会`sleep`5000ms，所以只有一个线程可以抢到锁后面的都会超时 throw `TimeOutException`典型的`限时等待`模型

```java
t0 get the lock
t0 is working...
`base_thread_study.chaper10.Lock$TimeOutException: time is out`
	at base_thread_study.chaper10.BooleanLock.lock(BooleanLock.java:41)
	at base_thread_study.chaper10.LockTest.lambda$null$0(LockTest.java:12)
	at java.lang.Thread.run(Thread.java:748)
`base_thread_study.chaper10.Lock$TimeOutException: time is out`
	at base_thread_study.chaper10.BooleanLock.lock(BooleanLock.java:41)
	at base_thread_study.chaper10.LockTest.lambda$null$0(LockTest.java:12)
	at java.lang.Thread.run(Thread.java:748)
t2 Time out
t1 Time out
t0  release the lock monitor
```

## 14.给应用程序注入钩子Hook

关于`Hook`是什么就不多介绍了，这里的钩子和`git`,`svn`里面的是一样的，类似的在使用`Tomcat`等服务的时候，在你关闭它之后它仍然会打印日志和释放一些资源，这就是`Hook`的一种，当然`Hook`有多种，这只是其中一种。

```java
public class ExitCap{
	public static void main(String []arg){
	int i=0;
	Runtime.getRuntime().addShutdownHook(new Thread(()->{
   	    System.out.println("The test app will shutdown");
	    notifyAndRelease();
	}));
	while(true){
		try{
		Thread.sleep(1000);
		System.out.println("i am working");
		}catch(Exception e){
		 //donothing
		}
		if(i>10){
		   throw new RuntimeException();
		}
		i++;    
	}
    }

    public static void notifyAndRelease(){
	System.out.println("notify to admin");
    	
	try{
	Thread.sleep(1000);
	}catch(Exception e){}
	
	System.out.println("release the resources(socker. file, connection.)");
	
	try{
        Thread.sleep(1000);
        }catch(Exception e){}
	
	System.out.println("release and notify done");
    }
}

```

这里是在`Linux`上进行的测试，因为效果比较明显，顺便也熟悉下`Linux`的命令，可以看到上面的钩子就是通过`Runtime.getRuntime().addShutdownHook()`注入了一个`Thread`进去的，这样就会检测到程序的退出并触发`Hook`做一些释放资源之类的工作。

```java
i am working
i am working
i am working
i am working
i am working
i am working
i am working
i am working
i am working
i am working
i am working
Exception in thread "main" java.lang.RuntimeException
	at base_thread_study.chaper10.ExitCap.main(ExitCap.java:18)
i am working
`The test app will shutdown`
`notify to admin`
`release the resources(socker. file, connection.)`
`release and notify done`
```

上面是在正常情况下终止线程比如 `异常`，`ctrl C`或者 `kill pid`如果使用 `kill -9 pid`就不会触发钩子，强制停止，所以一般不建议用`kill -9`

## 15.捕获线程的Runtime异常

在Java多线程环境下，所有线程都不允许抛出未捕获的`checked exception`(比如sleep的InterruptException)，也就是各个线程必须自己把自己的`checked exception`处理掉，但是如果是`unchecked exception `呢？主要就是指`RuntimeException`此类异常抛出时该线程会`shutdown`但是其它线程不受影响也无法感知到这个异常，就像下面的例子

```java
public class ThreadException {
    public static void main(String[] args) {
        Thread thread = null;
        try {
            thread=new Thread(()->{
                int res=1/0;
            });
            thread.start();
        }catch (Exception e){
            System.out.println("捕获到异常");
        }
    }
}
```

控制台输出`main线程`并没有捕获到异常，其实这也是一种很好的理念，每个线程的事情应该由线程自己去处理不应该由其他线程去干扰，正如`stop/resume/suspend`这些方法被弃用的原因。但是这些异常如果不去处理可能会导致一些严重的后果，JDK1.5之后官方也提供了API去处理线程的异常。setDefaultUncaughtExceptionHandler()和setUncaughtExceptionHandler()前者是`Thread`的静态方法，用于给所有的线程设置默认的异常处理，后者是实例方法，针对每个线程会给每个线程加上一个异常处理器，如下Demo

```java
public class ThreadException {
    private static int A = 10;
    private static int B = 0;

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
              int res = A / B;
            }
        );
        //最好在start前设置异常处理器，放在后面可能会起不到作用。
        thread.setUncaughtExceptionHandler((t, e) -> {
            System.out.println(t.getName());
            System.out.println(e);
        });
        thread.start();
    }
}
```

> `Thread-0`
> `java.lang.ArithmeticException: / by zero`

可以看到已经捕获到了这个异常，当线程遇到未捕获的异常而结束时会调用`UncaughtExceptionHandler` 处理一些"后事"和释放一些宝贵的资源，`setUncaughtExceptionHandler`建议放在线程start之前，不然可能起不到作用。

## 16.ThreadGroup线程组

### 获取线程组信息

```java
public class ThreadGroupAPI {
    public static void main(String[] args) {
        ThreadGroup tgp = new ThreadGroup("TGP1");
        Thread t = new Thread(tgp, "t0") {
            @Override
            public void run() {
                while (true) {
                    try {
                        System.out.println(getThreadGroup().getName());
                        System.out.println(getThreadGroup().getParent());
                        //可以访问，文档上说不行
                        System.out.println(getThreadGroup().getParent().activeCount());
                        Thread.sleep(10000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        t.start();

        ThreadGroup tgp2 = new ThreadGroup("TGP2");
        Thread t1 = new Thread(tgp2, "t0") {
            @Override
            public void run() {
                System.out.println(tgp.getName());
                System.out.println(tgp.activeCount());
                Thread[] threads=new Thread[tgp.activeCount()];
                tgp.enumerate(threads);
                //也可以访问
                Arrays.asList(threads).forEach(System.out::println);
            }
        };
        t1.start();
        //System.out.println(tgp2.getName());
        //System.out.println(tgp2.getParent().getName());
    }
}
```

文档上说的不能访问其他线程组的信息，这里测试的几个都可以，可能描述有点问题，`线程组`的创建类似于`线程`的创建，如果没有显示的指定线程组都会默认加到父线程的线程组中。

### 打断线程组interrupt()

> Interrupts all threads in this thread group.
>
> First, the <code>checkAccess</code> method of this thread group is
>
> called with no arguments; this may result in a security exception.
>
> This method then calls the <code>interrupt</code> method on all the
>
> threads in this thread group and in `all of its subgroups.`

打断该线程组里面所有的线程，包括子线程组的线程。

### 线程组setDaemon()

和线程的`setDaemon`不一样。

> Changes the daemon status of this thread group.
>
> First, the <code>checkAccess</code> method of this thread group is
>
> called with no arguments; this may result in a security exception.
>
> A daemon thread group is `automatically` `destroyed` when its last
>
> thread is stopped or its `last thread group is destroyed`.

当最后一个线程执行完毕后自动销毁线程组，当然与其对应的也有手动销毁的方法`destroy()`这个方法如果线程没执行完毕就调用会抛`IllegalThreadStateException`，其他的方法详细可以参考文档。

## 17.线程池

### 为什么要使用线程池

 创建和销毁线程开销大，利用好线程池可以避免cpu花费不必要的时间在这上面，从而专注于具体的任务:)

基本的线程池包括下面几部分：

①任务队列

②拒绝策略(抛出异常，直接丢弃，阻塞，临时队列)

③`init`(`min`)初始大小

④`active`中间常态大小

⑤`max`最大个数，超过就会加到任务队列中，任务队列也满就会执行拒绝策略

> min<=active<=max

### 手写线程池

#### 临时队列

```java
public class SimpleThreadPool {
    //线程池大小
    private final int size;
    //默认大小
    private final static int DEFAULT_SIZE = 10;
	//任务队列
    private final static LinkedList<Runnable> TASK_QUEUE = new LinkedList<>();
	//线程序号
    private static volatile int seq = 0;
	//线程组
    private final static ThreadGroup GROUP = new ThreadGroup("Pool_Group");
	//线程前缀名
    private final static String THREAD_PREFIX = "SIMPLE_THREAD_POOL-";
	//线程队列
    private final static List<MyThread> THREAD_QUEUE = new ArrayList<>();

    public SimpleThreadPool(int size) {
        this.size = size;
        init();
    }

    public SimpleThreadPool() {
        this(DEFAULT_SIZE);
    }

    private void init() {
        for (int i = 0; i < size; i++) {
            createThreadQueue();
        }
    }
	
    //暴露对外的接口，提交任务队列
    public void submit(Runnable runnable) {
        synchronized (TASK_QUEUE) {
            TASK_QUEUE.addLast(runnable);
            //唤醒线程池中的线程
            TASK_QUEUE.notifyAll();
        }
    }

    private void createThreadQueue() {
        MyThread thread = new MyThread(GROUP, THREAD_PREFIX + (seq++));
        thread.start();
        THREAD_QUEUE.add(thread);
    }

    private enum ThreadState {
        FREE, RUNNING, BLOCKED, DEAD
    }
	
    //包装的线程类
    private static class MyThread extends Thread {
        private volatile ThreadState threadState = ThreadState.FREE;

        public ThreadState getThreadState() {
            return this.threadState;
        }

        public MyThread(ThreadGroup group, String name) {
            super(group, name);
        }

        @Override
        public void run() {
            OUTER:
            while (this.threadState != ThreadState.DEAD) {
                //当前线程没有dead
                Runnable runnable;
                synchronized (TASK_QUEUE) {
                    while (TASK_QUEUE.isEmpty()) {
                        //任务队列为空，全员wait
                        try {
                            this.threadState = ThreadState.BLOCKED;
                            TASK_QUEUE.wait();
                        } catch (InterruptedException e) {
                            System.out.println("break");
                            break OUTER;
                        }
                    }
                    runnable = TASK_QUEUE.removeFirst();
                }
                //这里应该并行
                Optional.of(runnable).ifPresent(t -> {
                    this.threadState = ThreadState.RUNNING;
                    t.run();
                    this.threadState = ThreadState.FREE;
                });
            }
        }

        public void close() {
            this.threadState = ThreadState.DEAD;
        }
    }
}
```

最开始实现的时候`synchronized`的范围太大，将具体的执行`run`的过程也同步了起来，这明显是有问题的，只需要同步共享变量就可以了，同步了后面的代码那就跟单线程一样了。

#### 关闭线程池&拒绝策略

```java
public class SimpleThreadPool {
    //线程池大小
    private final int size;
	//任务队列大小
    private final int queueSize;
	//默认线程池大小
    private final static int DEFAULT_SIZE = 10;
	//线程池中线程编号
    private static volatile int seq = 0;
	//默认任务队列的大小
    private final static int DEFAULT_TASK_QUEUE_SIZE = 2000;
	//线程组
    private final static ThreadGroup GROUP = new ThreadGroup("Pool_Group");
	//线程名前缀
    private final static String THREAD_PREFIX = "SIMPLE_THREAD_POOL-";
	//任务队列
    private final static LinkedList<Runnable> TASK_QUEUE = new LinkedList<>();
	//线程队列
    private final static List<MyThread> THREAD_QUEUE = new ArrayList<>();
	//拒绝策略
    private final DiscardPolicy discardPolicy;
	//线程次是否销毁
    private volatile boolean destroy = false;
	//默认的拒绝策略（抛异常）
    public final static DiscardPolicy DEFAULT_DISCARD_POLICY = () -> {
        throw new DiscardException("Discard this Task!!!!(Default Policy)");
    };
    public SimpleThreadPool(int size, int queueSize, DiscardPolicy discardPolicy) {
        this.size = size;
        this.queueSize = queueSize;
        this.discardPolicy = discardPolicy;
        init();
    }

    public SimpleThreadPool() {
        this(DEFAULT_SIZE, DEFAULT_TASK_QUEUE_SIZE, DEFAULT_DISCARD_POLICY);
    }

    private void init() {
        for (int i = 0; i < size; i++) {
            createThreadQueue();
        }
    }

    public void submit(Runnable runnable) {
        if (destroy) {
            throw new IllegalStateException("The Pool is shutdown , you can't submit now ! !");
        }
        synchronized (TASK_QUEUE) {
            if (TASK_QUEUE.size() > queueSize) {
                discardPolicy.discard();
            }
            TASK_QUEUE.addLast(runnable);
            //唤醒线程池中的线程
            TASK_QUEUE.notifyAll();
        }
    }

    public void shutdown() throws InterruptedException {
        //判断任务队列是否为空
        while (!TASK_QUEUE.isEmpty()) {
            Thread.sleep(50);
        }
        int initVal = THREAD_QUEUE.size();
        while (initVal > 0) {
            for (MyThread thread : THREAD_QUEUE) {
                if (thread.getThreadState() == ThreadState.BLOCKED) {
                    thread.close();
                    thread.interrupt();
                    initVal--;
                } else {
                    Thread.sleep(10);
                }
            }
        }
        this.destroy = true;
        System.out.println("My Thread pool is shutdown");
    }

    public boolean isDestroy() {
        return this.destroy;
    }

    private void createThreadQueue() {
        MyThread thread = new MyThread(GROUP, THREAD_PREFIX + (seq++));
        thread.start();
        THREAD_QUEUE.add(thread);
    }


    private enum ThreadState {
        FREE, RUNNING, BLOCKED, DEAD
    }

    public static class DiscardException extends RuntimeException {
        public DiscardException(String message) {
            super(message);
        }
    }

    @FunctionalInterface
    public interface DiscardPolicy {
        void discard() throws DiscardException;
    }

    private static class MyThread extends Thread {
        private volatile ThreadState threadState = ThreadState.FREE;

        public ThreadState getThreadState() {
            return this.threadState;
        }

        public MyThread(ThreadGroup group, String name) {
            super(group, name);
        }

        @Override
        public void run() {
            OUTER:
            while (this.threadState != ThreadState.DEAD) {
                Runnable runnable;
                synchronized (TASK_QUEUE) {
                    while (TASK_QUEUE.isEmpty()) {
                        try {
                            this.threadState = ThreadState.BLOCKED;
                            TASK_QUEUE.wait();
                        } catch (InterruptedException e) {
                            System.out.println(Thread.currentThread().getName() + " is dead");
                            break OUTER;
                        }
                    }
                    runnable = TASK_QUEUE.removeFirst();
                }
                Optional.of(runnable).ifPresent(t -> {
                    this.threadState = ThreadState.RUNNING;
                    t.run();
                    this.threadState = ThreadState.FREE;
                });
            }
        }

        private void close() {
            this.threadState = ThreadState.DEAD;
        }
    }
}

```

- `shutdown`方法实现

①先轮询任务队列是否为空，不为空就会让`当前线程`等待`线程队列`的线程执行完所有任务。

②当任务队列为空时，遍历`线程队列`，然后打断`BLOCK`的线程并且设置为`DEAD`状态跳出循环，因为`任务队列`为空`线程队列`里面的线程都会在`TASK_QUEUE`上面`BLOCK`住，但是也存在特殊情况，可能某个线程刚拿到最后一个任务，这种情况我们可以稍微等一下，等它`BLOCK`，毕竟这是个`lg(N)-lg(N2)`的方法

③设置`destory`状态为true，然后在`submit`的时候会根据这个变量来判断是否已经销毁，如果已经销毁就会抛出一个`RunntimeException`

- `拒绝策略`实现

这里实现了一个·默认的拒绝策略，抛出异常，在submit的时候判断任务队列是不是满的，如果满了就直接抛异常，这里如果用这种方式拒绝，一但出现异常`当前线程`就会`直接结束`可能就无法关闭连接池。

#### 自动扩容&闲时回收

```java
public class SimpleThreadPool extends Thread {
    //线程池大小
    private int size;
    //线程大小变化值
    private int min;
    private int active;
    private int max;
    //默认值
    private final static int MIN = 4;
    private final static int ACTIVE = 8;
    private final static int MAX = 12;
    //任务队列大小
    private final int queueSize;

    public int getMin() {
        return min;
    }

    public int getActive() {
        return active;
    }

    public int getMax() {
        return max;
    }

    //线程池中线程编号
    private static volatile int seq = 0;
    //默认任务队列的大小
    private final static int DEFAULT_TASK_QUEUE_SIZE = 2000;

    //线程组
    private final static ThreadGroup GROUP = new ThreadGroup("Pool_Group");
    //线程名前缀
    private final static String THREAD_PREFIX = "SIMPLE_THREAD_POOL-";
    //任务队列
    private final static LinkedList<Runnable> TASK_QUEUE = new LinkedList<>();
    //线程队列
    private final static List<MyThread> THREAD_QUEUE = new ArrayList<>();
    //拒绝策略
    private final DiscardPolicy discardPolicy;
    //线程池是否销毁
    private volatile boolean destroy = false;
    //默认的拒绝策略（抛异常）
    public final static DiscardPolicy DEFAULT_DISCARD_POLICY = () -> {
        throw new DiscardException("Discard this Task!!!!(Default Policy)");
    };

    public SimpleThreadPool(int min, int active, int max, int queueSize, DiscardPolicy discardPolicy) {
        this.queueSize = queueSize;
        this.discardPolicy = discardPolicy;
        this.min = min;
        this.active = active;
        this.max = max;
        init();
    }

    public SimpleThreadPool() {
        this(MIN, ACTIVE, MAX, DEFAULT_TASK_QUEUE_SIZE, DEFAULT_DISCARD_POLICY);
    }

    //线程池的线程，维护整个线程
    @Override
    public void run() {
        while (!destroy) {
            System.err.printf("Pool#min:%d,active:%d,max:%d,currentSize:%d,taskRemain:%d\n",
                    this.min, this.active, this.max, this.size, TASK_QUEUE.size());
            try {
                Thread.sleep(5000);
                if (TASK_QUEUE.size() > active && size < active) {
                    for (int i = size; i < active; i++) {
                        createThreadQueue();
                    }
                    System.out.println("increment to active success");
                    this.size = active;
                } else if (TASK_QUEUE.size() > max && size < MAX) {
                    for (int i = size; i < max; i++) {
                        createThreadQueue();
                    }
                    System.out.println("increment to max success");
                    this.size = max;
                }
                if (TASK_QUEUE.isEmpty() && size > active) {
                    System.out.println("==================reduce=================");
                    //防止并发修改，在shutdown的时候reduce
                    synchronized (THREAD_QUEUE) {
                        int release = size - active;
                        //Itertor可以在遍历的过程中remove
                        for (Iterator<MyThread> it = THREAD_QUEUE.iterator(); it.hasNext(); ) {
                            if (release <= 0)
                                break;
                            MyThread mt = it.next();
                            //如果该线程在工作就不要打断它
                            if(mt.getThreadState()==ThreadState.RUNNING){
                                continue;
                            }
                            mt.close();
                            mt.interrupt();
                            it.remove();
                            release--;
                        }
                        this.size = active;
                    }

                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void init() {
        /*    for (int i = 0; i < size; i++) {
            createThreadQueue();
        }*/
        for (int i = 0; i < this.min; i++) {
            createThreadQueue();
        }
        this.size = min;
        this.start();
    }

    public void submit(Runnable runnable) {
        if (destroy) {
            throw new IllegalStateException("The Pool is shutdown , you can't submit now ! !");
        }
        synchronized (TASK_QUEUE) {
            if (TASK_QUEUE.size() > queueSize) {
                discardPolicy.discard();
            }
            TASK_QUEUE.addLast(runnable);
            //唤醒线程池中的线程
            TASK_QUEUE.notifyAll();
        }
    }

    public void shutdown() throws InterruptedException {
        while (!TASK_QUEUE.isEmpty()) {
            Thread.sleep(50);
        }
        int initVal = THREAD_QUEUE.size();
        synchronized (THREAD_QUEUE) {
            while (initVal > 0) {
                for (MyThread thread : THREAD_QUEUE) {
                    if (thread.getThreadState() == ThreadState.BLOCKED) {
                        //设置为DEAD状态
                        thread.close();
                        thread.interrupt();
                        initVal--;
                    } else {
                        Thread.sleep(10);
                    }
                }
            }
        }
        this.destroy = true;
        System.out.println("My Thread pool is shutdown");
    }

    public boolean isDestroy() {
        return this.destroy;
    }

    private void createThreadQueue() {
        MyThread thread = new MyThread(GROUP, THREAD_PREFIX + (seq++));
        thread.start();
        THREAD_QUEUE.add(thread);
    }


    private enum ThreadState {
        FREE, RUNNING, BLOCKED, DEAD
    }

    public static class DiscardException extends RuntimeException {
        public DiscardException(String message) {
            super(message);
        }
    }

    @FunctionalInterface
    public interface DiscardPolicy {
        void discard() throws DiscardException;
    }

    private static class MyThread extends Thread {
        private volatile ThreadState threadState = ThreadState.FREE;

        public ThreadState getThreadState() {
            return this.threadState;
        }

        public MyThread(ThreadGroup group, String name) {
            super(group, name);
        }

        @Override
        public void run() {
            OUTER:
            while (this.threadState != ThreadState.DEAD) {
                Runnable runnable;
                synchronized (TASK_QUEUE) {
                    while (TASK_QUEUE.isEmpty()) {
                        try {
                            this.threadState = ThreadState.BLOCKED;
                            TASK_QUEUE.wait();
                        } catch (InterruptedException e) {
                            System.out.println(Thread.currentThread().getName() + " is dead");
                            break OUTER;
                        }
                    }
                    runnable = TASK_QUEUE.removeFirst();
                }
                Optional.of(runnable).ifPresent(t -> {
                    this.threadState = ThreadState.RUNNING;
                    t.run();
                    this.threadState = ThreadState.FREE;
                });
            }
        }

        private void close() {
            this.threadState = ThreadState.DEAD;
        }
    }
}
```

相比上面固定的size这个版本

① 增加了三个字段用于动态的扩容，因为需要管理这些线程，所以将整个线程池也继承了`Thread`并实现了run方法，主要就是判断`TASK_QUEUE.size() > active && size < active`当前任务队列任务多于`active`并且当前线程队列线程数小于`active`，就可以扩容到active，max同理

②`TASK_QUEUE.isEmpty() && size > active` 闲时回收，任务队列没有任务，但是线程队列线程还很多，浪费了资源，所以需要`reduce`一些空闲的线程。这里有两个小细节，1.在reduce和shutdown的时候需要同步`线程队列`不然在`reduce`的时候`shutdown`会产生`并发修改异常`（一个在遍历，一个在remove）。

### 测试线程池

```java
public class ThreadPoolTest {
    public static void main(String[] args) throws InterruptedException {
        SimpleThreadPool threadPool= new SimpleThreadPool();
        IntStream.rangeClosed(0, 40)
                .forEach(i -> {
                    threadPool.submit(() -> {
                        System.out.println("The task " + i + "  runnable by thread " + Thread.currentThread().getName() + " start");
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("The task " + i + "  runnable by thread " + Thread.currentThread().getName() + " end");
                    });
                    System.out.println("submit " + i);
                });
        threadPool.shutdown();
    }
}
```

## 参考资料

- 《Java多线程编程实战指南》 
- [并发编程网](http://ifeve.com)
- [Java语言规范](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.2.4)

