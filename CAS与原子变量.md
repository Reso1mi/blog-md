---
title: CAS 与原子变量
tags:
  - 多线程
  - 并发编程
categories:
  - 并发
date: 2019/4/22
cover: 'http://static.imlgw.top/image/20190713/wallhaven-zmpd3j.jpg'
abbrlink: 2c91a79
---

## CAS

_这个** CAS **可不是单点登陆的那个 CAS😄!!!_

> CAS（Compare-and-Swap），是对一种处理器指令的称呼，很多 Java 多线程相关的类库的最终实现都会借助 CAS

​	从所周知，类似`i++`自增这样的操作并不是原子的，是一个`read-modify-write`的操作 ，如果要保证这种操作的原子性按照之前的做法可以使用`synchronized`内部锁来解决，但是这样似乎有点太小题大做了，锁确实可以解决这个问题，但是前面的文章也提到过，锁是很消耗性能的，并不是最好的做法，比较好的做法就是** CAS**，它能够将这些操作转换为原子操作。

​	Compare and Swap，比较并交换，顾名思义是一种`if-then-act`的操作，而这个操作的原子性由`处理器`保证（硬件锁），如果一个线程想要将变量 V 的值由 A 变为 B，借助 CAS 就会产生类似下面代码的作用

```java
boolean comapreAndSet(Variable V,Object A,Objext B){
    if(V.get()==A){ //判断是否和当前 V 的值相同（是否被修改）
        V.set(B);   //没被修改就更新
        return true;
    }
    return false; //被修改过就直接 return
}
```

这样一来就是先下手为强了，当你最先修改了 V 的值，后面的所有线程都会直接失败，所以实际上也是一种快速失败策略，当然你也可以尝试再次请求直到成功为止。

## 原子变量类

_原子变量类_是基于 CAS 实现的一组保证共享变量`read-modify-write`操作（例如自增）原子性的工具类

| 分组         | 类                                                           |
| ------------ | ------------------------------------------------------------ |
| 基础数据类型 | AtomicInteger，AtomicLong，AtomicBoolean                     |
| 数组类       | AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray    |
| 字段更新器   | AtomicIntegerFieldUpdater，AtomicLongFieldUpdater，AtomicReferenceFieldUpdater |
| 引用型       | AtomicReference，AtomicStampedReference，AtomicMarkableReference |

关于怎么使用就不多介绍，API 上都写的明明白白，这里有个地方需要注意，数组类单纯的 GET/SET 并不是原子操作。

### 利用 CAS 写一个锁

[前面的文章](http://imlgw.top/2019/04/07/java-duo-xian-cheng-xue-xi-bi-ji/)  利用内部锁实现过一个** BooleanLock**，这里利用 CAS 再实现一个简易的锁

**getLockException**

```java
public class GetLockException extends Exception{
    public GetLockException(String message) {
        super(message);
    }

    public GetLockException() {
        super();
    }
}
```

**CASLock** 

```java
public class CASLock {
    private static final AtomicInteger value = new AtomicInteger();

    private Thread lockedThread;

    public void trylock() throws GetLockException {
        boolean success = value.compareAndSet(0, 1);
        if (!success) {
            throw new GetLockException("获得锁失败");
        }
        lockedThread = Thread.currentThread();
    }

    public void unlock() {
        if (0 == value.get()) {
            return;
        }
        if (lockedThread == Thread.currentThread()) {
            //解铃还须系铃人
            boolean success = value.compareAndSet(1, 0);
            System.out.println(Thread.currentThread().getName() + " 释放了锁");
        }
    }
}
```

其实挺简单的，值得注意的地方就是释放锁的时候别忘了判断是不是当前线程加的锁，解铃还须系铃人😂

### ABA 问题

> 从所周知，CAS 成立的条件就是共享变量当前值和当前线程所提供的旧值相同，我们就可以认为这个变量没有被修改过，那么问题来了，对于一个共享变量** V**，如果当前线程看到它的时候它的值是 A，当它想执行 CAS 修改这个变量的时候，另一个线程将** V **的值从 A-->B-->A，那么这时当前线程再来执行 CAS 的时候，是否可以认为变量** V **没有被修改过呢？这里执行肯定是会成功的，但是这样结果是否可以接受呢 ?

#### 无法接受的例子

![mark](http://static.imlgw.top///20190423/vakfPgUQChGb.png?imageslim)

上图为用**单链表**实现的栈结构，若 T2 先抢到了执行权，将 A，B 弹出栈，然后依次`push`了 D，C，A，然后 T1 执行，利用 CAS，head.compareAndSet(A，B)，执行成功，栈顶变为 B，然而 B 就是个孤儿节点，这样一来 C，D 节点就被莫名其妙被丢掉了这显然是有问题的

#### 如何解决 ABA 问题

其实 ABA 问题并非完全无法接受，要考虑具体的场景，当然 Java 中也提供了解决的方案：

`AtomicStampedReference` 这个类看名字就知道是带了戳的，带了一个类似版本号的东西，直接上源码吧。

```java
public class AtomicStampedReference<V> {
    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

    private volatile Pair<V> pair;

    public AtomicStampedReference(V initialRef, int initialStamp) {
        pair = Pair.of(initialRef, initialStamp);
    }

    /**
     * Returns the current value of the reference.
     *
     * @return the current value of the reference
     */
    public V getReference() {
        return pair.reference;
    }

    public int getStamp() {
        return pair.stamp;
    }

    public V get(int[] stampHolder) {
        Pair<V> pair = this.pair;
        stampHolder[0] = pair.stamp;
        return pair.reference;
    }

    public boolean weakCompareAndSet(V   expectedReference,
                                     V   newReference,
                                     int expectedStamp,
                                     int newStamp) {
        return compareAndSet(expectedReference, newReference,
                             expectedStamp, newStamp);
    }

    /**
     * Atomically sets the value of both the reference and stamp
     * to the given update values if the
     * current reference is {@code ==} to the expected reference
     * and the current stamp is equal to the expected stamp.
     *
     * @param expectedReference the expected value of the reference
     * @param newReference the new value for the reference
     * @param expectedStamp the expected value of the stamp
     * @param newStamp the new value for the stamp
     * @return {@code true} if successful
     */
    public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference && //期望值和当前值相等
            expectedStamp == current.stamp &&	//期望的戳和当前的戳一致
            ((newReference == current.reference && //新的值是不是和当前的值一样
              newStamp == current.stamp) ||		//新的戳是不是和当前的戳一样
             casPair(current, Pair.of(newReference, newStamp))); //如果不一样就利用 CAS 设置新值
    }

    public void set(V newReference, int newStamp) {
        Pair<V> current = pair;
        if (newReference != current.reference || newStamp != current.stamp)
            this.pair = Pair.of(newReference, newStamp);
    }

    public boolean attemptStamp(V expectedReference, int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            (newStamp == current.stamp ||
             casPair(current, Pair.of(expectedReference, newStamp)));
    }

    // Unsafe mechanics 底层调用 unsafe 的方法
    private static final sun.misc.Unsafe UNSAFE = sun.misc.Unsafe.getUnsafe();
    private static final long pairOffset =objectFieldOffset(UNSAFE, "pair", AtomicStampedReference.class);
    //cas 设置新值
    private boolean casPair(Pair<V> cmp, Pair<V> val) {
        return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
    }

    static long objectFieldOffset(sun.misc.Unsafe UNSAFE,
                                  String field, Class<?> klazz) {
        try {
            return UNSAFE.objectFieldOffset(klazz.getDeclaredField(field));
        } catch (NoSuchFieldException e) {
            // Convert Exception to corresponding Error
            NoSuchFieldError error = new NoSuchFieldError(field);
            error.initCause(e);
            throw error;
        }
    }
}
```

这里删掉了部分注释， 可以看到里面封装了一个`Pair`里面有对象的引用和一个戳，在进行 CAS 的时候会判断`期望的引用`（传进来的引用）和`当前实际的引用`是不是一致，`期望的戳`（传进来的戳）和`当前实际的戳`是不是一致的，不一致就会直接`fail`，关键的 CAS 代码：

```java
public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return  
            expectedReference == current.reference && //期望值和当前值相等
            expectedStamp == current.stamp &&	//期望的戳和当前的戳一致
            ((newReference == current.reference && //新的值是不是和当前的值一样
              newStamp == current.stamp) ||		//新的戳是不是和当前的戳一样
             casPair(current, Pair.of(newReference, newStamp))); //如果不一样就利用 CAS 设置新值
 }
```

**测试 AtomicStampedReference**

```java
public class AtomicRefStampedTest {
   static AtomicStampedReference<Integer> reference=new AtomicStampedReference<>(100,0);

    public static void main(String[] args) {
        //第一个线程进行 ABA 操作
        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
                System.out.println("t1 "+reference.compareAndSet(100, 101, reference.getStamp(), reference.getStamp()+1));
                System.out.println(reference.getStamp()+","+reference.getReference());
                System.out.println("t1 "+ reference.compareAndSet(101, 100,reference.getStamp() , reference.getStamp()+1));
            } catch (InterruptedException e) {

            }

        }).start();

		//第二个线程等待第一个线程执行完
        new Thread(()->{
            try {
                int stamp = reference.getStamp();
                //假设执行到这里发生上下文切换
                System.out.println("Before sleep:stamp="+stamp);
                TimeUnit.SECONDS.sleep(5);
                System.out.println("After sleep:stamp="+reference.getStamp());
                boolean b = reference.compareAndSet(100, 101, stamp, stamp + 1);
                System.out.println(b);
            } catch (InterruptedException e) {

            }
        }).start();
    }
}
/**
	Before sleep:stamp=0
	t1 true
	1,101
	t1 true
	After sleep:stamp=2
	false
**/
```

结果肯定是 t2 执行失败了，毕竟戳不一样了，就算引用一样也没用。

#### 小插曲 (Integer 缓存）

这里一开始发生了一个小插曲，首先这里是的引用类型是 `Integer`类型的，然后我在进行 CAS 的时候从 100--->200 , 然后又从 200-->100，可能细心的朋友已经知道啥问题了，后面的从 200-->100 会失败，为啥？这个 200 和前面的 200 不是一个对象，引用不一样，那为啥 101 就可以呢？对，Integer 有一个缓冲池，大小在-128--127 之间的数，可以直接从缓冲池中拿，我开始在这里纠结了好一会儿😂

### 字段更新器

如果我们只需要某个类里的某个字段，也就是说让普通的变量也享受原子操作，可以使用原子更新字段类，如在某些时候由于项目前期考虑不周全，项目需求又发生变化，使得某个类中的变量需要执行多线程操作，由于该变量多处使用，改动起来比较麻烦，而且原来使用的地方无需使用线程安全，只要求新场景需要使用时，可以借助原子更新器处理这种场景，Java 中提供了几种字段更新器`AtomicIntegerFieldUpdater`，`AtomicLongFieldUpdater`，`AtomicReferenceFieldUpdater`，看名字就知道是对应啥的

#### AtomicIntegerFieldUpdater 测试

```java
public class AtomicIntegerFieldUpdaterTest {
    public static void main(String[] args) {
        AtomicIntegerFieldUpdater updater = AtomicIntegerFieldUpdater.newUpdater(TestUpdate.class, "num");
        TestUpdate testUpdate = new TestUpdate();
        Stream.of("t1", "t2", "t3", "t4", "t5").forEach(s -> {
            new Thread(() -> {
                int MAX = 100;
                for (int i = 0; i < MAX; i++) {
                    System.out.println(updater.getAndIncrement(testUpdate));
                }
            }, s).start();
        });
    }

    static class TestUpdate {
        volatile int num;
    }
}
```

这样就保证了 Integer 字段自增操作的原子性，另外两个与之类似。

**需要注意的地方**

- 操作的字段不能是 static 类型。

- 操作的字段不能是 final 类型的，因为 final 根本没法修改。

- 字段必须是 volatile 修饰的，也就是数据本身是读一致的。

- 属性必须对当前的 Updater 所在的区域是可见的，也就是说无论何时都应该保证操作类与被操作类间的可见性。

![mark](http://static.imlgw.top///20190430/hIuzU4rePaAe.png?imageslim)

> 使用字段更新器比起直接使用原子类要节约内存，但是操作起来不方便

## Unsafe 双刃剑

**Unsafe **类，看名字就知道不安全，并不是它写的不安全，而是用起来不安全，因为它可以像 c/c++一样去操作内存地址，**unsafe **里面的所有方法都是** native **的，底层都是 c/c++实现的，直接与操作系统底层交互，上面 CAS 执行也依赖于** unsafe **类中的方法，其实整个并发包里的类都依赖于** unsafe**，但是官方并不建议用户使用这个类

- Unsafe 有可能在未来的 Jdk 版本移除或者不允许 Java 应用代码使用，这一点可能导致使用了 Unsafe 的应用无法运行在高版本的 Jdk
- Unsafe 的不少方法中必须提供原始地址（内存地址）和被替换对象的地址，偏移量要自己计算，一旦出现问题就是 JVM 崩溃级别的异常，会导致整个 JVM 实例崩溃，表现为应用程序直接崩掉。
- Unsafe 提供的直接内存访问的方法中使用的内存不受 JVM 管理（无法被 GC)，需要手动管理，一旦出现疏忽很有可能成为内存泄漏的源头。

### 获取 Unsafe

```java
    //获取 Unsafe
    public static Unsafe getUnsafe() {
        Field f = null;
        try {
            f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            return (Unsafe)f.get(null);
        } catch (Exception e) {
            throw new RuntimeException();
        }
    }
```

### CAS 相关

Java 中的 CAS 实现调用的就是三个本地方法，第一个参数代表的就是实例对象，第二个参数代表需要 CAS 字段在该实例上的偏移量（不用自己计算，Unsafe 提供了方法计算偏移量），第三个参数就是期望值，最后一个参数就是更新的值

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

### 利用 Unsafe 自己写一个原子 Counter

```java
class CASCounter implements Counter {
    private volatile long counter = 0;

    public CASCounter() throws NoSuchFieldException {
        unsafe = getUnsafe();
        //获取 counter 字段的内存偏移量
        offset= unsafe.objectFieldOffset(CASCounter.class.getDeclaredField("counter"));
        System.out.println(offset);
    }
	
    private Unsafe unsafe;
    private long offset;
    
    public static Unsafe getUnsafe() {
        Field f = null;
        try {
            f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            return (Unsafe)f.get(null);
        } catch (Exception e) {
            throw new RuntimeException();
        }
    }

    @Override
    public void increment() {
        long current=counter;
        while (!unsafe.compareAndSwapLong(this,offset,current,current+1)){
            current=counter;
        }
    }

    @Override
    public long getCounter() {
        return counter;
    }
}

interface Counter {
    void increment();

    long getCounter();
}
```

### Unsafe 的骚操作

**绕过构造器创建对象**

```java
public class UnsafeFooTest {
    public static void main(String[] args) throws ClassNotFoundException,InstantiationException, NoSuchFieldException {
        Unsafe unsafe = UnsafeTest.getUnsafe();
        //绕过构造器创建对象
        Simple simple = (Simple) unsafe.allocateInstance(Simple.class);
        System.out.println(simple.get()); //null
        System.out.println(simple.getClass().getClassLoader());
    }

    static class Simple {
        private String a = "a";
        public Simple() {
            a = "new";
            System.out.println("============== ");
        }

        public String get() {
            return a;
        }
        static {
            System.out.println("静态代码块");
        }
    }
}
```

**修改字段在内存中的值**

```java
public class UnsafeFooTest {
    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, NoSuchFieldException {
        Unsafe unsafe = UnsafeTest.getUnsafe();
        Permission permission = new Permission();
        permission.doSth();
        //通过反射也可以做到，但是 unsafe 直接是到内存地址中将值修改了
        Field access_allow = permission.getClass().getDeclaredField("ACCESS_ALLOW");
        //获取字段再对象中的内存偏移量，可以简单理解为指针（内存地址）
        unsafe.putLong(permission,unsafe.objectFieldOffset(access_allow),-1);
        permission.doSth();
    }
}

class Permission {
    private int ACCESS_ALLOW = 0;

    private boolean isAllow() {
        return  ACCESS_ALLOW==-1;
    }

    public void doSth() {
        if (isAllow()) {
            System.out.println("i am workind");
        }
    }
}
```

**defindClass 加载类文件**

```java
public class UnsafeFooTest {

    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, NoSuchFieldException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {
        Unsafe unsafe = UnsafeTest.getUnsafe();
        byte[] bytes = loadClassContent();
        Class<?> aClass = unsafe.defineClass(null, bytes, 0, bytes.length, ClassLoader.getSystemClassLoader(), null);
        int get = (int) aClass.getMethod("get").invoke(aClass.newInstance(), null);
        System.out.println(get);
    }
    
    //将 class 字节码加载到内存中
    public static byte[] loadClassContent() {
        File f = new File("D:\\ClassLoaderTest\\Res.class");
        FileInputStream stream = null;
        byte[] bytes=null;
        try {
            stream = new FileInputStream(f);
             bytes = new byte[(int) f.length()];
            stream.read(bytes);
            stream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return bytes;
    }
}
```

将编译好的 class 文件放到对应的目录下

```java
public class Res {
    private int i=0;

    public Res(){
        this.i=1;
    }

    public int get() {
        return i;
    }
}
```

**结果**，是不是很牛皮？🐂🍺是🐂🍺但是这个玩意尽量的别用。

![mark](http://static.imlgw.top/blog/20190720/zLJJUhQ6GAdc.png?imageslim)

**Unsafe **里面的方法还有很多这里就不都列举了，毕竟暂时还用不到，如果想了解更多可以看看这几篇文章

- [JAVA 中神奇的双刃剑--Unsafe](https://www.cnblogs.com/throwable/p/9139947.html)

- [R 大关于 Unsafe 的使用建议](https://www.zhihu.com/question/29266773?sort=created)

- [Java 魔法类：Unsafe 应用解析](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)