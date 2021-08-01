---
title: Socket 网络编程
tags:
  - Socket
categories:
  - 网络编程
date: 2019/7/19
cover: 'http://static.imlgw.top/image/20190719/Ob9k514MhYVn.png?imageslim'
abbrlink: df38ad26
---

### Socket 概述

​	套接字（socket）是一个抽象层，应用程序可以通过它发送或接收数据，可对其进行像对文件一样的打开、读写和关闭等操作。套接字允许应用程序将 I/O 插入到网络中，并与网络中的其他应用程序进行通信。网络套接字是 IP 地址与端口的组合。socket 提供的函数是**操作系统内核将“TCP/IP 协议栈 + 底层网卡”抽象出来的一个个用户友好的函数，用于操纵本地的“TCP/IP 协议栈 + 底层网卡”与远端的服务器/主机完成通信的任务。**

### TCP

🔸 TCP 是面向连接的通信协议

🔸 通过`三次握手`建立连接，通讯完成时要拆除连接

🔸 由于 TCP 是面向连接的所以只能用于端到端的通讯

🔸 三次握手四次挥手

🔸 具有校验机制，可靠，数据传输稳定

#### 简单的 Socket 小案例

**Socket 客户端**

```java
package TcpDemo;
import java.io.*;
import java.net.*;
import java.util.Scanner;

/**
 * @author imlgw.top
 * @date 2019/7/7 9:45
 */
public class Client {
    private static final int REMOTE_PORT = 20000;

    private static final int LOCAL_PORT = 30000;

    public static void main(String[] args) throws IOException {
        Socket socket = creatSocket();
        initSocket(socket);
        //setRecessAddress 前
        socket.bind(new InetSocketAddress(InetAddress.getLocalHost(), LOCAL_PORT));
        //连接远程 server
        socket.connect(new InetSocketAddress(InetAddress.getLocalHost(), REMOTE_PORT), 3000);
        System.out.println("客户端已经发起连接");
        System.out.println("客户端信息：" + socket.getLocalAddress() + "port:" + socket.getLocalPort());
        System.out.println("服务端信息" + socket.getInetAddress() + "port:" + socket.getPort());
        try {
            sendMsg(socket);
        } catch (Exception e) {
            System.err.println("连接异常关闭！！！！");
            e.printStackTrace();
        } finally {
            socket.close();
        }
    }

    private static void initSocket(Socket socket) throws SocketException {
        socket.setSoTimeout(3000);
        //是否复用未完全关闭后的端口 (TIME_WAIT 状态），必须在 bind 前，所以就不能通过构造器来绑定本地端口
        socket.setReuseAddress(true);
        //是否开启 Nagle 算法（默认开启） https://baike.baidu.com/item/Nagle%E7%AE%97%E6%B3%95
        socket.setTcpNoDelay(false);
        //长时间无数据相应的时候发送确认数据（心跳包）时间大约两个小时
        socket.setKeepAlive(true);
        
        /*
          close 关闭后的处理
          这个 Socket 选项可以影响 close 方法的行为。
          false 0 默认情况 关闭后立即返回，底层系统接管输出流，将缓冲区的数据发送完成
          true 0 关闭后直接返回 缓冲区数据直接抛弃 直接发送 RES 结束命令到对方，无需经过 2MSL 等待
          true 200 关闭时最长阻塞 200s 随后按第二情况处理
          （是 s 不是 ms, 开始搞错了 设置了 20 重启就会端口占用。)
        */
        socket.setSoLinger(true, 0);
       
        /*
        设置紧急数据是否内敛，如果这个 Socket 选项打开，
        可以通过 Socket 类的 sendUrgentData 方法
        向服务器发送一个单字节的数据 这个单字节数据并不经过输出缓冲区，而是立即发出。
        虽然在客户端并不是使用 OutputStream 向服务器发送数据，
        但在服务端程序中这个单字节的数据是和其它的普通数据混在一起的
        因此，在服务端程序中并不知道由客户
        端发过来的数据是由 OutputStream
        还是由 sendUrgentData 发过来的
        */
        socket.setOOBInline(true);
        //设置收发缓冲器大小，默认 32K
        socket.setReceiveBufferSize(64*1024);
        socket.setSendBufferSize(64*1024);
        //设置性能参数的 优先级  短链接 延迟 带宽
        socket.setPerformancePreferences(1,1,1);
    }
    @SuppressWarnings("all")
    private static Socket creatSocket() throws IOException {
        /*
        //无代理模式，相当于空构造函数
        Socket socket = new Socket(Proxy.NO_PROXY);
        //HTTP 代理模式传输的数据将通过 www.imlgw.top 转发
        socket = new Socket(new Proxy(Proxy.Type.HTTP, new InetSocketAddress(Inet4Address.getByName("www.imlgw.top"), 80)));

        //下面两种方式回在创建的时候就去链接远程的服务器（具体看源码）, 然而一般情况下其实在连接之前我们还需要设置一些参数
        //新建一个套接字 链接到远程服务器和端口（本地端口为系统分配）
        socket = new Socket("imlgw.top", REMOTE_PORT);
        //新建套接字直接链接到远程端口 并绑定本地端口
        socket=new Socket("imlgw.top",REMOTE_PORT,InetAddress.getLocalHost(),LOCAL_PORT);
        */

        //新建 socket 然后绑定到本地端口
        Socket socket = new Socket();
        return socket;
    }

    private static void sendMsg(Socket socket) throws IOException {
        //键盘的输入流
        Scanner scanner = new Scanner(System.in);
        //拿到 socket 的输出流
        OutputStream socketOutputStream = socket.getOutputStream();
        //转换为打印流
        PrintStream printStream = new PrintStream(socketOutputStream);
        //socket 的输入流
        InputStream inputStream = socket.getInputStream();
        //转换位 buffer 流
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));

        boolean flag = true;
        do {
            //客户端发送消息
            printStream.println(scanner.nextLine());
            //服务端的响应
            String s = bufferedReader.readLine();
            if ("bye".equals(s)) {
                flag = false;
            } else {
                System.out.println("服务端响应：" + s);
            }
        } while (flag);
        bufferedReader.close();
        printStream.close();
        scanner.close();
    }
}
```

**Socket 服务端**

 这里为了同时处理多个客户端设计成了**多线程**异步的模式

```java
package TcpDemo;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.*;

/**
 * @author imlgw.top
 * @date 2019/7/7 9:46
 */
public class Server {

    private static final int SERVER_PORT = 20000;

    public static void main(String[] args) throws IOException {
        ServerSocket server = creatServerSocket();
        initServerSocket(server);
        //初始化之后再绑定，不然一些设置会失效，比如 setReuseAddress
        server.bind(new InetSocketAddress(InetAddress.getLocalHost(), SERVER_PORT), 50);
        System.out.println("服务器准备就绪");
        System.out.println("服务端信息" + server.getInetAddress() + " port:" + server.getLocalPort());
        //监听客户端的消息
        while (true) {
            //阻塞方法
            Socket client = server.accept();
            ClientHandle clientHandle = new ClientHandle(client);
            new Thread(clientHandle).start();
        }
    }

    private static void initServerSocket(ServerSocket server) throws SocketException {
        //同 client
        server.setReuseAddress(true);
        //设置 accept 的 buffer
        server.setReceiveBufferSize(64 * 1024);
        //设置 timeout
        //server.setSoTimeout(2000);
        //设置性能参数，连接前设置
        server.setPerformancePreferences(1, 1, 1);
    }

    private static ServerSocket creatServerSocket() throws IOException {
        ServerSocket server = new ServerSocket();
        //绑定端口 backlog: 新连接队列的长度限制，不是链接的数量，是允许等待的队列长度
        //server.bind(new InetSocketAddress(InetAddress.getLocalHost(),SERVER_PORT),50);
        //server =new ServerSocket(SERVER_PORT,50); 等效方案
        //server =new ServerSocket(SERVER_PORT,50,InetAddress.getLocalHost());
        return server;
    }

    private static class ClientHandle implements Runnable {
        private Socket socket;

        ClientHandle(Socket client) {
            this.socket = client;
        }

        //接收消息
        public void run() {
            System.out.println("新客户端连接：" + socket.getInetAddress() + "port：" + socket.getPort());
            try {
                //输入流获取信息
                BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                //输出流响应客户端
                PrintStream printStream = new PrintStream(socket.getOutputStream());
                boolean flag = true;
                do {
                    String s = reader.readLine();
                    if ("bye".equalsIgnoreCase(s)) {
                        flag = false;
                        System.out.println("客户端关闭了连接");
                        printStream.println("bye");
                    } else {
                        System.out.println(s);
                        printStream.println("字符串长度#" + s.length());
                    }
                } while (flag);
                reader.close();
                printStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

> 具体的一些常用的方法解释都在代码的注释中 [参考](https://elf8848.iteye.com/blog/1739598)

![mark](http://static.imlgw.top/image/20190707/eBKfzShaiIHU.png?imageslim)

#### 传输基本数据类型

上面的哪个小案例传送的都是字符串类型的数据，也许有同学会说这些基本类型不都是可以通过字符串来传吗？为什么要费那个劲去传这些基本类型？其实不然，这里假设要传送的是 int 类型的 12345678 ，如果通过 `int` 来传输只要在范围内都是** 4 **个字节大小固定，然而通过`String`传送将会是"12345678" 也就是** 8 **个字节，消耗要比使用`int`要大，而且长度不固定，不方便后期接受的长度判断。

 **`int` 类型**  

在网络上传输的都是以** Byte **为基本单位，如果要传送** int **我们就需要将** int **转换为** byte**，一个** int **是 4 个字节，我们可以将其转换为一个** byte[] **数组，废话不多说，上代码

```java
/**
 * @author imlgw.top
 * @date 2019/7/11 13:23
 */
public class ByteTools {
    public static byte[] int2byte(int a) {
        //无符号右移
        return new byte[]{
                (byte) (a >>> 24),
                (byte) (a >>> 16),
                (byte) (a >>> 8),
                (byte) (a)
        };
    }
    public static int byte2int(byte[] a) {
        //&0xff-->转换为 int 将高位补 0, 低 8 位不变
        //-127 ：10000001（补） &0xff --> 00000000 00000000 00000000 10000001
        return a[3] & 0xff | (a[2] & 0xff) << 8 | (a[1] & 0xff) << 16 | (a[0] & 0xff) << 24;
    }
}
```

为什么这样做的一些细节可以参考 [这篇博客](https://www.cnblogs.com/think-in-java/p/5527389.html)

有了这个工具类我们就可以将** int **转换为** byte[] **后进行传输，同时接收端也可以通过这个方法将数据还原。

等等🙄 ，这样一来不是所有的类型对应的都要去写个这样的转换的方法？那还是有点麻烦的，而且也没有什么技术含量，所以这样的事情** JDK **帮我们做了

**`ByteBuffer`**：nio 中的一个包，这里我还不太熟悉这个具体的作用，目前只知道可以用来包装** byte[]**，然后可以实现上面的类型转换

**Client 发送端**

```java
byte[] buffer=new byte[256];
//包装 buffer （装饰器模式？
ByteBuffer byteBuffer = ByteBuffer.wrap(buffer);
//byte  1
byteBuffer.put((byte) 126);
//int 类型 4
byteBuffer.putInt(123);
//char 2(unicode)
byteBuffer.putChar('A');
//long 8
byteBuffer.putLong(323333231234124321L);
boolean isOk=true;
//byte 1
byteBuffer.put((byte) (isOk?1:0));
//float 4
byteBuffer.putFloat(123.2132F);
//double 8 =28
byteBuffer.putDouble(231.1412421321);
//String 10
byteBuffer.put("HelloWorld".getBytes());
//发送 38 Byte 
socketOutputStream.write(buffer,0,byteBuffer.position());
```

需要注意的地方就是最后** write **的时候，第二个参数** len**，直接传** position()**, 就可以了，不用+1，这个 position 是下一个字节位置

> 这里其实我看的教程这里是加 1 了的，最后接收过来的数据长度死活对不上，我开始还以为是什么**内存对齐**，什么乱七八糟的然后才发现是这里有问题。

**Server 接收端**

基本类型的读取与上面对应的** get**，最后一个 String 需要注意，直接用原始的** buffer **就可以了，不需要借助** ByteBuffer**，这里同样后面不用-1

```java
String str = new String(buffer, byteBuffer.position(), readByteCount-byteBuffer.position());
```

**测试结果**

```java
服务器准备就绪
服务端信息 LAPTOP-V5R5ABUJ/192.168.25.1 port:20000
新客户端连接：/192.168.25.1port：30000
当前下标 28
接受到 Client 数据长度 (byte)：38
Client 发送的数据：
126
123
A
323333231234124321
true
123.2132
231.1412421321
HelloWorld
```

### UDP

🔸  UDP 是无连接的，即发送数据之前不需要建立连接，因此减少了开销和发送数据之前的时延。

🔸  UDP 使用尽最大努力交付，即不保证可靠交付，因此主机不需要维持复杂的连接状态表。

🔸   UDP 是面向报文的。发送方的 UDP 对应用程序交下来的报文，在添加首部后就向下交付 IP 层。UDP 对应用层交下来的报文，既不合并，也不拆分，而是保留这些报文的边界。因此，应用程序必须选择合适大小的报文。

🔸   UDP 没有拥塞控制，因此网络出现的拥塞不会使源主机的发送速率降低。很多的实时应用（如 IP 电话、实时视频会议等）要去源主机以恒定的速率发送数据，并且允许在网络发生拥塞时丢失一些数据，但却不允许数据有太多的时延。UDP 正好符合这种要求。

🔸   UDP 支持一对一、一对多、多对一和多对多的交互通信。

🔸   UDP 的首部开销小，只有 8 个字节，比 TCP 的 20 个字节的首部要短。

#### 单播

**消息接收者**

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketAddress;
/**
 * @author imlgw.top
 * @date 2019/7/7 21:31
 */
public class UDPProvide {
    public static void main(String[] args) throws IOException {
        //监听 20000 端口
        DatagramSocket socket=new DatagramSocket(20000);
        System.out.println("UDPProvide is start....");
        final byte[] buf=new byte[512];
        //构建接受的 DatagramPacket
        DatagramPacket udp_receive=new DatagramPacket(buf,buf.length);
        //构建接受的 DatagramPacket （阻塞）
        socket.receive(udp_receive);
        //获取发送人的 SocketAddress
        SocketAddress socketAddress = udp_receive.getSocketAddress();
        int datalen = udp_receive.getLength();
        //获取发送的数据
        String receive=new String(udp_receive.getData(),0,datalen);
        System.out.println("receive from the: "+socketAddress);
        System.out.println("receive data: "+ receive);
        //构建响应的 DatagramPacket
        byte[] bytes = ("provider receive the data success "+datalen).getBytes();
        DatagramPacket udp_sendBack=new DatagramPacket(bytes,bytes.length,socketAddress);
        socket.send(udp_sendBack);
        //结束
        System.out.println("UDPProvide Finished.");
        socket.close();
    }
}
```

**消息发送者**

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
/**
 * @author imlgw.top
 * @date 2019/7/7 21:31
 */

public class UDPSearch {
    public static void main(String[] args) throws IOException {
        System.out.println("UDPSearch is ready...");
        //构建 socket
        DatagramSocket socket = new DatagramSocket();
        byte[] buff= "hello world".getBytes();
        //构建发送段
        DatagramPacket udp_send=new DatagramPacket(buff,buff.length);
        //指定对方 ip
        udp_send.setAddress(InetAddress.getLocalHost());
        udp_send.setPort(20000);
        socket.send(udp_send);
        //获取响应段
        final byte[] buf=new byte[512];
        DatagramPacket udp_receive=new DatagramPacket(buf,buf.length);
        socket.receive(udp_receive);
        String s = new String(udp_receive.getData(), 0, udp_receive.getLength());
        System.out.println(s);
        System.out.println("UDPSearch is over");
        socket.close();
    }
}
```

#### 多播&广播

**消息建造器**

```java
/**
 * @author imlgw.top
 * @date 2019/7/8 8:48
 */
public class MessageCreator {
    private static final String SN_HEADER = "收到暗号，我是 SN:";
    private static final String PORT_HEADER = "这是暗号，请回送到该端口：";

    public static String buildWithPort(int port) {
        return PORT_HEADER + port;
    }

    public static int parsePort(String sn) {
        if (sn.startsWith(PORT_HEADER)) {
            return Integer.parseInt(sn.substring(PORT_HEADER.length()));
        }
        return  -1;
    }

    public static String buildWithSn(String sn){
        return SN_HEADER+sn;
    }

    public static String parseSn(String sn){
        if(sn.startsWith(SN_HEADER)){
            return sn.substring(SN_HEADER.length());
        }
        return null;
    }
}
```

**消息接受者**

```java
package UdpDemo2;
import java.io.IOException;
import java.net.*;
import java.util.UUID;

/**
 * @author imlgw.top
 * @date 2019/7/7 21:31
 */
public class UDPProvide {
    public static int PROVIDE_LISTEN_PORT = 20000;

    public static void main(String[] args) throws IOException {
        String sn = UUID.randomUUID().toString();
        Provider provider = new Provider(sn);
        new Thread(provider).start();
        System.in.read();
        provider.shutdown();
    }

    public static class Provider implements Runnable {
        public volatile boolean isDone = false;
        public DatagramSocket socket = null;
        public final String sn;

        public Provider(String sn) {
            this.sn = sn;
        }

        public void run() {
            System.out.println("UDPProvide is start....");
            try {
                socket = new DatagramSocket(PROVIDE_LISTEN_PORT);
                while (!isDone) {
                    final byte[] buf = new byte[512];
                    //构建接受的 DatagramPacket
                    DatagramPacket udp_receive = new DatagramPacket(buf, buf.length);
                    //接受 DatagramPacket （阻塞）
                    socket.receive(udp_receive);
                    //获取发送人的 SocketAddress
                    InetSocketAddress socketAddress = (InetSocketAddress) udp_receive.getSocketAddress();
                    int datalen = udp_receive.getLength();
                    //获取发送过来的数据
                    String receive = new String(udp_receive.getData(), 0, datalen);
                    //打印获取到的数据
                    System.out.println("receive from the: " + socketAddress);
                    System.out.println("receive data: " + receive);
                    //解析 sn, 获取需要回送的端口
                    int port = MessageCreator.parsePort(receive);
                    if (port != -1) {
                        //构建回送的 DatagramPacket
                        byte[] responseBody = MessageCreator.buildWithSn(sn).getBytes();
                        DatagramPacket udp_sendBack = new DatagramPacket(responseBody, 0, responseBody.length, socketAddress.getAddress(), port);
                        socket.send(udp_sendBack);
                    }
                }
            } catch (IOException e) {
                // e.printStackTrace();
            } finally {
                closeRes();
            }
            //结束
            System.out.println("UDPProvide Finished.");
        }

        public void shutdown() {
            isDone = true;
            //这里仅仅 isDone=true 远远不够，因为 socket.receive 是一个永久阻塞的方法
            //所以下面还要 close 这个 socket 这样就会捕获到一个异常然后结束
            closeRes();
        }

        private void closeRes() {
            if (socket != null) {
                socket.close();
                socket = null;
            }
        }
    }
}
```

与上面不同的是这里为了随时可以停止将其构建成了异步线程，当接受到终止信号的时候就会改变状态量，并 close 资源，然后利用异常停止线程。

**消息发送者**

```java
package UdpDemo2;

import java.io.IOException;
import java.net.*;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;

/**
 * @author imlgw.top
 * @date 2019/7/7 21:31
 */

public class UDPSearch {
    private static final int SEARCH_LISTEN_PORT = 30000;
    private static DatagramSocket socket = null;
    public static void main(String[] args) throws IOException, InterruptedException {
        System.out.println("UDPSearch is start...");
        Listener listen = listen();
        sendBoard();
        System.in.read();
        List<Device> devices = listen.closeAndGetDeviceList();
        for (Device device : devices) {
            System.out.println(device);
        }
    }

    private static Listener listen() throws InterruptedException {
        System.out.println("UDPSearch Listener is start");
        CountDownLatch countDownLatch=new CountDownLatch(1);
        Listener listener=new Listener(SEARCH_LISTEN_PORT,countDownLatch);
        new Thread(listener).start();
        countDownLatch.await();
        return listener;
    }

    public static void sendBoard() throws IOException {
        //系统自动分配的端口
        DatagramSocket socket = new DatagramSocket();
        //构建 socket
        byte[] buff = MessageCreator.buildWithPort(SEARCH_LISTEN_PORT).getBytes();
        //构建发送段
        DatagramPacket udp_send = new DatagramPacket(buff, buff.length);
        //广播地址
        udp_send.setAddress(InetAddress.getByName("255.255.255.255"));
        //接收方的端口
        udp_send.setPort(UDPProvide.PROVIDE_LISTEN_PORT);
        socket.send(udp_send);
        socket.close();
        System.out.println("UDPSearch Board is over");
    }

    private static class Device {
        int port;
        String ip;
        String sn;

        public Device(int port, String ip, String sn) {
            this.port = port;
            this.ip = ip;
            this.sn = sn;
        }
        @Override
        public String toString() {
            return "Device{" +
                    "port=" + port +
                    ", ip='" + ip + '\'' +
                    ", sn='" + sn + '\'' +
                    '}';
        }
    }

    public static class Listener implements Runnable {
        private final int listenPort;
        private final CountDownLatch countDownLatch;
        private final List<Device> deviceList = new ArrayList<Device>();
        //private static DatagramSocket socket = null;

        private boolean isDone = false;

        public Listener(int listenPort, CountDownLatch countDownLatch) {
            this.listenPort = listenPort;
            this.countDownLatch = countDownLatch;
        }

        public void run() {
            //通知已经启动
            countDownLatch.countDown();
            try {
                socket = new DatagramSocket(listenPort);
                while (!isDone) {
                    final byte[] buf = new byte[512];
                    //构建接受的 DatagramPacket
                    DatagramPacket udp_receive = new DatagramPacket(buf, buf.length);
                    //接受 DatagramPacket （阻塞）
                    socket.receive(udp_receive);
                    //获取发送人的 SocketAddress
                    InetSocketAddress socketAddress = (InetSocketAddress) udp_receive.getSocketAddress();
                    int datalen = udp_receive.getLength();
                    //获取发送过来的数据
                    String sn = new String(udp_receive.getData(), 0, datalen);
                    System.out.println("back from the：" + socketAddress);
                    System.out.println("back data：" + sn);
                    sn=MessageCreator.parseSn(sn);
                    if (sn != null) {
                        deviceList.add(new Device(socketAddress.getPort(), socketAddress.getAddress().toString(), sn));
                    }
                }
            } catch (IOException e) {
               //e.printStackTrace();
            } finally {
                closeRes();
            }
            System.out.println("UDPSearch Listener is Finished...");
        }

        public void closeRes() {
            if (socket != null) {
                socket.close();
                socket = null;
            }
        }

        List<Device> closeAndGetDeviceList() {
            isDone = true;
            closeRes();
            return deviceList;
        }
    }
}

```

这里需要注意的就是广播的地址** 255.255.255.255**

> 如果是在局域网内和其他机器通信需要关闭虚拟机的网卡，不然是走的虚拟机的网卡，其他机器接收不到。（我说怎么发送的** IP **不是我的本机的** IP**)

🔸UDP 是面向无连接的通讯协议，基于用户数据报的协议

🔸UDP 数据包括目的端口号和源端口号信息

🔸通讯不需要连接，所以可以实现广播发送，并不局限于端到端

🔸结构简单，无校验，速度快，容易丢包，可广播

🔸他一旦把应用程序发给网络层的数据发送出去就不保留数据备份

### UDP 辅助 TCP 实现点对点传输

客户端先利用`UDP`向局域网发送广播，然后对应的服务器接收到之后就会将对应的`TCP`的端口回送给客户端，然后二者进行 TCP 的双向通信，代码太多这里就只放一下`Server`端的`Handler`

```java
package udp_tcp_concurrency.server.handle;

import udp_tcp_concurrency.utils.CloseUtils;

import java.io.*;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 用来处理客户端的类
 * @author imlgw.top
 * @date 2019/7/17 11:48
 */
public  class ClientHandler {

    private final Socket socket;
    private final ServerReadHandler serverReadHandler;
    private final ServerWriterHandler serverWriterHandle;

    private final CloseNotify closeNotify;

    public ClientHandler(Socket socket, CloseNotify closeNotify) throws IOException {
        this.socket = socket;
        this.serverReadHandler = new ServerReadHandler(socket.getInputStream());
        this.serverWriterHandle = new ServerWriterHandler(socket.getOutputStream());
        this.closeNotify = closeNotify;
        System.out.println("新客户端连接：" + socket.getInetAddress() + "port：" + socket.getPort());
    }

    public void send(String str) {
        serverWriterHandle.send(str);
    }

    //从外界关闭
    public void stop() {
        serverReadHandler.stopRead();
        serverWriterHandle.stopWriter();
        CloseUtils.close(socket);
        System.out.println("客户端已经退出");
        System.out.println("address:" + socket.getInetAddress() + ",port:" + socket.getPort());
    }

    //自我关闭--->自闭
    private void stopByMyself() {
        stop();
        closeNotify.onSelfClosed(this);
    }

    //读取并打印到屏幕（启动 ClientReadHandle 线程）
    public void read2Print() {
        new Thread(serverReadHandler).start();
    }

    /**
     *  将已经关闭的 handle 暴露给 TCPServer 然后从 list 中移除
     */
    public interface CloseNotify{
        void onSelfClosed(ClientHandler clientHandler);
    }

    /**
     * 处理服务端用于读取客户端消息的 Handle
     */
    class ServerReadHandler implements Runnable {
        private boolean done = false;
        private final InputStream inputStream;

        public ServerReadHandler(InputStream inputStream) {
            this.inputStream = inputStream;
        }

        public void run() {
            try {
                //输入流获取信息
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                do {
                    String s = reader.readLine();
                    if (s == null) {
                        System.out.println("客户端已经无法发送数据");
                        //结束当前 Handle
                        ClientHandler.this.stopByMyself();
                        break;
                    }
                    //打印到屏幕
                    System.out.println(s);
                } while (!done);
            } catch (IOException e) {
                if (!done) {
                    //非正常关闭
                    System.err.println("连接异常断开"+e.getMessage());
                    ClientHandler.this.stopByMyself();
                }
            } finally {
                CloseUtils.close(inputStream);
            }
        }

        public void stopRead() {
            done = true;
            CloseUtils.close(inputStream);
        }
    }

    /**
     * 处理服务端向客户端发送消息的 Handle
     */
    class ServerWriterHandler  {
        private boolean done = false;
        private final PrintStream printStream;
        //线程池
        private final ExecutorService executorService;

        public ServerWriterHandler(OutputStream outputStream) {
            this.printStream = new PrintStream(outputStream);
            //单例线程池
            executorService = Executors.newSingleThreadExecutor();
        }

        public void send(String str) {
            //这里如果不用线程池
            executorService.submit(new WriteRunnable(str));
        }

        //线程池的 Runnable
        class WriteRunnable implements Runnable {
            private final String msg;

            public WriteRunnable(String msg) {
                this.msg = msg;
            }

            @Override
            public void run() {
                if(ServerWriterHandler.this.done){
                    return;
                }
                try {
                    ServerWriterHandler.this.printStream.println(msg);
                }catch (Exception e){
                    System.out.println("write 异常退出："+e.getMessage());
                }
            }
        }
        
        public void stopWriter() {
            done = true;
            CloseUtils.close(printStream);
            executorService.shutdownNow();
        }
    }
}

```

完整代码放在 [github](https://github.com/imlgw/socketDemo) 有一点需要注意的是这里用了一个单线程池去处理服务端发送消息的功能，这里其实用线程通信机制`wait/notify`也可以做到但是相比使用线程池会复杂许多。

### 局域网聊天室实现

> 这里的聊天室，其实关键的地方就在于对客户端发送的消息交由服务端进行转发。

基于上面的进行改造

```java
/**
* 回调接口
*/
public interface ClientHandleCallBack {
    /**
    * 将已经关闭的 handle 暴露给 TCPServer 然后从 list 中移除
    */
	void onSelfClosed(ClientHandler clientHandler);

    /**
    * 将消息交给服务器转发
    * @param clientHandler
    * @param msg
    */
	void onNewMessageArrived(ClientHandler clientHandler,String msg);
}
```

增加一个消息抵达的接口，然后为了避免阻塞交给异步的单线程池去处理

```java
	@Override
    public void onNewMessageArrived(ClientHandler clientHandler, String msg) {
        System.out.println("Receive from:"+clientHandler.getClientInfo()+" msg:"+msg);
        forwardThreadPool.submit(()->{
            for (ClientHandler clientHandle : clientHandles) {
                //跳过自己
                if(clientHandle.equals(clientHandler)){
                    continue;
                }
                //对其他客户端发送消息
                clientHandle.send(msg);
            }
        });
    }
```

详细代码 见 [Github](https://github.com/imlgw/socketDemo)
