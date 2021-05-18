---
title:
  Volatile关键字详解
tags:
  [多线程,并发编程]
categories:
  [并发]
date: 2019/4/29
cover: http://static.imlgw.top///20190429/O1leMFc6AcR4.png?imageslim
---

## JMM&CPU缓存

### CPU缓存

其实这个并不是指某一个具体的部件，`寄存器(Register)`，`高速缓存(Cache)`，`写缓冲器(Store Buffer)`，`无效化队列(Invalidate Queue)`等等都可以称为 CPU缓存。

#### 为什么要有CPU缓存？

`缓存`通常意义下都是为了加快速度，这里同样也是，因为CPU的速度比`主内存(RAM)`快很多，`主内存`会拖CPU后腿影响整体的效率，所以缓存就出现了，缓存的速度比`主内存`快很多(造价高)CPU会直接通过缓存来对主内存进行读写操作，所以缓存里面实际上相当于是`主内存`的副本。

#### 使用CPU缓存带来的问题

正常情况下CPU执行计算的过程如下

1️⃣程序以及数据被加载到主内存

2️⃣指令和数据被加载到CPU缓存

3️⃣CPU执行指令，把结果写到高速缓存

4️⃣高速缓存中的数据写回主内存

如果是单核CPU，上面的步骤没有任何问题，但如果是多核CPU就可能会出现一些意料之外的问题，假设有两个核

下面这种情况也是有可能发生的

1️⃣核0读取了一个字节，根据局部性原理，它相邻的字节同样被被读入核0的缓存

2️⃣核1做了上面同样的工作，这样核0与核1的缓存拥有同样的数据

3️⃣核0修改了那个字节，被修改后，那个字节被写回核0的缓存，但是该信息并没有写回主存

4️⃣核1访问该字节，由于核0并未将数据写回主存，数据不同步

#### 解决方案

🔶LOCK# 总线锁，效率很低，同时只能有一个CPU对内存操作，其他的CPU只能干等着

🔶缓存一致性`协议`，缓存一致性协议有多种，`MESI`协议是当前最主流的缓存一致性协议

![MESI状态](http://static.imlgw.top///20190411/GYdTPdVBGCQB.png?imageslim)日常处理的大多数计算机设备都属于`嗅探(snooping)`协议，CPU缓存不仅仅在做内存传输的时候才与总线打交道，而是不停在嗅探总线上发生的数据交换，跟踪其他缓存在做什么。所以当一个缓存代表它所属的处理器去`读写内存`时，其它处理器都会得到通知，它们以此来使自己的缓存保持同步。只要某个处理器一`写内存`，其它处理器马上知道这块内存在它们的缓存段中`已失效(Invaid)`，如果这个时候有处理器想`读内存`(会被立即察觉到，因为一直在嗅探总线)，那么已修改的缓存行(Cache line)就会立即刷新到主存中，然后设置为`Share`状态，这样一来读取到的数据就不是脏数据了。

再放一张 处理器&缓存&主内存交互的图（来自组成原理书上的图）

![Cache基本结构](http://static.imlgw.top///20190415/lJzPTl3yQWBn.jpg?imageslim)

既然有了MESI协议，是不是就不需要volatile的可见性语义了？当然不是

- **并不是所有的硬件架构都提供了相同的一致性保证，JVM需要volatile统一语义**（就算是MESI，也只解决CPU缓存层面的问题，没有涉及其他层面）。
- 可见性问题不仅仅局限于CPU缓存内，JVM自己维护的`内存模型`中也有可见性问题。使用volatile做标记，可以解决JVM层面的可见性问题。
- [这个回答应该很好的解释了](https://www.zhihu.com/question/277395220) 大概就是缓存一致性协议并不能保证实时性，而有时候我们需要保证严格的实时性

### Java内存模型(JMM)

> 为了屏蔽各个操作系统和硬件的差异，使得 Java 程序在所有平台下都能达到一致的内存访问效果，所以 Java 虚拟机定义了一种 Java 内存模型。

Java内存模型(即Java Memory Model，简称JMM)本身是一种抽象的概念，并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。（这里的变量不包括局部变量和方法参数，因为那是线程私有的，不会产生竞争）

Java 虚拟机规定所有的变量都存储在主内存（Main Memory），每个线程都有自己的工作线程（Work Memory 有些地方称为线程栈）。

线程的工作内存中保存了使用到的变量的主内存副本拷贝，线程对变量的操作是在自己的工作内存中，首先要将变量从主内存拷贝的自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，而不能直接对主内存的变量进行读取赋值。

不同线程之间无法直接访问对方工作内存中的变量，需要通过主内存来进行传递。

![JMM](http://static.imlgw.top///20190411/PjL8vV724vXx.png?imageslim)

（来自 [zejian](https://blog.csdn.net/javazejian/article/details/72772461)）

工作内存实际上就是对上面**CPU缓存**的抽象。

#### 内存间交互

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。
read：把一个变量的值从主内存传输到工作内存中
load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
use：把工作内存中一个变量的值传递给执行引擎
assign：把一个从执行引擎接收到的值赋给工作内存的变量
store：把工作内存的一个变量的值传送到主内存中
write：在 store 之后执行，把 store 得到的值放入主内存的变量中
lock：作用于主内存的变量
unlock：对应lock

**JMM**还定义了一些关于`happens-before`关系的规则如下

- 代码的执行顺序，编写在前面的发生在编写在后面的。

- unlock 必须发生在lock之后

- volatile 修饰的 写操作先发生在读之前

- 传递规则 ，A 先于B B先于C A肯定先于C

- 线程启动规则，start肯定现场发生与run

- 线程中断方法，interrupt 必须发生在捕获之前

- 对象的初始化必须发生在finalize前

- 线程终结规则，所有操作都发生在线程死亡前

[参考](https://www.logicbig.com/tutorials/core-java-tutorial/java-multi-threading/happens-before.html)

## Volatile干了什么？

### 保证可见性

这里我们来看一个具体的Demo。

```java
//线程1
boolean stop = false;
while(!stop){
    //doSomething();
}
 
//线程2
stop = true;
```

其实在上一篇 [Java多线程基础](<http://imlgw.top/2019/04/07/java-duo-xian-cheng-xue-xi-bi-ji/>) 里面 "优雅的结束线程" 里面有类似的代码

![mark](http://static.imlgw.top///20190428/LQuwmhuMiwxp.png?imageslim)

当时没有说明为啥要加`Volatile`，其实这里上面的代码如果不给状态量加上 `volatile`  并且用`server`模式运行有可能就会陷入死循环，即使在主线程里面将`isCancel`修改为`true`仍然无法结束，线程陷入了死循环，永远无法停止！！!为什么会这样？？？

![mark](http://static.imlgw.top///20190410/nCcFbqyGiqIz.png?imageslim)

🔸 在`server`模式下JIT对我们的代码进行了优化（这也是为什么要用server模式运行的原因，**Client VM的编译器没有像Server VM一样执行许多复杂的优化算法**）。它会将代码优化为类似下面这样的效果

```java
if(!stop){
 	while (stop){
		//do something....
 	}
}
```

`JIT`认为只有一个线程对其进行访问，所以为了避免重复的读取状态变量`stop`就将代码进行了 [循环不变表达式外提](https://zh.wikipedia.org/wiki/%E5%BE%AA%E7%8E%AF%E4%B8%8D%E5%8F%98%E4%BB%A3%E7%A0%81%E5%A4%96%E6%8F%90)（wikipedia），而这恰恰导致了死循环

🔸另一方面，也和计算机的储存系统有关，也就是上面`CPU缓存`的问题中提到的，这里 `stop`就是共享变量，当线程①和②运行的时候先将主内存的`stop`拷贝了一份到`工作内存`中，其中一个线程修改了`stop`的值但是其他的线程无法感知到这个变化就可能会陷入死循环。

📢 `volatile`在这里起到的作用就是

① 阻止`JIT`的异常优化

② 在一个线程修改了`volatile`修饰的共享变量后会**立即刷新到主内存**当中，这个过程称为_冲刷处理器缓存_。如果一个线程在读`voaltile`修饰的变量就会使相应的处理器**必须从主内存中进行同步**，这个过程称之为_刷新处理器缓存_，从而保证了可见性，通俗的讲就是`读必须从主内存中读，写必须同步到主内存中`。

### 保证有序性

**重排序**

提到有序性就不得不说重排序，先来看一个`Demo`

```java
private boolean isReady=false;

public void writer(){
    int data=getFromXxx(); //①
    isReady=true 		//②
}

public void reader(){
    if(isReady){
        //doSth
    }
}
```

这一看似乎没有什么问题，writer线程完成后开始read，问题就出来这里，**有可能data数据还没获取到，isReady就已经是true了**也就是说②和①的执行交换了顺序，也就是所谓的**重排序**，这样的重排序将会导致不可预知的错误，而导致这种现象的来源很多，比如编译器(JIT)，处理器和存储子系统(Cache)，至于为什么要重排序，主要还是为了提升性能，当然重排序对单线程来讲是没有影响的(有影响那还得了😂)

**其实不只是上面那种比较显而易见的重排序，还有下面这种比较隐含的**

```java
public class SingletonObjectPlus {
    private static  SingletonObjectPlus singletonObjectPlus =null;
    public static SingletonObjectPlus getSingletonObject3(){
    	if(singletonObjectPlus==null){
        	synchronized(SingletonObject.class){
            	if(singletonObjectPlus==null){
                	singletonObjectPlus= new SingletonObjectPlus();
            	}
        	}
    	}
     	return singletonObjectPlus;
	}
}
```

熟悉的朋友可能看出来了，这是一个DCL单例，那它有什么问题呢？它也会被重排序么？那么会在哪里重排序呢？

上述代码确实有问题，问题在**new SingletonObjectPlus();**里面，实际上new这个操作可以划分为如下好几步

1. 分配对象所需的空间 `objRef=allocate(SingleObjectPlus.class);`
2. 初始化引用的对象 `invokeConstructor(objRef);`
3. 设置`singletonObjectPlus`指向刚分配的内存地址`singletonObjectPlus=objRef`

而这些步骤有可能就会被重排序，比如将③排到②之前，也就是对象还没有初始化完成就会被返回(已经分配空间了，就不为null了)，这样在`最外层if`判断的时候就可能会直接返回一个初始化未完成的对象

> 发生这样重排序的概率很低，并不是必然出现的，重排序也不是随意的顺序调整，而是按照一定的规则去重排序，保证不会对单线程程序运行结果造成影响，显而易见，如果两条语句之间存在依赖关系，肯定是不会重排序的，具体就是两条语句访问同一个变量地址，至少有一条为写操作，那么这两条指令就存在依赖关系就不会被重排序比如 x=1;x=2;这样的 就不会被重排序。

其实上面的问题都很好解决，只要在**isReady**和**singletonObjectPlus**上加上`volatile`就ok了，在这里volatile会禁止指令的重排序（底层通过调用处理器提供的内存屏障）

### 保障Long/Double变量写的原子性

这一点其实很容易被遗忘，实际上Java对所有除了Long和Double的变量的**读写**操作都是原子性的，包括基础类型(byte，boolean，short，float，和int)和引用类型。因为Double和Long类型的变量会占用64位，如果在`32位机器`上JVM对这种变量的读写可能就是会被分解为两个操作而在多线程的情况下就会出现问题，这里就不做演示了，知道有这么个事就行了。在加上`Volatile`之后就可以保证该操作的原子性了。

### 注意

> volatile在保障可见性的时候仅仅只能保障能够读取到该共享变量的相对新值，对于引用类型变量和数组类型的变量，volatile能保证的也仅仅是该变量本身的可见性，而对于数组中的元素，引用类型中的字段（实例变量，静态变量）则无法保证其可见性，对于这些变量可见性的保障可以利用JUC工具包中的`Atomic原子类`。

## 内存屏障

先简单了解两个指令：

- Store：将处理器缓存的数据刷新到内存中。
- Load：将内存存储的数据拷贝到处理器的缓存中。

| 屏障类型            | 指令示例                   | 说明                                                         |
| ------------------- | -------------------------- | ------------------------------------------------------------ |
| LoadLoad Barriers   | Load1;`LoadLoad`;Load2     | 该屏障确保Load1数据的装载先于Load2及其后所有装载指令的的操作 |
| StoreStore Barriers | Store1;`StoreStore`;Store2 | 该屏障确保Store1立刻刷新数据到内存(使其对其他处理器可见)的操作先于Store2及其后所有存储指令的操作 |
| LoadStore Barriers  | Load1;`LoadStore`;Store2   | 确保Load1的数据装载先于Store2及其后所有的存储指令刷新数据到内存的操作 |
| StoreLoad Barriers  | Store1;`StoreLoad`;Load2   | 该屏障确保Store1立刻刷新数据到内存的操作先于Load2及其后所有装载指令的操作。它会使该屏障之前的所有内存访问指令(存储指令和访问指令)完成之后,才执行该屏障之后的内存访问指令 |

StoreLoad Barriers同时具备其他三个屏障的效果，因此也称之为`全能屏障`（mfence），是目前大多数处理器所支持的；但是相对其他屏障，该屏障的开销相对昂贵

- 按照可见性划分，内存屏障可以分为**加载屏障**(Load Barrier)，和**存储屏障**(Store Barrier)

- 按照有序性划分可分为**获取屏障**(Acquire Barrier)和**释放屏障**(Release Barrier)

具体那个充当加载屏障，那个充当存储屏障，我并不想讨论，各种博客各种资料各有各的说法，其实关于究竟底层是如何实现，如何插入，插入的哪一种这些细节我们不用去关心，不同的CPU不同的架构实现的方式都不一样，太过深入也没有多大的意义，很多博客介绍的也`完全不同`，我们只需要知道大概的原理就行了。如果想了解更多可以参考下列文章

[聊聊原子变量、锁、内存屏障那点事](http://www.0xffffff.org/2017/02/21/40-atomic-variable-mutex-and-memory-barrier/)

[一文解决内存屏障](https://monkeysayhi.github.io/2017/12/28/%E4%B8%80%E6%96%87%E8%A7%A3%E5%86%B3%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C/)

[面试必问的volatile，你了解多少？](https://www.jianshu.com/p/506c1e38a922)

[深入理解 Java 内存模型（四）——volatile](https://www.infoq.cn/article/java-memory-model-4/?utm_source=infoq&%253Butm_medium=related_content_link&%253Butm_campaign=relatedContent_articles_clk)

### 锁与内存屏障

用**OneNote**画了几张图

![mark](http://static.imlgw.top///20190428/EUPpWHTPnlH8.png?imageslim)



实际上锁就是通过**内存屏障**来保证了有序性和可见性，通过**互斥排它**来保证了原子性

### Volatile和内存屏障

**Volatile写操作和内存屏障**

![volatile变量的写操作](http://static.imlgw.top///20190428/PImPUeLGqau6.png?imageslim)

🔔 写线程对于`volatile变量的写操作`会产生类似于`锁释放`的效果。在写完成后会`冲刷处理器缓存`将结果立即刷新到主存中让其他处理器对应的缓存行失效，让其他处理器可同步该数据

> volatile变量在原子性方面仅仅保证对被修饰的变量的读写`本身`的原子性。也就是说这个操作不能涉及任何共享变量(包括volatile变量本身)的访问，比如 volatile1=volatile2+1，volatile++ 这样的操作无法保证它的原子性，另外，voaltile可以保证`long`和`double`变量在`32位`机上写的原子性

这里我们再回头看看前面的单例的例子

①分配对象所需的空间 objRef=allocate(SingleObjectPlus.class);

②初始化引用的对象 invokeConstructor(objRef);

③设置singletonObjectPlus指向刚分配的内存地址 singletonObjectPlus=objRef

虽然这里volatile子保证了子操作③的原子性 但是①②操作只涉及到了局部变量没有涉及到共享变量，由于内存屏障的作用①②操作不可能重排序到③之后，所以可以保证在得到返回之前对象一定已经初始化完毕了，不会出现没初始化完毕就返回的情况

**Volatile读操作和内存屏障**

![volatile变量的读操作](http://static.imlgw.top///20190428/tJcC98zgp532.png?imageslim)

🔔 读线程对于`volatile变量的读操作`会产生类似于`获得锁`的效果。读volatile变量前会先`刷新处理器缓存`从主存或其他处理器缓存中`同步`该数据

> volatile只能保证读线程读到共享变量的相对新值，对于引用类型和数组类型的并不能保证实例的字段或数组的元素的相对新值，只是保障了`引用地址`的相对新值(`相对新值`表示读的过程中其他线程有可能更改了这个值，对应的还有`最新值`，读的过程中写线程无法更改)

### Volatile变量的开销&场景

**开销**

volatile变量的读写都不会导致上下文切换，所以开销比锁要小，从上面的介绍中可以看出 写一个voaltile会使该操作和该操作前的所有写操作对后面的线程是可见的，所以它的成本会比普通变量大一些但是比锁小一点，读一个volatile变量也会比锁小，但是会比普通变量大因为变量都会从内存或其他处理器高速缓存中去拿无法直接从寄存器中去拿，但是也很快了。

**应用场景**

🔶 使用volatile变量作为状态标志位，应用程序的某个状态由一个线程设置，其他线程会读取该状态作为后面操作的依据，此时用volatile作为同步机制好处就是一个线程可以及时"通知"另一个线程某种事件(例如掉线重连)而避免使用锁造成较大开销

🔶使用volatile保障可见性，一个线程更新了共享变量其他线程无需加锁也可以看到该更新

🔶volatile bean模式（下面是我的个人理解可以直接跳过）

```java
public class Person {
    private volatile String firstName;
    private volatile String lastName;
    private volatile int age;
    
    public Person(String firstName,String lastName,int age){
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
	public String getFirstName() { return firstName; }
	public String getLastName() { return lastName; }
	public int getAge() { return age; }
 
	public void setFirstName(String firstName) { 
    	this.firstName = firstName;
	}
 
	public void setLastName(String lastName) { 
    	this.lastName = lastName;
	}
 
	public void setAge(int age) { 
    	this.age = age;
	}
}
```

关于这个场景的我理解的就是会保证类似volatile Person  preson=new Person(xx,xx,xx,xx);这样的操作具有可见性或者说完整性，不会初始化一半就返回对象要么为null要么就初始化完毕，类似于上面的提到的dcl单例的例子。

🔶简易读写锁

允许读线程读取的时候写线程进行更新，典型的例子就是实现一个计数器如下

```java
public class Counter {
    private volatile int value;
    public int getValue() { return value; }
    public synchronized int increment() {
        return value++;
    }  	
}
```

想了解更多去看看IBM这篇文章[Java 理论与实践:正确使用 Volatile 变量](https://www.ibm.com/developerworks/cn/java/j-jtp06197.html)

## 一个小问题

![可见性的例子](http://static.imlgw.top///20190428/4CYfervOyHH3.png?imageslim)

上面可见性的问题，图中的代码如果循环里面加上图中框内类似的代码，会发现即使共享变量上面不加**volatile**程序依然可以正常退出，上面出现的死循环并没有出现 (我的JDK版本是1.8，不同的版本情况可能不一样)，那是不是说这些操作也达到了保证可见性的作用呢？其实仔细分析这几行代码，后面三种都会刷新或冲刷处理器缓存(print里面也是加锁了的)，我一开始觉得可能是这个原因导致的，但是按道理应该是只会保证同步块内部的变量的可见性，但是sleep并没有加锁，是个本地方法为啥还是会导致这样的结果呢？这里我也不想深究了，我感觉也没啥必要了，具体的场景下该加**volatile**还是老老实实加**volatile**，如果继续探究下可以看下[这篇文章 ](http://www.importnew.com/19434.html)。

## 参考资料

- 《Java多线程编程实战指南：核心篇》

- [javazejian](https://blog.csdn.net/javazejian/article/details/72772461)

- ...