---
title: 
  Java多线程之设计模式
tags: 
  [设计模式,多线程]
categories:
  [并发]
date: 2019/4/9
cover: http://static.imlgw.top/blog/20190726/C6uND3E3nwWJ.jpg?imageslim
---

## 观察者模式

>  观察者订阅被观察者的状态，当被观察者状态改变的时候会通知所有订阅的观察者的过程

**观察者接口**

```java
public  abstract class Observer {
    protected Subject subject;

    public Observer(Subject subject){
        this.subject=subject;
        subject.attach(this);
    }

    public  abstract  void update();
}
```

**观察者1**

```java
public class BinaryObserver extends Observer{

    public BinaryObserver(Subject subject) {
        super(subject);
    }

    @Override
    public void update() {
        System.out.println("Binarry String :"+ Integer.toBinaryString(subject.getState()));
    }
}
```

**观察者2**

```java
public class OctalOberver extends Observer{
    public OctalOberver(Subject subject) {
        super(subject);
    }

    @Override
    public void update() {
        System.out.println("Octal String:"+Integer.toOctalString(subject.getState()));
    }
}
```

**被观察者**

```java
public class Subject {
    //观察者们
    private  List<Observer> observers = new ArrayList<>();

    private int state;

    public int getState(){
        return this.state;
    }
	
    //注册观察者
    public void attach(Observer observer){
        observers.add(observer);
    }

    public void setState(int state){
        if(state==this.state){
            return;
        }
        this.state=state;
        notifyAllObserver();
    }
    //通知所有观察者线程
    private  void  notifyAllObserver(){
        observers.stream().forEach(Observer::update);
    }
}
```

**测试**

```java
public class ObserverCLi {
    public static void main(String[] args) {
        final Subject subject=new Subject();
        BinaryObserver binary=new BinaryObserver(subject);
        OctalOberver octalOberver = new OctalOberver(subject);
        System.out.println("==================");
        subject.setState(10);
        System.out.println("==================");
        subject.setState(12);
    }
}
```

## 读写锁分离模式

```java
public class ReadWriteLock {
    private int readingR = 0;
    private int waitingR = 0;
    private int writingW = 0;
    private int waitingW = 0;

    public synchronized void readLock() throws InterruptedException{
        this.waitingR++;
        try {
            //如果有线程在写就不能读
            while (writingW > 0) {
                this.wait();
            }
            this.readingR++;
        }finally {
            this.waitingR--;
        }
    }

    public synchronized void readUnlock() throws InterruptedException{
        this.readingR--;
        notifyAll();
    }

    public synchronized void writeLock() throws InterruptedException{
        this.waitingW++;
        try {
            while (readingR>0||writingW>0){
                this.wait();
            }
            this.writingW++;
        }finally {
            this.waitingW--;
        }
    }

    public synchronized void writeUnlock() throws InterruptedException{
        this.writingW--;
        notifyAll();
    }

}
```

只有读的时候不加锁，其他的时候加锁，在读的操作多于写的操作时，效率提升明显。

## 不可变对象设计模式

这个设计模式还是很重要也很常见的，`不可变`顾名思义，一个对象在被创建后对象所有的状态和属性都在其生命周期内都不会发生任何变化。

> 不可变对象一定是线程安全的(里面的任何属性或者应用类型的都不能被修改)，可变对象不一定是线程安全的(SrtingBuffer)。J2EE里面，Servlet就不是线程安全的，struts1的Action也不是线程安全的。

通常来说，创建不可变类原则有以下几条：

① 所有成员变量必须是`private`

② 最好同时用`final`修饰(非必须)

③ 不提供能够修改原有对象状态的方法

- 最常见的方式是不提供setter方法

- 如果提供修改方法，需要新创建一个对象，并在新创建的对象上进行修改

④ 通过构造器初始化所有成员变量，引用类型的成员变量必须进行深拷贝(deep copy)

⑤ getter方法不能对外泄露this引用以及成员变量的引用

⑥ 最好不允许类被继承(非必须)

　　JDK中提供了一系列方法方便我们创建不可变集合，如：

`Collections.unmodifiableList(List<? extends T> list)`

```java
final public class Person {
    //定义成final
    private final String name;
    private final String address;

    public Person(final String name,final String address) {
        this.name = name;
        this.address = address;
    }

    public String getAddress() {

        return address;
    }

    public String getName() {

        return name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

```

虽然说是不可变对象，但是其实通过反射等方法还是可以改变的。

```java
public class StringTest {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        /*String s="Hello";
        String s1=s.replace("l","K");
        System.out.println(s.hashCode());
        System.out.println(s1.hashCode());*/
        String s = "Hello World";
        System.out.println("s = " + s);//Hello World
		//String类里面的char[]
        Field valueFieldOfString = String.class.getDeclaredField("value");
        valueFieldOfString.setAccessible(true);

        char[] value = (char[]) valueFieldOfString.get(s);
        value[5] = '_';
        System.out.println("s = " + s); //Hello_World
    }
}
```

## Future设计模式(异步)

`Future`接口

```java
public interface Future<T> {
    T get() throws InterruptedException;
}

```

`FutureTask`接口

```java
public interface FutureTask<T> {
    T call();
}
```

`AsynFuture`异步

```java
public class AsynFuture<T> implements Future<T> {
    private volatile boolean done = false;

    private T result;

    public void done(T result) {
        synchronized (this) {
            this.result = result;
            this.done = true;
            //完成任务通知调用者
            this.notifyAll();
        }
    }

    @Override
    public T get() throws InterruptedException {
        synchronized (this) {
            while (!done) {
                this.wait();
            }
        }
        return result;
    }
}
```

`FutureService`连接`Future`和`FutureTask`

```java
public class FutureService {

    public <T> Future<T> submit(final FutureTask<T> task) {
        AsynFuture<T> asynFuture = new AsynFuture<>();
        new Thread(() -> {
            T result = task.call();
            asynFuture.done(result);
        }).start();
        return asynFuture;
    }

    //java8 回调 callback
    public <T> Future<T> submit(final FutureTask<T> task, final Consumer<T> consumer) {
        AsynFuture<T> asynFuture = new AsynFuture<>();
        new Thread(() -> {
            T result = task.call();
            asynFuture.done(result);
            consumer.accept(result);
        }).start();
        return asynFuture;
    }
}
```
测试`Future`

```java
public class SyncInvoker {
    public static void main(String[] args) throws InterruptedException {

        FutureService futureService = new FutureService();
        Future<String> submit = futureService.submit(() -> {
            try {
                Thread.sleep(10000l);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "FINISH";
        },System.out::println);

        System.out.println("===========");
        System.out.println(" do other thing.");
        Thread.sleep(1000);
        System.out.println("===========");
    }

    private static String get()
            throws InterruptedException {
        Thread.sleep(10000l);
        return "FINISH";
    }
}
```

> Future   ->未来的票据
>
> FutureTask   ->实际执行的任务
>
> FutureService  ->桥接Future和FutureTask

## Guarded Suspension设计模式

`保护性暂挂模式`

Guarded是被守护的意思。Suspension是暂停的意思，Guarded Suspension模式通过让线程等待来保证实例的安全性。

> 核心思想: 如果某个线程执行特定的操作前需要满足一定的条件，则在该条件未满足时将线程暂停运行（即暂挂线程，使其处于等待（waiting）状态，直到该条件满足时才继续运行）

**Request对象**

```java
public class Request {
    final private String value;

    public Request(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

**RequestQueue**

```java
public class RequestQueue {
    private final LinkedList<Request> queue = new LinkedList<>();

    public Request getRequest() {
        synchronized (queue) {
            while (queue.size() <= 0) {
                try {
                    //队列为空等一下
                    queue.wait();
                } catch (InterruptedException e) {
                    return null;
                }
            }
            Request request = queue.removeFirst();
            return request;
        }
    }

    public void putRequest(Request request) {
        synchronized (queue) {
            queue.addLast(request);
            queue.notifyAll();
        }
    }
}
```

**Server**

```java
public class ServerThread extends Thread{
    private final RequestQueue queue;

    private final Random random;

    private volatile boolean closed = false;

    public ServerThread(RequestQueue queue) {
        this.queue = queue;
        random = new Random(System.currentTimeMillis());
    }

    @Override
    public void run() {
        while (!closed) {
            Request request = queue.getRequest();
            //get可能会返回null
            if (null == request) {
                System.out.println("Received the empty request.");
                break;
            }
            System.out.println("Server ->" + request.getValue());
            try {
                Thread.sleep(random.nextInt(1000));
            } catch (InterruptedException e) {
                //打断后直接return
                return;
            }
        }
    }

    public void close() {
        this.closed = true;
        this.interrupt();
    }
}
```

**Client**

```java
public class ClientThread extends Thread {
    private final RequestQueue queue;

    private final Random random;

    private final String sendValue;

    public ClientThread(RequestQueue queue, String sendValue) {
        this.queue = queue;
        this.sendValue = sendValue;
        random = new Random(System.currentTimeMillis());
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println("Client -> request " + sendValue);
            //客户端发送请求
            queue.putRequest(new Request(sendValue));
            try {
                Thread.sleep(random.nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

**测试**

```java
public class SuspensionClient {
    public static void main(String[] args) throws InterruptedException {
        final RequestQueue queue = new RequestQueue();
        new ClientThread(queue, "Shaw").start();
        ServerThread serverThread = new ServerThread(queue);
        serverThread.start();
        //serverThread.join(); join住了后面还咋close？？？
        Thread.sleep(10000);
        serverThread.close();
    }
}
```



## ThreadLocal

线程局部变量，线程保险箱，以`Thread`作为`key`

> This class provides `thread-local variables`. These variables differ from their normal counterparts in that each thread that accesses one (via its `get` or `set` method) has its own, independently initialized copy of the variable.

模拟`ThreadLocal`

```java
public class ThreadLocalSimulator<T> {
    private final Map<Thread,T> threadMap=new HashMap<>();

    public void set(T t){
        Thread currentThread=Thread.currentThread();
        synchronized (threadMap){
            threadMap.put(currentThread,t);
        }
    }


    public T get(){
        Thread currentThread =Thread.currentThread();
        if(threadMap.get(currentThread)==null){
             threadMap.put(currentThread,initVal());
        }
        return threadMap.get(currentThread);
    }

    protected T initVal() {
        return null;
    }
}
```



## Balking设计模式

>核心思想：当不再适合或者没有必要进行这个操作时，就直接放弃进行这个操作而直接返回，不需要就算了

```java
public class BalkingData {
    private final String fileName;

    private String content;

    private boolean changed;

    public BalkingData(String fileName, String content, boolean changed) {
        this.fileName = fileName;
        this.content = content;
        this.changed = changed;
    }

    public synchronized void change(String newContent) {
        this.content = newContent;
        this.changed = true;
    }

    public synchronized void save() throws IOException {
        if (!changed) {
            //顾客没有服务请求，那么放弃提供服务操作，直接返回。
            return;
        }
        doSave();
        this.changed = false;
    }

    private void doSave() throws IOException {
        System.out.println(Thread.currentThread().getName() + " call  do save content");
        try(Writer writer = new FileWriter(fileName, true)) {
            writer.write(content);
            writer.write("\n");
            writer.flush();
        }
    }
}
```

**CustomerThread**

```java
public class CustomerThread extends Thread {

    private final BalkingData balkingData;

    private final Random random = new Random(System.currentTimeMillis());

    public CustomerThread(BalkingData balkingData) {
        super("Customer");
        this.balkingData = balkingData;
    }

    @Override
    public void run() {
        try {
            balkingData.save();
            for (int i = 0; i < 20; i++) {
                balkingData.change("No." + i);
                Thread.sleep(random.nextInt(1000));
                balkingData.save();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**WaiterThread**

```java
public class WaiterThread extends Thread {

    private final BalkingData balkingData;

    public WaiterThread(BalkingData balkingData) {
        super("Waiter");
        this.balkingData = balkingData;
    }

    @Override
    public void run() {
        for (int i = 0; i < 200; i++) {
            try {
                balkingData.save();
                Thread.sleep(1_000L);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## CountDown设计模式

**CountDown**

```java
public class CountDown {
    private final int total;

    //计数器
    private int counter;

    public CountDown(int total) {
        this.total = total;
    }

    public void down(){
        synchronized (this){
            this.counter++;
            this.notifyAll();
        }
    }

    public void await() throws InterruptedException {
        synchronized (this){
            while (counter!=total){
                this.wait();
            }
        }
    }
}
```

**测试**

```java
public class JDKCountDown {

    private static final Random random = new Random(System.currentTimeMillis());


    public static void main(String[] args) throws InterruptedException {
        //JDK的CountDown
        //final CountDownLatch latch=new CountDownLatch(5);
        final CountDown latch=new CountDown(5);

        System.out.println("准备多线程处理任务");
        //the first phase
        IntStream.range(0,5).forEach(i->{
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+" is working");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                latch.down();
            },String.valueOf(i)).start();
        });
        latch.await();
        //the second phase
        System.out.println("多线程任务全部结束，准备第二阶段任务");
        System.out.println("........");
        System.out.println("Finish");
    }
}
```

## 单例模式

### Double check

```java
public class SingletonObjectPlus {
    private static volatile  SingletonObjectPlus singletonObjectPlus =null;
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

> 这种方式在实例对象上要加上 `volatile`，避免实例对象没有初始化完成就返回

### InstanceHolder

```java
public class SingleGraceful {
    private SingleGraceful(){

    }
    //private静态内部类
    private static class InstanceHolder{
        //只会被初始化一次
        private final static SingleGraceful instance=new SingleGraceful();
    }

    public static SingleGraceful getInstance(){
        return SingleGraceful.InstanceHolder.instance;
    }
}
```

JVM只会为static变量分配一次内存，也就是只会初始化一次，而内部类不会在其外部类被加载的同时被加载，所以这也是一种很简洁很优秀的单例

### Enum

```java
public class SingleGraceful2 {
    private SingleGraceful2() {}
    
    private enum Singleton {
        INSTANCE;
        
        private final SingleGraceful2 instance;

        Singleton() {
            instance = new SingleGraceful2();
        }

        public SingleGraceful2 getInstance() {
            return instance;
        }
    }
    public static SingleGraceful2 getInstance() {
        return Singleton.INSTANCE.getInstance();
    }
}
```

- 枚举类构造函数是`private`类型的
- 枚举类的域(field)(`INSTANCE`)其实是相应的enum类型(`Singleton`)的一个静态实例对象，所以只会被初始化一次，构造器也只会被调用一次
- 枚举单例可以防止`反序列化` ，`反射`，`克隆`对单例的破坏，所以是一种极其优秀的单例实现。



> 这里其实还有很多没有介绍出来，设计模式这些东西没有实际的场景去用确实难以体会到它的精髓，需要慢慢的积累经验才行。

