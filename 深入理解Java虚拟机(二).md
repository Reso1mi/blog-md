---
title: 
  深入理解Java虚拟机（二）
tags: 
  [ClassLoader,JVM]
categories:
  [JVM]
date: 2019/8/17
cover: http://static.imlgw.top/blog/20190826/dYaTXirlDNhC.jpg?imageslim
---

> 这一篇主要讲JVM的类加载机制，本来很久之前就写了，但是这几天又重新学习了一遍，纠正了之前很多错误的观点，然后又补充了很多东西

## 类加载的过程

**前言：**

在Java语言中，类型的加载，连接和初始化过程都是在**运行期间**完成的，这与那些在**编译期间**需要进行链接工作的语言（C/C++）不同，这样毫无疑问会增加类加载的性能开销，但是会为Java提供高度的灵活性，Java天生可以动态扩展的就是依赖于运行时期**动态加载和动态链接**这个特点实现的，比如一个接口，完全可以在运行时期动态的指定其具体的实现类。又或者用户可以通过类加载器让一个本地的引用运行时从其他地方（网络等）加载一个二进制的流作为程序代码的一部分。

![mark](http://static.imlgw.top///20190418/em4YNNmXXmho.png?imageslim)

### ①加载

- 通过一个类的全限定名来获取定义此类的二进制字节流。
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- 在内存中生成一个代表这个类的 `java.lang.Class` 对象

#### 类加载的最终产物

> 类加载的最终产物是位于堆中的`Class`对象

《深入理解Java虚拟机》里面介绍的JVM内存结构(p39)

![JVM内存结构](http://static.imlgw.top///20190417/yk2bzxILsCpL.png?imageslim)



**堆（Heap）**：最大的一块区域，线程共享。所有的对象实例以及数组都要在堆上分配。回收器主要管理的对象。

> The Java Virtual Machine has a *heap* that is `shared` among `all` Java Virtual Machine threads. The heap is the run-time data area from which memory for all class instances and arrays is allocated.
>
> The heap is created on virtual machine `start-up`. Heap storage for objects is reclaimed by an automatic storage management system (known as a *garbage collector(GC)*); 
>
> 摘自  [官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.3)

**方法区（Method Area）**：又称为`非堆`，线程共享。存储类信息、常量、静态变量、即时编译器编译后的代码。

> The Java Virtual Machine has a *method area* that is `shared` among all Java Virtual Machine threads. The method area is analogous to the storage area for compiled code of a conventional language or analogous to the "text" segment in an operating system process. It stores per-class structures such as the **run-time constant pool**, **field** and **method data**, and the code for methods and constructors, including the special methods ([§2.9](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.9)) used in class and instance initialization and interface initialization.

**运行时常量池(run-time constant pool)**：

是方法区的一部分，用于存放编译期生成的各种字面量"123"，"LGW" 等字符串常量池，和符号引用。运行时常量池具有动态性，并非只有Class文件中的内容才能进入运行时常量池，运行期间也能将新的常量放入池中。如String.intern()方法。

**方法栈（JVM Stack）**：

线程私有。存储局部变量表、操作栈、动态链接、方法出口，`对象指针`。

**本地方法栈（Native Method Stack）**：

线程私有。为虚拟机使用到的Native 方法服务。如Java使用c或者c++编写的接口服务时，代码在此区运行。

**程序计数器（Program Counter Register）**：

线程私有，它可以看作是当前线程所执行的字节码的行号指示器。指向下一条要执行的指令。

#### 加载类的方式

- 本地磁盘 classpath
- 内存中加载 ，动态代理？RPC?
- 通过网络加载.class
- 从zip，jar中加载
- 数据库中提取.class
- 动态编译

### ②连接:  

#### 验证 

**文件格式验证**

- 是否以魔数`0xCAFEBABE`（咖啡宝贝）开头

- 主、次版本号是否在当前虚拟机处理范围之内

- 常量池中的常量中是否有不被支持的常量类型（检查常量tag标志）

- .....

**元数据验证**

第二阶段是对字节码描述的信息进行语义分析，以保证其描述的信息符合 Java 语言规范的要求

**字节码验证**

主要目的是通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的

由于数据流验证的高复杂性，虚拟机设计团队为了避免过多的时间消耗在字节码验证阶段，在 JDK 1.6 之后的 Javac 编译器和 Java 虚拟机中进行了一项优化，给方法体的 Code 属性的属性表中增加了一项名为" StackMapTable" 的属性, 只需要检查StackMapTable属性中的记录是否合法皆可以了

**符号引用验证**

符号引用的校验发生在虚拟机将`符号引用`转化为`直接引用`的时候，这个转化动作将在连接的第三阶段—-**解析**阶段中发生，通常需要校验以下内容：

- 符号引用中通过字符串描述的全限定名是否能找到对应的类
- 在指定类中是否存在符合方法的字段描述以及简单名称说描述的方法和字段。
- 符号引用中的类，字段，方法的访问性是否可以被当前类访问
- .......

如果所运行的全部代码（包括自己编写的及第三方包中的代码）都已经被反复使用和验证过，那么在实施阶段就可以考虑使用-Xverify:none 参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间

#### 准备

为类的**静态变量**分配内存，并初始化为默认值

| 数据类型  | 默认值   |
| --------- | -------- |
| boolean   | false    |
| char      | '/u0000' |
| byte      | (byte)0  |
| short     | (short)0 |
| int       | 0        |
| long      | 0L       |
| double    | 0.0d     |
| float     | 0.0f     |
| reference | null     |

🎯 当然并不是所有情况下都会初始化为零值，如果字段表的属性中有`ConstantValue` ，准备阶段就会直接初始化为这个这个`ConstantValue`属性的值

#### 解析

解析阶段是虚拟机将**常量池**内的**符号引用**替换为**直接引用**的过程。

- **符号引用：** 符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用的时候能无歧义的定位到目标中就行。
- **直接引用：** 直接引用可以是直接指向目标的指针，相对偏移量或者一个能间接定位到目标的句柄。

除 `invokedynamic`(动态语言支持) 指令以外，虚拟机实现可以对第一次解析的结果进行缓存。

解析动作主要针对类或接口、字段解析、类方法解析、接口方法解析、方法类型解析、方法句柄解析和调用点限定符 7 类符号引用进行。

关于这个也可以看看R大的回答 [JVM符号引用转换直接引用的过程 ](https://www.zhihu.com/question/30300585/answer/51335493)( 吹爆我R大 😁

### ③初始化：

初始化阶段就是执行`类构造器<clinit>`方法的过程

#### 类构造器&lt;client&gt;

💡`<clinit>`方法是由编译器自动收集类中的所有**类变量的赋值**动作和**静态语句块**（static块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中**出现的顺序决定的**，静态语句块只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块**可以赋值**，但是**不能访问**（比如print）

💡 和**实例构造器**`<init>` 不同，不需要显式的去调用父类的构造器，虚拟机会保证子类的`<clinit>`方法执行之前，父类的`<clinit>`方法已经执行完毕，因此虚拟机中第一个被执行的`<clinit>` 一定是Object类

💡` <clinit>` 方法对于类或者接口来说并不是必须的，如果类中没有静态语句块也没有静态变量的**赋值**操作，那么编译器可以不为这个类生成`<clinit>` 方法

💡 对于接口，不能使用static块，但是可以有静态变量的赋值操作。子类接口的`<clinit>`方法调用并不保证父接口的`<clinit>`方法被先调用，只有用到父接口的静态变量的时候，父接口`<clinit>`方法才会被调用。接口的实现类在初始化时也一样不会执行接口的`<clinit>`方法。

💡 虚拟机会保证一个类的`<clinit>`方法在多线程环境中被正确地加锁、同步。如果一个线程的`<clinit>`方法调用时间过长，就可能造成多个线程阻塞。Demo如下

```java
public class ClinitThreadTest {
    public static void main(String[] args) {
        new Thread(()->new SimpleObj()).start();
        new Thread(()->new SimpleObj()).start();
        new Thread(()->new SimpleObj()).start();
    }

    static class SimpleObj{
        private static AtomicBoolean init=new AtomicBoolean(true);

        static {
            System.out.println(Thread.currentThread().getName()+" i will be init");
            while (init.get()){

            }
            System.out.println("i am finished");
       }
    }
}
```

#### 为类的静态变量赋予正确的初始值

下面这个Demo很经典，可以帮助我们理解类加载的过程

```java
public class SingleTon {
    private static SingleTon ins=new SingleTon();

    public static  int x=0;

    public static int y;

    private SingleTon(){
        x++;y++;
    }

    public static SingleTon getIns(){
        return ins;
    }

    public static void main(String[] args) {
        SingleTon singleTon=getIns();
        System.out.println(singleTon.x);
        System.out.println(singleTon.y);
    }
}
```

如果不熟悉类加载的过程可能一眼就觉得应该是（1，1），其实不然，结合上面的加载过程分析

🔺 首先加载连接，然后执行准备工作，在执行完**准备阶段**工作后三个静态变量都有了**默认的初始值**，然后进入初始化阶段

🔺 `<clinit>`初始化阶段会按顺序**从上往下**依次赋予正确的初始值，所以**先执行**了`new Singleton()`给`ins`赋初始值，会调用它的构造器，然后x，y都++变为1，**再然后**就会给x，y赋予正确的初始值，x初始值为0，而y没有初始值所以就是（0，1）

### Class对象在哪里？

先说结论，Class对象和其他普通的Java对象一样都是存放在堆中的。

存放在方法区的是类的元数据(InstanceKlass，包括类的常量池( constant pool)  ，域(Field)信息  ，方法(Method)信息 ，除了常量外的所有静态(static)变量 等)，`java.lang.Class实例`并不负责记录真正的类元数据，而只是对VM内部的`InstanceKlass`对象的一个包装供Java的反射访问用，在《深入理解Java虚拟机》一书里面说的存放在方法区中应该是有问题的。

**类(静态)变量存放在哪里**

从JDK 1.3到JDK 6的HotSpot VM，静态变量保存在类的元数据（InstanceKlass）的末尾(永久代)。而从JDK 7开始的HotSpot VM，静态变量则是保存在类的Java镜像（java.lang.Class实例）的末尾，也就是堆中

参考 R大 [知乎回答](https://www.zhihu.com/question/50258991) 

## Java程序对类的使用方式

所有的java虚拟机实现必须在每个类或接口被java程序**首次主动使用**时才**初始化**它们

### 主动使用

- 创建类的实例（new）
- 对某个类或接口的静态变量进行读写（getstatic，putstatic）
- 调用类的静态方法（invokestatic）
- 反射某个类 （Class.forName()也可以设置不初始化类）
- 初始化子类时会先初始化父类
- 启动类 java HelloWorld  包含main函数的类
- Jdk1.7的动态语言支持

### 被动使用

除了上面 7 个之外，其它的都是被动使用，**不会初始化类**，下面的Demo有几个很容易出错的的例子

```java
package classloader_study.misc;
public class ClassActiveUse {
    static {
        System.out.println("main is init");
    }
    public static void main(String[] args) throws ClassNotFoundException {
        //System.out.println(Obj.t);
        //Obj.getObj();
        //Class.forName("classloader_study.Obj");
        //System.out.println(ObjChild.age);
        
        //父类会被初始化 通过子类调用父类的静态变量，子类不会初始化但是会被加载 
        //1. System.out.println(ObjChild.n);
        //不会初始化 定义应用数组也不会初始化类，但是会加载类
        //2. Obj [] arrays=new Obj[10]; 
        //不会初始化 常量会在编译期间放到常量池中不会初始化类也不会加载，子类加载也一样
        //3. System.out.println(Obj.t); 
        //Obj会被初始化   final修饰的复杂化类型再编译期间无法计算得到，会初始化类
        //4. System.out.println(Obj.x); 
        // 类加载器去加载
        //5. ClassLoader.loadClass();
    }
}

class Obj{

    public static  final int t=10;  //编译期间就已经确定了就是10
	
    public static int n=111;
    
    public static final int x=new Random().nextInt(100); //值不是常量，运行期间才会确定
    static { 
        System.out.println("Obj is init");
    }

    public static void getObj(){
        System.out.println("NULL");
    }
}

class ObjChild extends Obj{
    public static int age=12;

    static {
        System.out.println("Child is init");
    }
}
interface I{
    final static int a=0;
}
```

⚡new 一个Obj数组的时候，**会加载**Obj类，**不会初始化**Obj对象，但是会导致另一个类的初始化: `Lclassloader_study.misc.Obj`  这个类就代表了一个元素类型为Obj的一维数组，数组中应有的属性length和方法clone()都是在这个类实现的，这个类是由JVM在运行期间动态的为我们生成的，这个动作由`anewarray` 指令触发，而基本类型的数组由 `newarray` 指令触发。

⚡ final修饰的常量会在**编译期间**就放到调用这个变量的方法的类的常量池中，既不会加载也不会初始化，这一点可以通过反编译`ClassActiveUse` 看的到，两个类不存在任何关系了，甚至可以在编译完成后`将Obj的class文件删掉`仍然可以执行，但是后面的另一个final常量很明显在**编译期间无法确定值**，只有在运行期间才能回去到值，所以会加载并初始化类

⚡ 对接口的初始化和对类的初始化有一点不同，接口也有初始化过程，接口和类真正有区别是在上面主动使用的第5点，在**子接口被加载的时候并不要求其父接口全部完成了初始化**，只有在真正使用到父接口的时候才会初始化，这一点可以参考前文的 [类构造器部分](#③初始化：)

> `-XX:+TraceClassLoading` 可以用来追踪类的加载信息并且打印出来

###  对象的访问定位

**如果直接使用句柄访问**，java堆中将会划分出一块内存来作为句柄池，reference中存储的是对象的句柄地址，而句柄中包含了对象数据与类型数据各自的具体地址信息，如下图所示。

![mark](http://static.imlgw.top///20190418/FRA9GB31iiXH.png?imageslim)

**如果使用直接指针访问**，那么java堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而reference中存储的直接就是对象地址，如下图所示，每个对象都有一个自己Class对象的引用(`getClass`)

![mark](http://static.imlgw.top///20190418/1OtoI6wkivdz.png?imageslim)

这两种对象访问方式各有优势，使用句柄来访问的最大好处是reference中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而reference本身不需要修改。

使用直接指针访问方式的最大好处就是速度更快，它节省了一次指针定位的时间开销。HotSpot虚拟机使用的是直接指针访问的方式。句柄来访问的情况也十分常见。

## JVM类加载器

### 概述

> 虚拟机设计团队把加载阶段中的 `“通过一个类的全限定名来获取描述此类的二进制字节流”` 这个动作被放到Java虚拟机外部去实现，以便于让应用程序自己决定如何去获取所需要的类。实现这个动作的代码模块称为 `“类加载器”`

 类加载器并不需要等到某个类被 "首次使用" 时才加载它，这一点从前面讲解的 [被动使用的例子](#被动使用) 哪里就看得出来

JVM规范允许类加载器在预料某个类将要被使用的时候就预先加载它，如果在预先加载的过程中遇到了class文件缺失或者存在错误，类加载器必须在程序**首次主动使用**该类时才报告错误（LinkageError错误）

 如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误

![JVM类加载器](http://static.imlgw.top///20190419/pKeI6Bx6lbcX.png?imageslim)



**根（Bootstrap）类加载器**

该加载器没有父加载器。它负责加载虚拟机的核心类库，如java.lang.*等。根类加载器从系统属性sun.boot.class.path所指定的目录中加载类库。根类加载器的实现依赖于底层操作系统，属于虚拟机的实现的一部分，它并没有继承 java.lang.ClassLoader类(c/c++实现的)。

```java
System.out.println(System.getProperty("sun.boot.class.path"));
D:\java\jre\lib\resources.jar;D:\java\jre\lib\rt.jar;D:\java\jre\lib\sunrsasign.jar;D:\java\jre\lib\jsse.jar;D:\java\jre\lib\jce.jar;D:\java\jre\lib\charsets.jar;D:\java\jre\lib\jfr.jar;D:\java\jre\classes
```

**扩展（Extension）类加载器**

它的父加载器为根类加载器。它从java.ext.dirs系统属性所指定的目录中加载类库，或者从JDK的安装目录的jre.lib.ext子目录（扩展目录）下加载类库，如果把用户创建的**JAR**文件放在这个目录下，也会自动由扩展类加载器加载。扩展类加载器是纯Java类，是 java.lang.ClassLoader类的子类。

```java
System.out.println(System.getProperty("java.ext.dirs")); //java.ext.dirs属性
D:\java\jre\lib\ext;C:\WINDOWS\Sun\Java\lib\ext
```

**系统（System）类加载器**

也称为应用类加载器，它的父加载器为扩展类加载器。它从环境变量classpath或者系统属性java.class.path所指定的目录中加载类，它是用户自定义的类加载器的`默认父加载器`。系统类加载器是纯Java类，是java.lang.ClassLoader类的子类。

```java
System.out.println(System.getProperty("java.class.path"));
D:\java\jre\lib\charsets.jar;D:\java\jre\lib\deploy.jar;
D:\java\jre\lib\ext\access-bridge-64.jar;
D:\java\jre\lib\ext\cldrdata.jar;
D:\java\jre\lib\ext\dnsns.jar;
D:\java\jre\lib\ext\jaccess.jar;
D:\java\jre\lib\ext\jfxrt.jar;
D:\java\jre\lib\ext\localedata.jar;
D:\java\jre\lib\ext\nashorn.jar;
D:\java\jre\lib\ext\sunec.jar;
D:\java\jre\lib\ext\sunjce_provider.jar;
D:\java\jre\lib\ext\sunmscapi.jar;
D:\java\jre\lib\ext\sunpkcs11.jar;
D:\java\jre\lib\ext\zipfs.jar;
D:\java\jre\lib\javaws.jar;
D:\java\jre\lib\jce.jar;
D:\java\jre\lib\jfr.jar;
D:\java\jre\lib\jfxswt.jar;
D:\java\jre\lib\jsse.jar;
D:\java\jre\lib\management-agent.jar;
D:\java\jre\lib\plugin.jar;
D:\java\jre\lib\resources.jar;
D:\java\jre\lib\rt.jar;
C:\WorkSpace\concurrent_package\out\production\concurrent_package;
C:\JetBrains\IntelliJ IDEA 2018.1.4\lib\idea_rt.jar
```

> 其实所谓的父子加载器并不是继承的父子关系，而是包含的关系，子加载器中包含一个父加载器的引用

### 自定义类加载器

**先看下JDK的ClassLoader(1.8)源码**

```java
 protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        //没有父加载器就交给根加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    //需要子类去实现
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

这是ClassLoader类加载类的方法，可以看到中间有一段 

```java
if(parent!=null){  c = parent.loadClass(name, false);}
```

这就是**双亲委派机制**的实现，委托父类，让父类去加载，父类(根加载器)没有就会再层层下降如果有一个加载成功就会成功返回，除此之外还调用了一个没有实现的`findClass`

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}
```

也就是说想自定义类加载器的话就得重写`findClass`方法，实际上这个`findClass`才是类加载的核心，真正加载Class文件转换为CLass实例的就是`findClass`方法， `loadClass()`只是实现加载的逻辑，比如**双亲委派机制**

**实现一个简易的ClassLoader**

```java
public class MyClassLoader extends ClassLoader {
    //将字节码放到这个路径下
    private static final String DEFAULT_DIR = "D:\\ClassLoaderTest";

    private String dir = DEFAULT_DIR;

    private String classLoaderName;

    public MyClassLoader() {
        
    }

    public MyClassLoader(String classLoaderName) {
        this.classLoaderName = classLoaderName;
    }

    public MyClassLoader(ClassLoader parent, String classLoaderName) {
        super(parent);
        this.classLoaderName = classLoaderName;
    }

    public String getDir() {
        return dir;
    }

    public void setDir(String dir) {
        this.dir = dir;
    }

    public String getClassLoaderName() {
        return classLoaderName;
    }

    /**
     * xx.xx.xx.xx.xx.AAA
     * xx/xx/xx/xx/xx/.AAA.class
     *
     * @param name
     * @return
     * @throws ClassNotFoundException
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String classPath = name.replace(".", "/");
        File classFile = new File(dir, classPath + ".class");
        if (!classFile.exists()) {
            throw new ClassNotFoundException("The class " + name + " not found under " + dir);
        }
		//字节码文件的字节流
        byte[] classBytes = loadClassBytes(classFile);
        if (null == classBytes || classBytes.length == 0)
            throw new ClassNotFoundException("load the class " + name + " failed");
		//defineClass方法可以把二进制流字节组成的文件转换为一个java.lang.Class
        return this.defineClass(name, classBytes, 0, classBytes.length);
    }
	
    //将文件流转换为字节流
    private byte[] loadClassBytes(File classFile) {
        try (ByteArrayOutputStream baos = new ByteArrayOutputStream();
             FileInputStream fis = new FileInputStream(classFile)) {
            byte[] buffer = new byte[1024];
            int len;
            while ((len = fis.read(buffer)) != -1) {
                baos.write(buffer, 0, len);
            }
            baos.flush();
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

> defineClass方法可以把二进制流字节组成的文件转换为一个java.lang.Class，前提是Class文件是合法的。

```java
//自定义的需要加载的类
public class MyObject {
    static {
        System.out.println("MyObject static is init");
    }

    public void Hello(){
        System.out.println("Hello World");
    }
}
```

**测试自定义的ClassLoader**

```java
public class ClassLoaderTest {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        MyClassLoader loader=new MyClassLoader("Resolmi");
        //Class<?> aClass = loader.loadClass("classloader_study.myClassLoader.MyObject");
        Class<?> aClass = Class.forName("classloader_study.myClassLoader.MyObject", true, loader);
        System.out.println(aClass);
        System.out.println(aClass.getClassLoader());

        Object obj=aClass.newInstance();
        Method hello = aClass.getMethod("Hello", null);
        hello.invoke(obj, new Object[]{});
    }
}
```

这里直接`loadClass`或者`Class.forName()`都可以，通常我们的forName()都是默认用的**AppClassLoader**也就是系统加载器，但是我们也可以把我们的自定义加载器传递进去。

> tip: loadClass 不会初始化类，不属于上面提到的6种主动使用的方式，属于被动使用，Class.forName 第二个参数就是控制是否初始化

```java
MyObject static is init
class classloader_study.myClassLoader.MyObject
classloader_study.myClassLoader.MyClassLoader@74a14482
Hello World
```

如果使用了`ide`的话，这里很有可能编译器帮你自动编译了，也就是在你的classpath里面已经有字节码文件了，所以就直交给AppClassLoader加载了，所以需要将classpath里面的删掉，将字节码拷贝到你自定义的classLoader指定的目录里面。

### 双亲委派模式

即在类加载的时候，系统会判断当前类是否已经被加载，如果被加载，就会直接返回可用的类，否则就会尝试加载，在尝试加载时，会先请求双亲处理，如果双亲请求失败，则会自己加载

这里光看几个内置的ClassLoader可能还不太清晰这里用我们自定义的Loader来试试

```java
public class MyClassLoaderTest2 {
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InstantiationException, ClassNotFoundException {
        //loader1的加载目录是D:\\ClassLoaderTest（默认的）
        MyClassLoader loader1=new MyClassLoader("Resolmi-father");
        //设置loader1为loader2的父加载器
        MyClassLoader loader2=new MyClassLoader(loader1,"Resolmi2-son");
        //这个类存放在 D:\\ClassLoaderTest\\classloader_study\\myClassLoader 下面
        Class<?> aClass = loader2.loadClass("classloader_study.myClassLoader.MyObject");
        //设置Loader2的加载目录，这是个空目录，下面什么都没有
        loader2.setDir("D:\\ClassLoaderTest2");
        System.out.println(aClass);
        System.out.println(aClass.getClassLoader());
        System.out.println(((MyClassLoader)aClass.getClassLoader()).getClassLoaderName());
     }
}
```

控制台打印如下

```java
class classloader_study.myClassLoader.MyObject
classloader_study.myClassLoader.MyClassLoader@74a14482
Resolmi-father
```

我们用`Loader2`去加载类，但是这个类的加载目录是空的，然后我们指定`Loader2` 的父加载器为`Loader1`而`loader1` 得加载路径就是要加载得类的路径，可以看到这个这个类最终还是被加载出来了，而且是被 `loader1`加载出来的，也就是`loader2`把加载任务委托给了父加载器`loader1`,然后层层委托再回到`loader1`，由它加载。

到这里可能会有疑问，为什么要用双亲委派模式？这样走一圈多慢啊，其实这样做主要有两个方面的原因

💡 提高系统安全性，使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。对于一些系统核心类，用户自定义的不起作用了，因为都会交给`BootStrapLoader`去加载，比如自定义了一个java.lang.String的类，然后在加载的时候经过层层委派最后会交给 `BootStrapLoader`去加载然后返回，所以你自定义的String根本没有加载的机会，这样就避免了用户篡改Java核心的类

💡 避免重复加载，父Loader已经加载过的类，子Loader就不用再加载了，比如`Object类`这个类在`rt.jar`下，所以无论是在哪种环境下，最终都会交给`BootStrapClassLoader`去加载这个类，加载得到的都是同一个`Object类`，如果不采用双亲委派机制，让各个Loader自己加载那么可能加载出来的就会有很多个Object类（不是同一个Object类，下面会说到）

```java
Exception in thread "main" java.lang.SecurityException: Prohibited package name: java.lang
```

### 加密解密类加载器

本质上和上面的没什么区别，就是多了解密的功能，这里首先用加密工具类加密class

```java
public final class EncryptUtil {
	//相当于密钥
    public static final byte ENCRYPT_FACTOR = (byte) 0xff;

    private EncryptUtil() {

    }

    public static void encrypt(String source, String target) throws FileNotFoundException {
        try (FileInputStream in= new FileInputStream(source); FileOutputStream out = new FileOutputStream(target)) {
            int data;
            while ((data=in.read())!=-1){
                out.write(data^ENCRYPT_FACTOR);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    public static void main(String[] args) throws FileNotFoundException {
        encrypt("D:\\ClassLoaderTest\\classloader_study\\myClassLoader\\MyObject.class","D:\\ClassLoaderTest\\classloader_study\\myClassLoader\\MyObject2.class");
    }
}
```

然后用加密解密类加载器加载这个类

```java
public class DecryptClassLoader extends ClassLoader {

    private final String DEFAULT_DIR = "D:\\ClassLoaderTest";

    private String dir = DEFAULT_DIR;

    public DecryptClassLoader() {
        super();
    }

    public DecryptClassLoader(ClassLoader parent) {
        super(parent);
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String classPath = name.replace(".", "/");
        File classFile = new File(dir, classPath + ".class");
        if (!classFile.exists()) {
            throw new ClassNotFoundException("没找到对应的类文件 ：" + dir);
        }
        byte[] classBytes = loadClassByte(classFile);
        if (null == classBytes || classBytes.length == 0) {
            throw new ClassNotFoundException("加载失败");
        }
        return this.defineClass(name, classBytes, 0, classBytes.length);
    }

    private byte[] loadClassByte(File classFile) {
        try (FileInputStream in = new FileInputStream(classFile); ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            int data;
            while ((data=in.read())!=-1){
                //主要就是这里发生了变化，异或了0xff
                baos.write(data^0xff);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    public void setDir(String dir) {
        this.dir = dir;
    }
}
```

当然结果是加载成功啦😋，这里如果用其他的类加载器加载，或者把0xff那里去掉，就会报如下错误

```java
Exception in thread "main" java.lang.ClassFormatError: Incompatible magic value 889275713 in class file classloader_study/myClassLoader/MyObject
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
	at classloader_study.myClassLoader.MyClassLoader.findClass(MyClassLoader.java:64)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at classloader_study.encryption_study.ClassLoaderTest.main(ClassLoaderTest.java:15)
```

这个异常是链接阶段验证的错误，是上面提到的`defineClass()`抛出来的，因为你加了密，JVM在加载这个二进制流的时候就无法识别了自然就无法加载。

### 打破双亲委派机制

**覆盖loadClass()**

要打破双亲委派机制主要就是要覆盖`loadClass()`方法，自己定义加载类的方式。

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException {
    Class<?> clazz = null;

    //让父加载器加载java核心的包，因为有些类是继承的Java内部的核心类比如Object类
    if (name.startsWith("java.")) {
        try {
            ClassLoader system = ClassLoader.getSystemClassLoader();
            //这里仍然是委托上级
            clazz = system.loadClass(name);
            if (clazz != null) {
                if (resolve)
                    resolveClass(clazz);
                return clazz;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    try {
        //自己先找
        clazz = findClass(name);
    } catch (Exception e) {
        e.printStackTrace();
    }
	//找不到就交给父加载器
    if (clazz == null && getParent() != null) {
        getParent().loadClass(name);
    }

    return clazz;
}
```

其实就是自己先找，找不到才会交给父加载器，然后一个需要注意的就是你想加载的这个类可能继承了Java内部核心的类像`Object`类，然后要加载子类就要先加载它的父类，而你的这个包里面肯定是加载不到这些Java内部的核心类的，所以这些还是得交给上层的加载器去加载。

**测试效果**

```java
public class SimpleClassLoaderTest {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        SimpleClassLoader simpleClassLoader = new SimpleClassLoader();
        Class<?> aClass = simpleClassLoader.loadClass("classloader_study.break_parent.SimpleObject");
        System.out.println(aClass.getClassLoader());
    }
}
```

注意这里在`classpath`里面是有这个类的字节码文件的，按照双亲委托机制应该由AppClassLoader去加载

![mark](http://static.imlgw.top///20190420/5Nc0S6OoIa2D.png?imageslim)

但是仍然是由我们的SimpleClassLoader加载的，说明我们成功了打破了双亲委派机制（貌似Tomcat也是这种加载机制，有时间看看Tomcat的类加载器）

**面试题**

❓ **能不能自己写个类比如`java.lang.String`去覆盖Java的`String`？如果不覆盖`loadClass()`方法使用双亲委托肯定是不行，但是既然上面已经打破了双亲委托那是不是就可以了呢？**

`Talk is cheap，show me the code` 试试就知道了

先准备一个String类

```java
package java.lang;
/**
 * @author imlgw.top
 * @date 2019/4/18 12:01
 */
public class String {
    static {
        System.out.println("i am init");
    }
    public int getVal(){
        return  250;
    }
}
```

编译好之后放到我们自定义的`ClassLoader`的目录下，然后将我们`loadClass()`方法加载核心包的地方注释掉（不然还是会交给父加载器去加载）然后为了表示是我们自定义的`ClassLoader`加载的我们把classpath里面的字节码文件也删掉。

```java
 public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        SimpleClassLoader simpleClassLoader = new SimpleClassLoader();
        Class<?> aClass = simpleClassLoader.loadClass("java.lang.String");
        System.out.println(aClass.getClassLoader());
    }
```

然后就会看到如下的`SecurityException`

```java
java.lang.SecurityException: Prohibited package name: java.lang
	at java.lang.ClassLoader.preDefineClass(ClassLoader.java:662)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:761)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
	at classloader_study.break_parent.SimpleClassLoader.findClass(SimpleClassLoader.java:52)
	at classloader_study.break_parent.SimpleClassLoader.loadClass(SimpleClassLoader.java:77)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at classloader_study.break_parent.SimpleClassLoaderTest.main(SimpleClassLoaderTest.java:7)
```

其实Java为了安全，自定义类取名不能取`java.*`等核心包名开头，在`preDefineClass`里面做了检查，所以即使你打破了双亲委托机制你依然不能去覆盖Java的核心类（肯定不行啊😄）。

其实这里后来了解了`Unsafe`后也尝试用`Unsafe` 去加载这个类

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {
        Unsafe unsafe=UnsafeTest.getUnsafe();
        String classPath = name.replace(".", "/");
        File classFile = new File(dir, classPath + ".class");
        if (!classFile.exists()) {
            throw new ClassNotFoundException("The class " + name + " found under " + dir);
        }

        byte[] classBytes = loadClassBytes(classFile);
        if (null == classBytes || classBytes.length == 0)
            throw new ClassNotFoundException("load the class " + name + " failed");

        return unsafe.defineClass(null,classBytes,0,classBytes.length,SimpleClassLoader.this,null);
}
```

`loadClass` 同上，尝试加载你会发现 会提示找不到`Object` 类，嗯？已经在加载父类了，难不成还真可以？这里其实已经和上面的方法不同了，上面的方法是不会进入到加载父类的环节的，直接在加载前就被检测了包名然后GG了，随后我在`loadClass` 中让系统加载器去加载Object类，再次尝试加载

```java
java.lang.SecurityException: Prohibited package name: java.lang
	at sun.misc.Unsafe.defineClass(Native Method)
	at classloader_study.break_parent.SimpleClassLoader.findClass(SimpleClassLoader.java:57)
	at classloader_study.break_parent.SimpleClassLoader.loadClass(SimpleClassLoader.java:83)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at classloader_study.break_parent.SimpleClassLoaderTest.main(SimpleClassLoaderTest.java:7)
Exception in thread "main" java.lang.NullPointerException
	at classloader_study.break_parent.SimpleClassLoaderTest.main(SimpleClassLoaderTest.java:8)
```

还是熟悉的味道😂 `defineClass`虚拟机本地方法抛了异常

### 类加载器细节深入

**定义加载器&初始加载器**

⚡真正完成类的加载工作的类加载器和启动这个加载过程的类加载器，有可能不是同一个。真正`完成类的加载`工作是通过调用 `defineClass(findClass)`来实现的；而`启动`类的加载过程是通过调用 `loadClass`来实现的。前者称为一个类的定义加载器（defining loader），后者称为初始加载器（initiating loader）。在 Java 虚拟机判断两个类是否相同的时候，使用的是类的定义加载器。也就是说，哪个类加载器启动类的加载过程并不重要，重要的是最终定义这个类的加载器。两种类加载器的关联之处在于：**一个类的定义加载器是它引用的其它类的初始加载器**。如类 `com.example.Outer`引用了类 `com.example.Inner`，则由类 `com.example.Outer`的**定义加载器**负责启动类 `com.example.Inner`的加载过程。

**命名空间&运行时包**

> 每个类都有自己的命名空间，命名空间由`该加载器及所有父加载器所加载的类`组成，`子加载器的命名空间包含所有父加载器的命名空间`，因此子加载器可以加载的类可以看就按父加载器加载的类，例如系统类加载器可以看见根加载器加载的类。由父加载器加载的类看不见子加载器加载的类，如果两个类之间没有直接或者间接的父子关系，那么他们各自加载的类相互不可见

⚡数组类的Class不是由类加载器加载的，是由JVM在运行期间动态生成的，但是通过`getClassLoader`返回的类加载器和数组的元素的类加载器是一样的，原生的类型比如`int` 之类的没有类加载器返回的是null

⚡每个类加载器都有其自己的命名空间，命名空间由该加载器和其所有父类加载器所加载的类组成，同一份字节码两个不同的类加载器加载出来的不是同一个类。

```java
public class MyClassLoaderTest2 {
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InstantiationException, ClassNotFoundException {
        //注意这里的ClassLoader要么利用前面的打破了双亲委托机制的，要么把classpath里面字节码的删掉。
        MyClassLoader loader1=new MyClassLoader("Resolmi1");
        MyClassLoader loader2=new MyClassLoader("Resolmi2");
        Class<?> aClass2 = loader2.loadClass("classloader_study.myClassLoader.MyObject");
        Class<?> aClass1 = loader1.loadClass("classloader_study.myClassLoader.MyObject");
        //两个不同的加载器（没有父子关系）加载同一个类加载出来的不是同一个
        System.out.println(aClass1.hashCode()); //2133927002
        System.out.println(aClass2.hashCode()); //1836019240
     }
}
```

⚡**父类加载器**无法访问**子类加载器**加载的类，而**子加载器**可以访问到**父加载器**所加载的类

```java
public class ClassLoaderTest {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        //使用上面自定义的ClassLoader
        MyClassLoader loader=new MyClassLoader("Resolmi");
        Class<?> aClass = loader.loadClass("classloader_study.myClassLoader.Parent");
        //创建实例
        aClass.newInstance();
    }
}
```

**Parent类**

```java
public class Parent {
    public Parent() {
        System.out.println("Parent is load by" + this.getClass().getClassLoader());
        Hello();
    }

    public void Hello() {
        //父加载器访问子加载器加载的类
        System.out.println("Parent can see the " + Sub.class);
    }
}
```

**Sub类**

```java
public class Sub {
    public Sub(){
        System.out.println("Sub is load by"+this.getClass().getClassLoader());
        new Parent(); //构造Parent类
    }
}
```

做完了这些工作之后，编译代码，然后将classpath中的`Parent.class`拷贝到自定义的ClassLoader路径下面，然后删掉classpath中的`Parent.class` ，然后运行

```java
Sub is load byclassloader_study.myClassLoader.MyClassLoader@74a14482
Parent is load bysun.misc.Launcher$AppClassLoader@18b4aac2
Exception in thread "main" java.lang.NoClassDefFoundError: classloader_study/myClassLoader/Sub
	at classloader_study.myClassLoader.Parent.Hello(Parent.java:15)
	at classloader_study.myClassLoader.Parent.<init>(Parent.java:10)
	at classloader_study.myClassLoader.Sub.<init>(Sub.java:10)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at java.lang.Class.newInstance(Class.java:442)
	at classloader_study.myClassLoader.ClassLoaderTest.main(ClassLoaderTest.java:14)
Caused by: java.lang.ClassNotFoundException: classloader_study.myClassLoader.Sub
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 9 more
```

可以看到，我们的`Sub` 是由我们自定义的`MyClassLoader` 加载的，因为我们删掉了Classpath中的字节码，而在自定义的路径中有所以最后还是由我们的 `MyClassLoader` 加载了，所以我们的`Parent`类首先也会由我们的`自定义类加载器MyClassLoader`去作为**初始加载器**加载，由于我们的Classpath中还有字节码文件，所以在双亲委派机制下`Sub类` 最终由`AppClassLoader`加载 ，两个类由不同的类加载器加载，然后我们在`Parent` 类中试图访问`Sub` 类，结果抛出了`ClassNotFoundException`  和`NoClassDefFoundError` 异常

❓ **那我们反过来在`Sub`类中访问 `Parent` 类会发生什么，改造一下Parent和Sub**

**Parent类**

```java
public class Parent {
    public Parent() {
        System.out.println("Parent is load by" + this.getClass().getClassLoader());
        //Hello();
    }

    public void Hello() {
        //父加载器访问子加载器加载的类
        System.out.println("Parent can see the " + Sub.class);
    }
}
```

**Sub类**

```java
public class Sub {
    public Sub(){
        System.out.println("Sub is load by"+this.getClass().getClassLoader());
        new Parent(); //构造Sub类
        //访问父加载器加载的类
        System.out.println("Sub can see "+Parent.class);
    }
}
```

和上面一样删掉Classpath中Sub类的class文件没然后运行

```java
findclass is invoke MyClassLoader is loadclassloader_study.myClassLoader.Sub
Sub is load byclassloader_study.myClassLoader.MyClassLoader@74a14482
Parent is load bysun.misc.Launcher$AppClassLoader@18b4aac2
Sub can see class classloader_study.myClassLoader.Parent

Process finished with exit code 0
```

没有任何问题，由此就可以证明我们上面的结论是正确的。

❓ **面试题：如何让一个类的static代码块执行两次**

用不同的类加载器去加载这个类，至于为什么应该不用我多说了吧。

**类的卸载和ClassLoader的卸载**

>  由Java虚拟机自带的类加载器所加载的类，在虚拟机的生命周期中**始终不会被卸载**，Java虚拟机本身会引用这些类加载器，而这些类加载器则会始终引用他们所加载的类的Class对象，因此这些Class对象始终是可达的。

⚡JVM中的Class只有满足以下三个条件，才能被GC回收，也就是该Class被卸载（unload）

- 该类所有的实例都已经被GC。
- 加载该类的ClassLoader实例已经被GC。(Class对象里面有对ClassLoader的引用)
- 该类的java.lang.Class对象没有在任何地方被引用。

GC的时机我们是不可控的，那么同样的我们对于Class的卸载也是不可控的，使用`-XX:+TraceClassUnloading` 或者jvisualvm可以看到类的卸载

## 线程上下文加载器(**TCCL**)

​	在说TCCL之前不得不说一下另一个话题，SPI（Service Provider Interface，SPI）服务提供接口，由第三方为这些接口提供实现。常见的 SPI 有 JDBC、JCE、JNDI、JAXP 和 JBI 等。这些 SPI 的接口由 **Java 核心库**来提供，而这些 SPI 的实现代码则是作为 Java 应用所依赖的 jar 包被包含进**类路径**（classpath）里。SPI接口中的代码经常需要加载具体的实现类。

> 为什么要使用SPI?  SPI是JDK内置的一种**服务提供发现机制**。这样做的好处主要是为了解耦，实现动态替换，减少硬编码（比如jdk1.6之前的Class.forName("XXXX") ）面向接口编程，在很多开源框架中都有体现，比如Dubbo，Spring等

那么问题来了，`SPI的接口`是Java核心库的一部分位于`rt.jar`中，是由**根加载器**(Bootstrap Classloader)来加载的，而`SPI的实现类`是一般是第三方的提供的，位于`classpath`目录中，而**根加载器**很明显是无法直接加载到这个目录下的SPI 的实现类的 (双亲委派)，那`SPI`是如何自动加载到实现类的呢？

为了解决这个问题，虚拟机提供了**线程上下文加载器（TCCL）**配合`ServiceLoader`来帮助上层加载器加载类，`TCCL`破坏了“双亲委派模型”，可以在执行过程中切换为`TCCL` 来加载第三方的SPI实现类，抛弃双亲委派机制，使程序可以逆向使用类加载器。**TCCL**默认是系统类加载器，也可以通过`setContextClassLoader`去设置

### JDBC源码案例分析

翻到了最开始学JDBC的时候写的代码😄

![JDBC](http://static.imlgw.top///20190420/K4uQf6h5czDb.png?imageslim)

**贾琏欲执事**，可以看到第一步还是加载并且初始化驱动，前面已经提到这一步其实没有必要了，jdk1.6之后因为ServiceLoader，SPI机制的出现，就不用再显示的加载驱动，但是正如上面所说`Driver`只是个接口存放于`rt.jar` 中，由根加载器所加载，那SPI是怎么自动的加载到`mysql`的`Driver`实例的呢？😕

我们一步步的来看，首先`DriverManager.getConnection()`这里，`getConnection`是个静态方法，调用它就会先执行`DriverManager`类的静态代码块，而静态代码块里面主要执行的就是`loadInitialDrivers()`

### loadInitialDrivers()源码

```java
private static void loadInitialDrivers() {
    String drivers;
    try {
        drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
            public String run() {
                return System.getProperty("jdbc.drivers");
            }
        });
    } catch (Exception ex) {
        drivers = null;
    }
    // If the driver is packaged as a Service Provider, load it.
    // Get all the drivers through the classloader
    // exposed as a java.sql.Driver.class service.
    // ServiceLoader.load() replaces the sun.misc.Providers()
	// 如果驱动正确打包为jar就会用ServiceLoader去加载它
    AccessController.doPrivileged(new PrivilegedAction<Void>() {
        public Void run() {
            /****************************************************/
			/*ServiceLoad工具类，注意这个ServiceLoad的加载器，默认就是TCCL*/
            ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
            /****************************************************/
            /*在这里会获取到一个Drivers的迭代器，但是其实还没有开始加载类*/
            Iterator<Driver> driversIterator = loadedDrivers.iterator();

            /* Load these drivers, so that they can be instantiated.
             * It may be the case that the driver class may not be there
             * i.e. there may be a packaged driver with the service class
             * as implementation of java.sql.Driver but the actual class
             * may be missing. In that case a java.util.ServiceConfigurationError
             * will be thrown at runtime by the VM trying to locate
             * and load the service.
             *
             * Adding a try catch block to catch those runtime errors
             * if driver not available in classpath but it's
             * packaged as service and that service is there in classpath.
             */
            try{
                while(driversIterator.hasNext()) {
                    //迭代的过程中通过next反射加载并初始化这个驱动字节码
                    //没有接收返回的数据库驱动实例
                    driversIterator.next();
                }
            } catch(Throwable t) {
            // Do nothing
            }
            return null;
        }
    });

    println("DriverManager.initialize: jdbc.drivers = " + drivers);
	//加载Jdk中的驱动实例（虽然我并不知道是什么）总之我们第三方的驱动已经加载好了
    if (drivers == null || drivers.equals("")) {
        return;
    }
    String[] driversList = drivers.split(":");
    println("number of Drivers:" + driversList.length);
    for (String aDriver : driversList) {
        try {
            println("DriverManager.Initialize: loading " + aDriver);
            Class.forName(aDriver, true,
                    ClassLoader.getSystemClassLoader());
        } catch (Exception ex) {
            println("DriverManager.Initialize: load failed: " + ex);
        }
    }
}
```

可以看到中间有一行很关键的代码

 `ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);`

这就是我上面所提到的 `ServiceLoader` 

> 对于SPI机制，JDK提供了`java.util.ServiceLoader`工具类，在使用某个服务接口时，它可以帮助我们查找该服务接口的实现类，加载和初始化，前提条件是基于它的约定：当服务的提供者提供了服务接口的一种实现之后，在`jar`包的`META-INF/services/`目录里同时创建一个以`服务接口命名的文件`。该文件里就是实现该服务接口的具体实现类（去解压看看那些jar包就可以看见这些信息o(*￣▽￣*)ブ）

### ServiceLoader类源码

为了节约篇幅删掉了一些注释，发现其实整个类也没多少行大概2，3百行的样子，需要注意这个类并不是线程安全的，所以使用的时候需要注意

```java
public final class ServiceLoader<S> implements Iterable<S> {
	//目录前缀就是从这里来的
    private static final String PREFIX = "META-INF/services/";

    //实现类Service
    private final Class<S> service;

    // The class loader used to locate, load, and instantiate providers
    private final ClassLoader loader;

    // The access control context taken when the ServiceLoader is created
    private final AccessControlContext acc;

    // Cached providers, in instantiation order
    // 按照实例的顺序，来缓存服务提供者避免重复的加载，具体可以看下面的iterator方法
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // The current lazy-lookup iterator
    private LazyIterator lookupIterator;

    public void reload() {
        providers.clear();
        //初始化懒加载迭代器
        lookupIterator = new LazyIterator(service, loader);
    }
	
    //构造器
    private ServiceLoader(Class<S> svc, ClassLoader cl) {
        service = Objects.requireNonNull(svc, "Service interface cannot be null");
        loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
        acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
        reload();
    }

    private static void fail(Class<?> service, String msg, Throwable cause)
        throws ServiceConfigurationError
    {
        throw new ServiceConfigurationError(service.getName() + ": " + msg,
                                            cause);
    }

    private static void fail(Class<?> service, String msg)
        throws ServiceConfigurationError
    {
        throw new ServiceConfigurationError(service.getName() + ": " + msg);
    }

    private static void fail(Class<?> service, URL u, int line, String msg)
        throws ServiceConfigurationError
    {
        fail(service, u + ":" + line + ": " + msg);
    }

    // Parse a single line from the given configuration file, adding the name
    // on the line to the names list.
    //
    private int parseLine(Class<?> service, URL u, BufferedReader r, int lc,
                          List<String> names)
        throws IOException, ServiceConfigurationError
    {
        String ln = r.readLine();
        if (ln == null) {
            return -1;
        }
        int ci = ln.indexOf('#');
        if (ci >= 0) ln = ln.substring(0, ci);
        ln = ln.trim();
        int n = ln.length();
        if (n != 0) {
            if ((ln.indexOf(' ') >= 0) || (ln.indexOf('\t') >= 0))
                fail(service, u, lc, "Illegal configuration-file syntax");
            int cp = ln.codePointAt(0);
            if (!Character.isJavaIdentifierStart(cp))
                fail(service, u, lc, "Illegal provider-class name: " + ln);
            for (int i = Character.charCount(cp); i < n; i += Character.charCount(cp)) {
                cp = ln.codePointAt(i);
                if (!Character.isJavaIdentifierPart(cp) && (cp != '.'))
                    fail(service, u, lc, "Illegal provider-class name: " + ln);
            }
            if (!providers.containsKey(ln) && !names.contains(ln))
                names.add(ln);
        }
        return lc + 1;
    }

    private Iterator<String> parse(Class<?> service, URL u)
        throws ServiceConfigurationError
    {
        InputStream in = null;
        BufferedReader r = null;
        ArrayList<String> names = new ArrayList<>();
        try {
            in = u.openStream();
            r = new BufferedReader(new InputStreamReader(in, "utf-8"));
            int lc = 1;
            while ((lc = parseLine(service, u, r, lc, names)) >= 0);
        } catch (IOException x) {
            fail(service, "Error reading configuration file", x);
        } finally {
            try {
                if (r != null) r.close();
                if (in != null) in.close();
            } catch (IOException y) {
                fail(service, "Error closing configuration file", y);
            }
        }
        return names.iterator();
    }

    // Private inner class implementing fully-lazy provider lookup
    // 看名字就知道了，懒迭代器，在迭代的时候才真正的加载
    private class LazyIterator
        implements Iterator<S>
    {

        Class<S> service;
        ClassLoader loader;
        Enumeration<URL> configs = null;
        Iterator<String> pending = null;
        String nextName = null;

        private LazyIterator(Class<S> service, ClassLoader loader) {
            this.service = service;
            this.loader = loader;
        }

        private boolean hasNextService() {
            if (nextName != null) {
                return true;
            }
            if (configs == null) {
                try {
                    String fullName = PREFIX + service.getName();
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
                    fail(service, "Error locating configuration files", x);
                }
            }
            while ((pending == null) || !pending.hasNext()) {
                if (!configs.hasMoreElements()) {
                    return false;
                }
                pending = parse(service, configs.nextElement());
            }
            nextName = pending.next();
            return true;
        }
        
		//迭代器的next
        private S nextService() {
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {
                //利用TCCL加载实现类，但是不初始化
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
                //newInstance()初始化了对应的实现类
                S p = service.cast(c.newInstance());
                //放到providers中
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }

        public boolean hasNext() {
            if (acc == null) {
                return hasNextService();
            } else {
                PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
                    public Boolean run() { return hasNextService(); }
                };
                return AccessController.doPrivileged(action, acc);
            }
        }

        public S next() {
            if (acc == null) {
                return nextService();
            } else {
                PrivilegedAction<S> action = new PrivilegedAction<S>() {
                    public S run() { return nextService(); }
                };
                return AccessController.doPrivileged(action, acc);
            }
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

    }

    /**
     * Lazily loads the available providers of this loader's service.
     * @return  An iterator that lazily loads providers for this loader's
     *          service
     */
    public Iterator<S> iterator() {
        return new Iterator<S>() {

            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
                if (knownProviders.hasNext())
                    return true;
                return lookupIterator.hasNext();
            }

            public S next() {
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                return lookupIterator.next();
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

        };
    }

    /* 重载的ServiceLoad */
    public static <S> ServiceLoader<S> load(Class<S> service,
                                            ClassLoader loader)
    {
        return new ServiceLoader<>(service, loader);
    }

	//DriverManage里面就是调用的这个方法
    public static <S> ServiceLoader<S> load(Class<S> service) {
        //拿到了TCCL
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        //调用上面重载的ServiceLoad方法
        return ServiceLoader.load(service, cl);
    }


    public static <S> ServiceLoader<S> loadInstalled(Class<S> service) {
        ClassLoader cl = ClassLoader.getSystemClassLoader();
        ClassLoader prev = null;
        while (cl != null) {
            prev = cl;
            cl = cl.getParent();
        }
        return ServiceLoader.load(service, prev);
    }

    public String toString() {
        return "java.util.ServiceLoader[" + service.getName() + "]";
    }

}
```

> 首先我们要明白一点，`Driver接口`，`DriverManage类`，以及`ServiceLoader`都是由**根加载器**去加载的(如果不相信的话可以用TraceClassLoading去查看)，所以在`ServiceLoader`中也是无法直接加载具体得实现类的

前面`loadInitialDriver()`调用的就是这里的`ServiceLoader.load(Class< S> service)` 方法，这个方法中悄悄的拿到了`TCCL` ，而TCCL在`Launcher` 类（系统加载器和扩展加载器都是在Launcher中实现的）中默认设置成了系统加载器，具体可以去看一下源码这里我就不展开了，然后调用另一个重载的构造方法将`TCCL` 传递进去，最终调用了 `reload()`方法

```java
public void reload() {
    providers.clear();
    //初始化懒加载迭代器
    lookupIterator = new LazyIterator(service, loader);
}
```

 在这个方法中首先清空服务提供者(providers)缓存，然后初始化了一个`LazyIterator` 看名字就知道是啥意思了，其实到这里仍然没有任何具体的加载动作，因为这里采用的是按需加载，也就是懒加载，在迭代的时候才会去加载类

```java
private class LazyIterator implements Iterator<S>{
	Class<S> service;
    ClassLoader loader;
    Enumeration<URL> configs = null;
    Iterator<String> pending = null;
    String nextName = null;

    private LazyIterator(Class<S> service, ClassLoader loader) {
        this.service = service;
        this.loader = loader;
    }

    private boolean hasNextService() {
        if (nextName != null) {
            return true;
        }
        if (configs == null) {
            try {
                //拿到接口全名
                String fullName = PREFIX + service.getName();
                if (loader == null)
                    configs = ClassLoader.getSystemResources(fullName);
                else
                    configs = loader.getResources(fullName);
            } catch (IOException x) {
                fail(service, "Error locating configuration files", x);
            }
        }
        while ((pending == null) || !pending.hasNext()) {
            if (!configs.hasMoreElements()) {
                return false;
            }
            //解析
            pending = parse(service, configs.nextElement());
        }
        nextName = pending.next();
        return true;
    }
    
	//迭代器的next
    private S nextService() {
        if (!hasNextService())
            throw new NoSuchElementException();
        String cn = nextName;
        nextName = null;
        Class<?> c = null;
        try {
            //具体加载类的地方就是在这里
            //利用前面传进来的TCCL加载实现类，但是不初始化类
            c = Class.forName(cn, false, loader);
        } catch (ClassNotFoundException x) {
            fail(service,
                 "Provider " + cn + " not found");
        }
        if (!service.isAssignableFrom(c)) {
            fail(service,
                 "Provider " + cn  + " not a subtype");
        }
        try {
            //newInstance()实例化对应的实现类
            S p = service.cast(c.newInstance());
            //put到providers中
            providers.put(cn, p);
            return p;
        } catch (Throwable x) {
            fail(service,
                 "Provider " + cn + " could not be instantiated",
                 x);
        }
        throw new Error();          // This cannot happen
    }

    public boolean hasNext() {
        if (acc == null) {
            return hasNextService();
        } else {
            PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
                public Boolean run() { return hasNextService(); }
            };
            return AccessController.doPrivileged(action, acc);
        }
    }

    public S next() {
        if (acc == null) {
            return nextService();
        } else {
            PrivilegedAction<S> action = new PrivilegedAction<S>() {
                public S run() { return nextService(); }
            };
            return AccessController.doPrivileged(action, acc);
        }
    }

    public void remove() {
        throw new UnsupportedOperationException();
    }

}
```
🔸 可以看到在迭代器的`nextService()` 中才开始利用的`Class.forName()` 加载的具体实现类，而这个加载器正器正是前面`reload`传递过来的 `TCCL` 也就是默认的系统类加载器

🔸 随后在紧跟的try语句中通过 `newInstance()` 实例化了具体的实现类(MySql的驱动) ，然后put进providers队列并且返回实例化的实现类，但是在`loadInitialDrivers`中并没有接收这个返回，那他这里实例化是什么用意呢？

我们回到`getConnection()` 方法

### getConnection()源码

前面的静态方法调用完毕驱动已经加载，下面就是获取数据库连接了.

```java
public static Connection getConnection(String url,
    String user, String password) throws SQLException {
    java.util.Properties info = new java.util.Properties();

    if (user != null) {
        info.put("user", user);
    }
    if (password != null) {
        info.put("password", password);
    }
	// Reflection.getCallerClass()调用者的Class对象
    return (getConnection(url, info, Reflection.getCallerClass()));
}
```

一般获取连接都是调用的上面这个方法，这个方法最终会调用另一个重载的方法，同时传入一个调用者的Class对象

```java
private static Connection getConnection(
    String url, java.util.Properties info, Class<?> caller) throws SQLException {
    /*
     * When callerCl is null, we should check the application's
     * (which is invoking this class indirectly)
     * classloader, so that the JDBC driver class outside rt.jar
     * can be loaded from here.
     */
    //Caller就是调用者的CLass也就是我们的应用代码类
    //获取到我们应用类的类加载器
    ClassLoader callerCL = caller != null ? caller.getClassLoader() : null;
    synchronized(DriverManager.class) {
        // synchronize loading of the correct classloader.
        if (callerCL == null) {
            //如果为空就，获取线程线下文加载器
            callerCL = Thread.currentThread().getContextClassLoader();
        }
    }

    if(url == null) {
        throw new SQLException("The url cannot be null", "08001");
    }

    println("DriverManager.getConnection(\"" + url + "\")");

    // Walk through the loaded registeredDrivers attempting to make a connection.
    // Remember the first exception that gets raised so we can reraise it.
    SQLException reason = null;
	
    //遍历这个registeredDrivers里面都是DriverInfo
    for(DriverInfo aDriver : registeredDrivers) {
        // If the caller does not have permission to load the driver then
        // skip it.
        //检查加载驱动的加载器是不是调用者的类加载器
        if(isDriverAllowed(aDriver.driver, callerCL)) {
            try {
                println("    trying " + aDriver.driver.getClass().getName());
                //获取连接
                Connection con = aDriver.driver.connect(url, info);
                if (con != null) {
                    // Success!
                    println("getConnection returning " + aDriver.driver.getClass().getName());
                    return (con);
                }
            } catch (SQLException ex) {
                if (reason == null) {
                    reason = ex;
                }
            }

        } else {
            println("    skipping: " + aDriver.getClass().getName());
        }

    }

    // if we got here nobody could connect.
    if (reason != null)    {
        println("getConnection failed: " + reason);
        throw reason;
    }

    println("getConnection: no suitable driver found for "+ url);
    throw new SQLException("No suitable driver found for "+ url, "08001");
}
```

🔸 可以看到中间有一个foreach循环，遍历`registeredDrivers`，这是个`CopyOnWriteArrayList` 这个类看名字就知道存放的是已经注册的`Drivers` 实现类，那这些实现类是什么时候注册进来的呢？回到我们之前抛出的一个问题，在`ServiceLoader`的迭代器中加载了具体的类之后进行了实例化，但是`DriverManager` 中并没有接收这个实例，我们来看一下具体的驱动实现类

```java
package com.mysql.cj.jdbc;

import java.sql.DriverManager;
import java.sql.SQLException;

public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }

    static {
        try {
            //注册到DriverManager中去
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

🔸 相信看到这里大家就明白了，前面初始化这个类的作用就是为了能触发这个**静态代码块**，进而执行registerDriver()方法进行注册，真是妙啊👏一环套一环

🔸 还有一个需要注意的地方就是这里注册的虽然是个空的Driver类，但是别忘了它还有个父类NonRegisteringDriver

这个类才是真正的实现类具体的`connect()` 等方法都是在这个里面实现的，而Driver继承了它的方法

🔸 在遍历`registeredDrivers` 的时候还调用了一个`isDriverAllowed(aDriver.driver, callerCL)` 方法这个方法第一个参数就是驱动的实现类，第二个参数就是前面获取到的**调用者的类加载器** ，作用就是通过利用传进来的加载器尝试加载这个类，然后判断是不是同一个类，（众所周知不同的加载器因为命名空间的存在，即使加载同一份字节码文件得到的也不是一个类） 如果是就允许加载，否则不允许，为啥要这样做呢？其实还是因为命名空间的问题，因为有了SPI的机制，你**加载初始化这个实现类的加载器**(TCCL)和最终去**调用实现类的方法的类的加载器**有可能不是同一个，因为程序员可以很容易的将TCCL修改成其他的类加载器，如果不保证一致的话后面就会出现`ClassCastException`等异常 

```java
private static boolean isDriverAllowed(Driver driver, ClassLoader classLoader) {
    boolean result = false;
    if(driver != null) {
        Class<?> aClass = null;
        try {
            aClass =  Class.forName(driver.getClass().getName(), true, classLoader);
        } catch (Exception ex) {
            result = false;
        }

         result = ( aClass == driver.getClass() ) ? true : false;
    }

    return result;
}
```

### **总结**

当高层提供了统一接口让低层去实现（面向接口编程，解耦），同时又要是在高层加载（或实例化）低层的类时，必须通过线程上下文类加载器来帮助高层的ClassLoader找到并加载该类。

## Jar Hell

Jar包地狱，[参考](https://www.hidennis.tech/2016/05/30/what-is-jar-hell/)

这个问题其实可以通过`OSGI`等组件化框架来解决，使用OSGI可以完美解决这个问题，OSGI是基于模块（Bundle）驱动的，每个模块都有属于自己的classpath和类加载器，模块之间通过包暴露和引入进行关联，每个模块有着自己独立的生命周期，我们可以动态地对模块进行加载、卸载、更新。如此看来，OSGI可以用一句话描述，就是一个为Java提供的动态模块化的系统。但是OSGI太过复杂，实用性并不强

[阿里架构师对OSGI的评价](http://hellojava.info/?p=152)

这里我主要想说的是怎么在代码中利用类加载器来检测Jar Hell

```java
public class JarHell {  
    public static void main(String[] args) {  
        try {  
            Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources("org/apache/log4j/Logger.class");  
            while(urls.hasMoreElements()) {  
                URL url = urls.nextElement();  
                System.out.println(url);  
            }  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
       
    }  
}  
```

这样就可以找到classpath中冲突的jar包，当然通过idea的工具会更方便 😂

## 参考

《深入理解Java虚拟机》

[深入探讨 Java 类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html#code4)

