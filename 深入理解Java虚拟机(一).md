---
title:
  深入理解Java虚拟机（一）
tags:
  [JVM]
categories:
   [JVM]
date: 2019/8/11
cover:  http://static.imlgw.top/blog/20190817/D1Ru1EJew7lf.jpg?imageslim
---



本文在 [CyC大佬](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA?id=minor-gc-%e5%92%8c-full-gc) 的博客基础之上做了一些扩充改编（ 改编不是乱编，戏说不是胡说，今年下半年.......🐵

## Java内存区域

![mark](http://static.imlgw.top/blog/20190817/rTyjnrafmg3Q.png?imageslim)

### 程序计数器

- 程序计数器是一块较小的内存空间，它可以看成当前线程执行的字节码的行号指示器

- 程序计数器位于线程独占去
- 如果线程执行的是Java方法，这个计数齐记录的是正在执行的虚拟机字节码指令的地址，如果正在执行的是native方法，这个计数器的值为undefined
- 此区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域 

### Java虚拟机栈

每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。

![CyC](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/8442519f-0b4d-48f4-8229-56f984363c69.png)

可以通过 -Xss 这个虚拟机参数来指定每个线程的 Java 虚拟机栈内存大小，在 JDK 1.4 中默认为 256K，而在 JDK 1.5+ 默认为 1M：

```java
java -Xss2M HackTheJavaCopy to clipboardErrorCopied
```

该区域可能抛出以下异常：

- 当线程请求的栈深度超过最大值，会抛出 StackOverflowError 异常；
- 栈进行动态扩展时如果无法申请到足够内存，会抛出 OutOfMemoryError 异常。

### 堆

所有对象实例以及数组都在这里分配内存，是垃圾收集的主要区域（"GC 堆"）。

现代的垃圾收集器基本都是采用分代收集算法，其主要的思想是针对不同类型的对象采取不同的垃圾回收算法。可以将堆大致分成两块：

- 新生代（Young Generation）
- 老年代（Old Generation）

堆不需要连续内存，并且可以动态增加其内存，增加失败会抛出 OutOfMemoryError 异常。

可以通过 -Xms 和 -Xmx 这两个虚拟机参数来指定一个程序的堆内存大小，第一个参数设置初始值，第二个参数设置最大值。

### 方法区

Java虚拟机规范中将其描述为堆的一个逻辑部分，但是它还有一个别名就叫做`非堆`，而在HotSpot中则称之为`永久代（Permanent Generation）`。

这一块主要用于存放**已被加载的类信息（类的版本，字段，方法，接口）**、**运行时常量池**、**静态变量**、即时编译器(JIT)编译后的代码等数据。

和堆一样不需要连续的内存，并且可以动态扩展，动态扩展失败一样会抛出 OutOfMemoryError 异常。

对这块区域进行垃圾回收的主要目标是对常量池的回收和对类的卸载，但是一般比较难实现。

`HotSpot` 虚拟机把它当成永久代来进行垃圾回收。但很难确定永久代的大小，因为它受到很多因素影响，并且每次 Full GC 之后永久代的大小都会改变，所以经常会抛出 OutOfMemoryError 异常。为了更容易管理方法区，从 JDK 1.8 开始，移除永久代，并把方法区移至元空间，它位于本地内存中，而不是虚拟机内存中。

方法区是一个 JVM 规范，永久代与元空间都是其一种实现方式。在 JDK 1.8 之后，原来永久代的数据被分到了堆和元空间中。元空间存储类的元信息，**静态变量和字符串常量池等被放入堆中**。

### 运行时常量池

运行时常量池是方法区的一部分，**Class 文件中的常量池**（编译器生成的**字面量**和**符号引用**）会在**类加载后**被放入这个区域

除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()

> When the intern method is invoked, if the pool already contains a string equal to this {@code String} object as determined by the {@link #equals(Object)} method, then the string from the pool is returned. Otherwise, this {@code String} object is added to the pool and a reference to this {@code String} object is returned.

```java
public class StringIntern {
    public static void main(String[] args) {
        //s1 s2存放在局部变量表中
        // abc 存放在常量池中（字节码常量）
        String s1="abc";
        String s2="abc";

        System.out.println(s1==s2); //true
        //new出来的一定是在堆内存中
        String s3=new String("abc");
        System.out.println(s1==s3); //false
        //运行时常量,intern将字符串添加到常量池中并且返回一个引用
        System.out.println(s1==s3.intern()); //true
        //颠覆认知的
        //实际上这里创建了两个字符串对象，一个在堆中，一个在常量池中
        System.out.println(s3==s3.intern()); //false
    }
}
```

关于String常量池 参考可以这篇 [文章](https://blog.csdn.net/qq_34115899/article/details/86583262)

或这一篇[美团技术团队](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)

### 直接内存

在 JDK 1.4 中新引入了 NIO 类，它可以使用 Native 函数库直接分配堆外内存，然后通过 Java 堆里的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在堆内存和堆外内存来回拷贝数据。

## 对象创建

![mark](http://static.imlgw.top/blog/20190811/JcRnzxY4HN5q.png?imageslim)

假设Java堆中内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式称为`“指针碰撞”`（Bump the Pointer）。如果Java堆中的内存并不是规整的，已使用的内存和空闲的内存相互交错，那就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为`“空闲列表”`（Free List）。选择哪种分配方式由`Java堆是否规整`决定，而Java堆是否规整又由所采用的`垃圾收集器是否带有压缩整理功能决定`。因此，在使用Serial、ParNew等带Compact过程的收集器时，系统采用的分配算法是指针碰撞，而使用CMS这种基于Mark-Sweep算法的收集器时，通常采用空闲列表。

### 线程安全问题

创建对象的时候需要修指针指向的位置，在并发情况下这并不是线程安全的，虚拟机可以采用CAS加上失败重试（自旋）的方式保证更新操作的原子性，但是这样会影响性能，另一种是把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在Java堆中预先分配一小块内存，称为`本地线程分配缓冲`（Thread Local Allocation Buffer，TLAB）。哪个线程要分配内存，就在哪个线程的TLAB上分配，只有TLAB用完并分配新的FLAB时，才需要同步锁定。虚拟机是否使用TLAB，可以通过-XX：+/-UseTLAB参数来设定。

###  **对象结构**

#### Object Header

对象头包括两部分

**自身运行时数据（Mark Word）**比如 对象哈希值，对象分带年龄，锁状态标志，线程持有的锁，偏向线程ID，偏向时间戳等，另一部分就是**类型指针**

#### InstanceData

实例数据，是对象真正存储的有效信息，也是在程序中定义的各个类型字段的内容，无论是从父类继承下来的还是在之类种定义的，都需要记录下来。

#### **Padding**

这一部分并不是必须的，其实就是用于内存对齐的，HotSpot自动内存管理系统要求对象起始地址必须是8个字节的整数倍，也就是对象大小必须是8的整数倍，如果不够则需要填充这部分就是Padding

### HotSpot源码

找到了openJdk的Java8对应的创建对象的[源码](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/6a0ead6dc6db/src/share/vm/interpreter/bytecodeInterpreter.cpp)，大概1966行左右

## 堆的划分

### 堆结构分代

Java虚拟机将堆内存划分为新生代、老年代和永久代，永久代是HotSpot虚拟机特有的概念（JDK1.8之后为metaspace替代永久代），它采用永久代的方式来实现方法区，其他的虚拟机实现没有这一概念，而且HotSpot也有取消永久代的趋势，在JDK 1.7中HotSpot已经开始了“去永久化”，把原本放在永久代的字符串常量池移出。永久代主要存放常量、类信息、静态变量等数据，与垃圾回收关系不大，新生代和老年代是垃圾回收的主要区域。

**新生代（Young Generation）**

新生成的对象优先存放在新生代中，新生代对象朝生夕死，存活率很低，在新生代中，常规应用进行一次垃圾收集一般可以回收70% ~ 95% 的空间，回收效率很高。

**老年代（Old Generationn）**

在新生代中经历了多次（具体看虚拟机配置的阀值）GC后仍然存活下来的对象会进入老年代中。老年代中的对象生命周期较长，存活率比较高，在老年代中进行GC的频率相对而言较低，而且回收的速度也比较慢。

**永久代（Permanent Generationn）**

永久代存储类信息、常量、静态变量、即时编译器编译后的代码等数据，对这一区域而言，Java虚拟机规范指出可以不进行垃圾收集，一般而言不会进行垃圾回收。

### 堆结构分代的意义

　　Java虚拟机根据对象存活的周期不同，把堆内存划分为几块，一般分为新生代、老年代和永久代（`对HotSpot虚拟机而言`），这就是JVM的内存分代策略。
　　堆内存是虚拟机管理的内存中最大的一块，也是`垃圾回收最频繁`的一块区域，我们程序所有的对象实例都存放在堆内存中。给堆内存分代是为了提高对象内存分配和垃圾回收的效率。试想一下，如果堆内存没有区域划分，所有的新创建的对象和生命周期很长的对象放在一起，随着程序的执行，堆内存需要频繁进行垃圾收集，而每次回收都要遍历所有的对象，遍历这些对象所花费的时间代价是巨大的，会严重影响我们的GC效率。
　　有了内存分代，情况就不同了，新创建的对象会在新生代中分配内存，经过多次回收仍然存活下来的对象存放在`老年代`中，静态属性、类信息等存放在永久代中，新生代中的对象存活时间短，只需要在新生代区域中频繁进行GC，老年代中对象生命周期长，内存回收的频率相对较低，不需要频繁进行回收，永久代中回收效果太差，一般不进行垃圾回收，还可以根据不同年代的特点采用合适的垃圾收集算法。分代收集大大提升了收集效率，这些都是内存分代带来的好处。

## 垃圾收集

垃圾收集主要是针对`堆和方法区`进行。程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在于线程的生命周期内，线程结束之后就会消失，因此不需要对这三个区域进行垃圾回收。

### 判断一个对象是否可以被回收

在堆中存放着Java世界中几乎所有的对象实例，垃圾收集器在对堆进行回收的时候，第一件事情就是要确定这些对象之中那些还“活着”，那些已经“死去”（不会再被使用）

#### **引用计数法**

给对象加上一个引用计数器，每当有一个地方引用它的时候，计数器值就加1，引用失效的时候就减一，计数器为0的对象就是不可能再被使用的。

但是在主流的Java虚拟机中都没有使用引用计数器来进行内存管理，主要的原因就是它很难解决循环引用的问题。

```java
public class Test {
    public Object instance = null;
    public static void main(String[] args) {
        Test a = new Test();
        Test b = new Test();
        a.instance = b;
        b.instance = a;
        a = null;
        b = null;
        doSomething();
    }
}
```

上面的代码中 `Test a`  和`Test b` 互相引用，在后续将两个对象引用赋为null后两个对象的引用计数器仍然不为0，导致无法回收这两个对象。

#### **可达性分析法**

以 `GC Roots` 为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收，Java 虚拟机使用该算法来判断对象是否可被回收。

Java语言中可以作为GC Root的对象包括下面几种

- 虚拟机栈中局部变量表中引用的对象
- 本地方法栈中 JNI 中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象

#### OopMap，安全点，安全区

采用可达性分析法首先就找找到那些是`GC Root`，目前主要有两种查找 GC Roots 的方法：

💡**保守式 GC**：遍历方法区和栈区查找，无法使用复制算法，除非采用`句柄访问对象的方式`但效率不高，无法准确区分是不是引用（指针）类型的变量

💡**准确式 GC**：与保守式GC相对的就是准确式GC，就是我们准确的知道，某个位置上面是否是指针，对于java来说，就是知道对于某个位置上的数据是什么类型的，这样就可以判断出所有的位置上的数据是不是指向GC堆的引用，包括栈和寄存器里的数据。HotSpot则是通过 `OopMap` 数据结构来记录在对象内的什么偏移量上是什么类型的数据

很明显，保守式 GC 的成本太高。准确式 GC 的优点就是能够让虚拟机快速定位到 GC Roots。

对应 `OopMap` 的位置即可作为一个`Safe Point`（安全点）。

**什么是安全点？**

OopMap的作用是为了在GC的时候，快速进行可达性分析，所以OopMap并不需要一发生改变就去更新这个映射表。只要这个`更新在GC发生之前`就可以了。所以OopMap只需要在预先选定的一些位置上记录变化的OopMap就行了。这些特定的点就是`SafePoint`（安全点）。由此也可以知道，程序并不是在所有的位置上都可以进行GC的，只有在达到这样的安全点才能暂停下来进行GC。

**安全点的选取**

在执行 GC 操作时，所有的工作线程必须停顿，这就是所谓的`"Stop-The-World"`，因为可达性分析算法必须是在一个确保一致性的内存快照中进行。如果在分析的过程中对象引用关系还在不断变化，分析结果的准确性就不能保证。

安全点意味着在这个点时，所有`工作线程的状态是确定`的，JVM 就可以安全地执行 GC 。

安全点的选取一般在以下几个位置，避免程序过长时间执行。

- 循环的末尾

- 方法临返回前

- 调用方法之后

- 抛异常的位置

**安全区**

​	安全点的使用似乎解决了OopMap计算的效率的问题，但是这里还有一个问题。安全点需要程序自己跑过去，那么对于那些已经停在路边休息或者看风景的程序（比如那些处在Sleep或者Blocked状态的线程），他们可能并不会在很短的时间内跑到安全点去。所以这里为了解决这个问题，又引入了安全区域的概念。

​	安全区域很好理解，就是在程序的一段代码片段中并`不会导致引用关系发生变化`，也就不用去更新OopMap表了，那么在这段代码区域内任何地方进行GC都是没有问题的。这段区域就称之为安全区域。线程执行的过程中，如果进入到安全区域内，就会标志自己已经进行到安全区域了。那么虚拟机要进行GC的时候，发现该线程已经运行到安全区域，就不会管该线程的死活了。所以，该线程在脱离安全区域的时候，要`自己检查`系统是否已经完成了GC或者根节点枚举（这个跟GC的算法有关系），如果完成了就继续执行，如果未完成，它就必须等待收到可以安全离开安全区域的Safe Region的信号为止。

**中断方式**

在程序需要GC的时候怎么让所有线程到达安全点中断然后进行GC ?

- 抢断式中断：抢断式中断就是在GC的时候，让所有的线程都中断，如果这些线程中发现中断地方不在安全点上的，就恢复线程，让他们重新跑起来，直到跑到安全点上。（现在几乎没有虚拟机采用这种方式）

- 主动式中断：主动式中断在GC的时候，不会主动去中断线程，仅仅是设置一个标志，当程序运行到安全点时就去轮训该位置，发现该位置被设置为真时就自己中断挂起。所以轮训标志的地方是和安全点重合的，另外创建对象需要分配内存的地方也需要轮询该位置。

#### 方法区的回收

方法区（HotSpot中的永久代），在Java规范中并没有要求虚拟机对该区域进行垃圾回收，这个区域的垃圾回收效率要远低于在`堆中` 

回收的主要对象：

- 废弃常量
- 无用的类(Class对象)

判断一个常量是否是废弃常量，只要没有任何对象引用常量池中的常量，该常量就可以回收。

判断一个类是不是无用的类条件则比较苛刻。

- 该类所有的实例都已经被回收，此时堆中不存在该类的任何实例。
- 加载该类的 ClassLoader 已经被回收。
- 该类对应的 Class 对象没有在任何地方被引用，也就无法在任何地方通过反射访问该类方法。

满足了上述条件虚拟机才**可以** 对其进行回收，但是这也并不是必然的。

#### Finalize()

这个方法忘了就好😁

当一个对象可被回收时，如果重写该对象的 finalize() 方法，那么就有可能在该方法中让对象重新被引用，从而实现自救。自救只能进行一次，如果回收的对象之前调用了 finalize() 方法自救，后面回收时不会再调用该方法。

### 引用类型

无论是通过引用计数算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。

Java 提供了四种强度不同的引用类型。

#### 强引用

被强引用关联的对象不会被回收。

使用 new 一个新对象的方式来创建强引用。

```java
Object obj = new Object();Copy to clipboardErrorCopied
```

#### 软引用

被软引用关联的对象只有在内存不够的情况下才会被回收。

使用 SoftReference 类来创建软引用。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联Copy to clipboardErrorCopied
```

#### 弱引用

被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。

使用 WeakReference 类来创建弱引用。

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;Copy to clipboardErrorCopied
```

####  虚引用

又称为幽灵引用或者幻影引用，一个对象是否有虚引用的存在，不会对其生存时间造成影响，也无法通过虚引用得到一个对象。

为一个对象设置虚引用的唯一目的是能在这个对象被回收时收到一个系统通知。

使用 PhantomReference 来创建虚引用。

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj, null);
obj = null;
```

### 垃圾收集算法

#### 标记 - 清除算法

![CyC](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/005b481b-502b-4e3f-985d-d043c2b330aa.png)



在标记阶段，程序会检查每个对象是否为活动对象，如果是活动对象，则程序会在对象头部打上标记。

在清除阶段，会遍历整个堆进行对象回收并取消标志位，另外，还会判断回收后的分块与前一个空闲分块是否连续，若连续，会合并这两个分块。回收对象就是把对象作为分块，连接到被称为 `“空闲链表”` 的单向链表，之后进行分配时只需要遍历这个空闲链表，就可以找到分块。

在分配时，程序会搜索空闲链表寻找空间大于等于新对象大小 size 的块 block。如果它找到的块等于 size，会直接返回这个分块；如果找到的块大于 size，会将块分割成大小为 size 与 (block - size) 的两部分，返回大小为 size 的分块，并把大小为 (block - size) 的块返回给空闲链表。

详细可以参考 [垃圾回收的算法与实现](http://www.ituring.com.cn/book/tupubarticle/10955)

#### 复制算法

![CyC](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/b2b77b9e-958c-4016-8ae5-9c6edd83871e.png)

将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理。

主要不足是`只使用了内存的一半`。

现在的商业虚拟机都采用这种收集算法回收新生代，但是并不是划分为大小相等的两块，而是一块`较大的 Eden` 空间和`两块较小的 Survivor 空间`，每次使用 Eden 和其中一块 Survivor。在回收时，将 Eden 和 Survivor 中还存活着的对象全部复制到另一块 Survivor 上，`最后清理 Eden 和使用过的那一块 Survivor。`

HotSpot 虚拟机的 Eden 和 Survivor 大小比例默认为 8:1，保证了内存的利用率达到 90%。如果每次回收有多于 10% 的对象存活，那么一块 Survivor 就不够用了，此时需要`依赖于老年代进行空间分配担保`，也就是借用老年代的空间存储放不下的对象。

#### 标记 - 整理

复制算法在对象存活率较高的时候就要进行很多的复制操作，效率将会变低，老年代也没有额外的空间做担保，所以老年代一般不能直接使用这种算法。

![CyC](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ccd773a5-ad38-4022-895c-7ac318f31437.png)

让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

- 不会产生内存碎片

- 需要移动大量对象，处理效率比较低。

#### 分代收集

分代收集顾名思义就是分代来收集，针对不同的代执行不同的收集算法。

一般将堆分为新生代和老年代。

- 新生代使用：少量存活对象，采用复制算法只需要付出少量的复制成本就可以完成收集。
- 老年代使用：存活率高没有担保，采用标记 - 清除 或者 标记 - 整理 算法，

### 垃圾收集器

![mark](http://static.imlgw.top/blog/20190813/3f2uQGV4SjI4.png?imageslim)

以上是 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以配合使用。

#### Serial收集器

**Serial（串行）**收集器是最基本、发展历史最悠久的收集器，它是采用`复制算法`的`新生代收集器`，曾经（JDK 1.3.1之前）是虚拟机`新生代`收集的唯一选择。它是一个单线程收集器，只会使用一个CPU或一条收集线程去完成垃圾收集工作，更重要的是**它在进行垃圾收集时，必须暂停其他所有的工作线程，直至Serial收集器收集结束为止（“Stop The World”）**。这项工作是由虚拟机在后台自动发起和自动完成的，在用户不可见的情况下把用户正常工作的线程全部停掉，这对很多应用来说是难以接收的。

下图展示了Serial 收集器（老年代采用Serial Old收集器）的运行过程：

![mark](http://static.imlgw.top/blog/20190813/03PTqvGk6jdy.png?imageslim)

为了消除或减少工作线程因内存回收而导致的停顿，HotSpot虚拟机开发团队在JDK 1.3之后的Java发展历程中研发出了各种其他的优秀收集器，这些将在稍后介绍。但是这些收集器的诞生并不意味着Serial收集器已经“老而无用”，实际上到现在为止，它依然是**HotSpot虚拟机运行在Client模式下的默认的新生代收集器**。它也有着优于其他收集器的地方：**简单而高效（与其他收集器的单线程相比），对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得更高的单线程收集效率。**

在用户的桌面应用场景中，分配给虚拟机管理的内存一般不会很大，收集几十兆甚至一两百兆的新生代（仅仅是新生代使用的内存，桌面应用基本不会再大了），停顿时间完全可以控制在几十毫秒最多一百毫秒以内，只要不频繁发生，这点停顿时间可以接收。所以，Serial收集器对于运行在Client模式下的虚拟机来说是一个很好的选择。

#### ParNew 收集器

`ParNew`收集器就是Serial收集器的多线程版本，它也是一个`新生代收集器`。除了使用多线程进行垃圾收集外，其余行为包括Serial收集器可用的所有控制参数、收集算法（复制算法）、Stop The World、对象分配规则、回收策略等与Serial收集器完全相同，两者共用了相当多的代码。

ParNew收集器的工作过程如下图（老年代采用Serial Old收集器）：

![mark](http://static.imlgw.top/blog/20190813/NDuVa0QjwwYM.png?imageslim)

ParNew收集器除了使用多线程收集外，其他与Serial收集器相比并无太多创新之处，但它却是许多运行在Server模式下的虚拟机中首选的新生代收集器，其中有一个与性能无关的重要原因是，`除了Serial收集器外，目前只有它能和CMS收集器（Concurrent Mark Sweep）配合工作`，CMS收集器是JDK 1.5推出的一个具有划时代意义的收集器，具体内容将在稍后进行介绍。

ParNew 收集器在`单CPU的环境`中绝对不会有比Serial收集器有更好的效果，甚至由于存在线程交互的开销，该收集器在通过超线程技术实现的两个CPU的环境中都不能百分之百地保证可以超越。在多CPU环境下，随着CPU的数量增加，它对于GC时系统资源的有效利用是很有好处的。它默认开启的收集线程数与CPU的数量相同，在CPU非常多的情况下可使用`-XX:ParallerGCThreads`参数设置。

#### Parallel Scavenge 收集器

`Parallel Scavenge`收集器也是一个并行的多线程`新生代收集器`，它也使用`复制算法`。Parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标是达到一个可控制的吞吐量（Throughput）。

> **吞吐量**：CPU用于运行用户代码的时间和CPU消耗的总时间（CPU执行时间+垃圾收集时间）的比值
>
> **单线程与多线程**：单线程指的是垃圾收集器只使用一个线程，而多线程使用多个线程；
>
> **并行（Parallel）**：指多条垃圾收集线程同时执行，但是此时用户线程仍然处于等待状态，也就是说和用户线程是串行的。
>
> **并发（Concurrent）**：并发指的是垃圾收集器和用户程序同时执行。除了 CMS 和 G1 之外，其它垃圾收集器都是以并行的方式执行。

`停顿时间越短就越适合需要与用户交互的程序`，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在`后台运算而不需要太多交互的任务`。

`Parallel Scavenge`  提供了两个参数用于较精确控制吞吐量

- `-XX:MaxGCPauseMills` 控制最大垃圾收集停顿时间

- `-XX:GCTimeRatio` 直接设置吞吐量大小

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的：新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

Parallel Scavenge收集器除了会显而易见地提供可以精确控制吞吐量的参数，还提供了一个参数`-XX:+UseAdaptiveSizePolicy`，这是一个开关参数，打开参数后，就不需要手工指定新生代的大小（-Xmn）、Eden和Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量，这种方式称为`GC自适应的调节策略（GC Ergonomics）`。自适应调节策略也是Parallel Scavenge收集器与ParNew收集器的一个重要区别

另外值得注意的一点是，Parallel Scavenge收集器`无法与CMS收集器配合使用`，所以在JDK 1.6推出Parallel Old之前，如果新生代选择Parallel Scavenge收集器，老年代只有Serial Old收集器能与之配合使用。

#### Serial Old收集器

Serial收集器的老年代版本，同样是单线程，使用标记整理算法

![mark](http://static.imlgw.top/blog/20190813/y1H9NyikWogV.png?imageslim)

这个收集器的主要意义是给 Client 场景下的虚拟机使用。如果用在 Server 场景下，它有两大用途：

- 在 JDK 1.5 以及之前版本（Parallel Old 诞生以前）中与 Parallel Scavenge 收集器搭配使用。
- 作为 CMS 收集器的后备预案，在并发收集发生 `Concurrent Mode Failure` 时使用（下面会介绍）。

#### Parallel Old收集器

是 Parallel Scavenge 收集器的老年代版本。

![mark](http://static.imlgw.top/blog/20190813/itz21ThRoKAD.png?imageslim)

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器

#### CMS收集器（Concurrent Mark Sweep）

并发的老年代收集器，Mark Sweep 指的是标记 - 清除算法，它的运作相对前面几种来说要更加复杂一些，整体分为4哥步骤

- **初始标记**：仅仅是标记一下GC Root能直接关联到的对象，速度很快，需要停顿
- **并发标记**：进行GC Root Tracing的过程，它在整个回收过程中耗时最长，不需要停顿，
- **重新标记**：为了`修正`并发标记阶段程序继续运行导致标记变化的那一部分对象的标记记录，需要停顿
- **并发清除**：不需要停顿

在整个过程中耗时最长的`并发标记`和`并发清除`过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

![mark](http://static.imlgw.top/blog/20190813/zJiEhyXRjfTO.png?imageslim)

**缺点**

- 对CPU资源非常敏感 其实，面向并发设计的程序都对CPU资源比较敏感。在并发阶段，它虽然不会导致用户线程停顿，但会因为占用了一部分线程（或者说CPU资源）而导致应用程序变慢，总吞吐量会降低。CMS默认启动的回收线程数是（CPU数量+3）/4，也就是当CPU在4个以上时，并发回收时垃圾收集线程不少于25%的CPU资源，并且随着CPU数量的增加而下降。但是当CPU不足4个时（比如2个），CMS对用户程序的影响就可能变得很大，如果本来CPU负载就比较大，还要分出一半的运算能力去执行收集器线程，就可能导致用户程序的执行速度忽然降低了50%，其实也让人无法接受，后面出现了`i-CMS`让GC线程和用户线程交替运行，但是表现很一般已被弃用。

- 无法处理`浮动垃圾`，可能出现 `Concurrent Mode Failure`。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存给用户线程使用，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够用户线程运行，就会出现 `Concurrent Mode Failure`，jdk1.6之后虚拟机会临时启用 Serial Old 来替代 CMS。

- 标记-清除算法导致的空间碎片 CMS是一款基于“标记-清除”算法实现的收集器，这意味着收集结束时会有大量空间碎片产生。空间碎片过多时，将会给大对象分配带来很大麻烦，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发`Full GC`

#### G1收集器

G1（Garbage-First），它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 G1 可以直接对新生代和老年代一起回收。

![mark](http://static.imlgw.top/blog/20190814/o8sxTPY30GnJ.png?imageslim)

G1 把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离。

![mark](http://static.imlgw.top/blog/20190814/3olH4WWVEysA.png?imageslim)

通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间（这两个值是通过过去回收的经验获得），并维护一个`优先列表`，每次根据允许的收集时间，优先回收价值最大的 Region。

但是即使是这样分了Region垃圾回收器就一定会以Region为单位分配么？显然是不可能的，Region不可能完全独立，一个对象分配在Region中并不是只能被当前Region部分引用，而是可以与整个Java堆的任意对象发生引用关系，这样在做可达性分析的时候岂不是仍然需要遍历整个堆? 这个问题并不是只在G1中存在，其他收集器也会存在，只是G1这样划分之后问题更加突出了，在`G1收集器`中每个 Region 都有一个 Remembered Set，用来记录该 Region 对象`的引用对象`所在的 Region。通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描。

如果不计算维护Remembered Set的操作，G1收集器的运作大致可划分为以下几个步骤：

- **初始标记（Initial Marking）** 仅仅只是标记一下GC Roots 能直接关联到的对象，并且修改TAMS（Nest Top Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可以的Region中创建对象，此阶段需要停顿线程，但耗时很短。
- **并发标记（Concurrent Marking）** 从GC Root 开始对堆中对象进行可达性分析，找到存活对象，此阶段耗时较长，但可与用户程序并发执行。
- **最终标记（Final Marking）** 为了修正在`并发标记期间`因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这阶段需要`停顿线程`，但是可并行执行。
- **筛选回收（Live Data Counting and Evacuation）** 首先对各个Region中的回收价值和成本进行排序，根据用户所期望的GC 停顿是时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，`但是因为只回收一部分Region，时间是用户可控制的`，而且停顿用户线程将大幅度提高收集效率。

**特点**

- **并行与并发** G1 能充分利用多CPU、多核环境下的硬件优势，使用多个CPU来缩短“Stop The World”停顿时间，部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让Java程序继续执行。
- **分代收集** 与其他收集器一样，分代概念在G1中依然得以保留。虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但它能够采用不同方式去处理新创建的对象和已存活一段时间、熬过多次GC的旧对象来获取更好的收集效果。
- **空间整合** G1从整体来看是基于`“标记-整理”`算法实现的收集器，从局部（两个Region之间）上来看是基于**“复制”**算法实现的。这意味着G1运行期间不会产生内存空间碎片，收集后能提供规整的可用内存。此特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC。
- **可预测的停顿** 这是G1相对CMS的一大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了降低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在GC上的时间不得超过N毫秒，这几乎已经是实时Java（RTSJ）的垃圾收集器的特征了。

#### 理解GC日志

GC(Garbage Collection)是Java虚拟机中一个很重要的组成部分，在很多情况下我们都需要查看它的日志，下面内容就是介绍如何查看GC日志。

可选参数

```java
-XX:+PrintGC 输出GC日志
-XX:+PrintGCDetails 输出GC的详细日志
-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-Xloggc:../logs/gc.log 日志文件的输出路径
```

**测试类**

```java
/**
 * @author imlgw.top
 * @date 2019/8/15 17:16
 */
public class GCTest {
    
    /**
     * vm参数 ：-XX:+PrintGCDetails 输出GC的详细日志
     */
    private byte[] data =new byte[1024*1024*10]; //10M

    public static void main(String[] args) {
        GCTest gcTest = new GCTest();
        gcTest=null; //for gc
        System.gc();
        System.out.println("GC Test");
    }
}
```

**控制台打印结果**

```java
[GC (System.gc()) [PSYoungGen: 12902K->712K(38400K)] 12902K->720K(125952K), 0.3622244 secs] [Times: user=0.00 sys=0.00, real=0.36 secs] 
[Full GC (System.gc()) [PSYoungGen: 712K->0K(38400K)] [ParOldGen: 8K->638K(87552K)] 720K->638K(125952K), [Metaspace: 3438K->3438K(1056768K)], 0.0103044 secs] [Times: user=0.00 sys=0.00, real=0.02 secs] 
GC Test
Heap
 PSYoungGen      total 38400K, used 998K [0x00000000d5e00000, 0x00000000d8880000, 0x0000000100000000)
  eden space 33280K, 3% used [0x00000000d5e00000,0x00000000d5ef9b20,0x00000000d7e80000)
  from space 5120K, 0% used [0x00000000d7e80000,0x00000000d7e80000,0x00000000d8380000)
  to   space 5120K, 0% used [0x00000000d8380000,0x00000000d8380000,0x00000000d8880000)
 ParOldGen       total 87552K, used 638K [0x0000000081a00000, 0x0000000086f80000, 0x00000000d5e00000)
  object space 87552K, 0% used [0x0000000081a00000,0x0000000081a9fb68,0x0000000086f80000)
 Metaspace       used 3446K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 376K, capacity 388K, committed 512K, reserved 1048576K
  
Process finished with exit code 0
```

`PSYoungGen` 看见这个其实就猜的到了，Parallel Scavenge收集器收集的新生代，代表GC的区域和收集器。下面的`ParOldGen`对应的就是Parallel Old的老年代 ，PSYoungGen后面紧跟的 `12902K->712K(38400K)` 则代表 `GC前该区域已经使用的容量 -> GC后该区域已使用的容量（该区域总容量）`  ，而方括号外面的部分则代表 `GC前Java堆已使用量 -> GC后Java堆已使用量（Java堆中容量）` 。

### 内存分配与回收策略

- **Minor GC**：回收新生代，因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度一般也会比较快。
- **Full GC**：回收老年代和新生代，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多，出现Full GC通常会伴随着Minor GC，但是并不一定有时候也会直接进行Full GC。

对象的内存分配，往大的方向讲就是在堆上分配内存（也有可能被JIT拆散并间接的在栈上分配），对象主要分配在新生代的`Eden` 区上，如果启动了`本地线程分配缓冲`（TLAB） 会优先在TLAB上分配，少数情况也会直接分配在老年代中，分配的规则不是确定的，取决于垃圾收集器的组合以及虚拟机相关的参数的设置。

> -Xms  初始Heap大小
>
> -Xmx  java heap最大值 
>
> -Xmn  Young generation的heap大小

#### 对象优先分配在Eden上

大多数情况，对象会直接在新生代Eden上分配。当Eden区没有足够的区域进行分配的时候，虚拟机将会发起一次`Minor GC` 。

下面我们通过代码来感受一下这个过程

```java
/**
 * @author imlgw.top
 * @date 2019/8/15 17:16
 */
public class GCTest {
    private static int _1M= 1024*1024; //1m

    /*
        -XX:+PrintGCDetails
        -verbose:gc
        -Xms20M 限制Java堆大小20M不可扩展
        -Xmx20M
        -Xmn10M 10M分配给新生代
        -XX:SurvivorRatio=8
    */
    public static void main(String[] args) throws InterruptedException {
        byte[] alloc1,alloc2,alloc3,alloc4;
        alloc1=new byte[2*_1M];
        alloc2=new byte[2*_1M];
        alloc3=new byte[2*_1M];
        System.out.println("GC Test"); //发生GC
        alloc4=new byte[3*_1M];
    }
}

```

**控制台打印**

```java
GC Test
[GC (Allocation Failure) [PSYoungGen: 8002K->696K(9216K)] 8002K->6840K(19456K), 0.0028974 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 696K->0K(9216K)] [ParOldGen: 6144K->6783K(10240K)] 6840K->6783K(19456K), [Metaspace: 3439K->3439K(1056768K)], 0.0042088 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 9216K, used 3154K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 38% used [0x00000000ff600000,0x00000000ff914930,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 10240K, used 6783K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 66% used [0x00000000fec00000,0x00000000ff29fc88,0x00000000ff600000)
 Metaspace       used 3446K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 376K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0

```

在`alloc4`的时候 发现空间已经不足以装下了，所以进行了一次Minor GC，随后GC掉了YoungGen大概6M的空间，但是这些对象大于`from` 和`to` 两块`Survivor` 区域无法直接存入所以通过分配担保进入老年代中，可以看到后面老年代66%的占用存放的就是这几个对象。GC结束后，`alloc4` 也顺利的进入的`Eden` 

#### 大对象直接进入老年代

所谓的大对象其实就是需要大量连续内存空间的Java对象，最典型的就是那种很长的字符串以及数组，比如我上面代码中的byte数组，经常出现大对象容易导致内存还有不少空间就提前触发垃圾收集以获取足够的连续空间来“安置”这些大对象，所以应该避免产生“朝生夕死”的短命大对象。

虚拟机提供了一个  `-XX: PretenureSizeThreshold`参数，令大于这个设置值的对象直接在老年代分配（只对Serial和ParNew两款收集器有效，Parallel一般不用设置）。

```java
public class GCTest {
    private static int _1M= 1024*1024; //1m

    /*
        -XX:+PrintGCDetails
        -verbose:gc
        -Xms20M 限制Java堆大小20M不可扩展
        -Xmx20M
        -Xmn10M 10M分配给新生代
        -XX:SurvivorRatio=8
    */
    public static void main(String[] args) throws InterruptedException {
        byte[] alloc1,alloc2,alloc3,alloc4;
        alloc1=new byte[9*_1M];
        System.out.println("GC Test");
    }
}
```

**控制台打印**

```java
GC Test
Heap
 PSYoungGen      total 9216K, used 2022K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 24% used [0x00000000ff600000,0x00000000ff7f9990,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
  to   space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
 ParOldGen       total 10240K, used 9216K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 90% used [0x00000000fec00000,0x00000000ff500010,0x00000000ff600000)
 Metaspace       used 3447K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 376K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0

```

可以看到这个对象几乎全部被放到了老年代里面，老年代占用率 90%

#### 长期存活的对象将进入老年代

虚拟机为对象定义年龄计数器 Age，对象在 Eden 出生并经过 Minor GC 依然存活并且能被Survivor容纳的话，将移动到 Survivor 中，年龄就增加 1 岁，对象在Survivor区域每熬过一次 Minor GC年龄就增加一岁，当他的年龄增加到一定程度（默认为15岁），就会将它移动到老年代中（很形象有没有 😁）
`-XX:MaxTenuringThreshold` 用来定义年龄的阈值。

#### 动态年龄判定

虚拟机并不是永远要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 `Survivor` 中`相同年龄`所有对象大小的总和大于 `Survivor 空间的一半`，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到MaxTenuringThreshold 中要求的年龄

#### 空间分配担保

在发生 Minor GC 之前，虚拟机先检查老年代最大可用的`连续空间`是否大于新生代所有对象总空间，如果条件成立的话，那么 Minor GC 可以确认是安全的。
如果不成立的话虚拟机会查看 HandlePromotionFailure 的值是否允许担保失败，如果允许那么就会继续检查老年代
最大可用的连续空间是否`大于`历次晋升到老年代对象的`平均大小`，如果大于，将尝试着进行一次 Minor GC；如果小于，或者 HandlePromotionFailure 的值不允许冒险，那么就要进行一次 Full GC，如果出现了担保失败，那就只好发起一次`Full GC`  ，绕的圈子是最大的，但是一般情况下这个开关还是默认打开的，避免产生太多的Full GC，JDK1.6之后这个参数就被废掉了，`只要老年代的连续空间大于新生代总空间或者每次晋升的平均大小就进行Minor GC否则进行Full GC`

#### Full GC触发条件

对于 Minor GC，其触发条件非常简单，当 Eden 空间快满时，就将触发一次 Minor GC。而 Full GC 则相对复杂，有以下情况：

**_调用 System.gc()_**
只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。

**_老年代空间不足_**
老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。
为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数
调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对
象进入老年代的年龄，让对象在新生代多存活一段时间。

**_空间分配担保失败_**
使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

**_JDK 1.7 及以前的永久代空间不足_**
在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据
当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也
会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。
为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

**_Concurrent Mode Failure_**
执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。

## 参考

**周志明《深入理解Java虚拟机》**

[CYC大佬的博客🙏](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA?id=minor-gc-%e5%92%8c-full-gc)