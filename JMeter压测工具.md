---
title: 
  JMeter压测
tags: 
  [JMeter,秒杀]
categories:
  [Web]
date: 2019/6/6
cover: http://static.imlgw.top/image/20190713/Blog-Built-in-JMeter-Functions.png
---

## JMeter入门

[官网地址](http://jmeter.apache.org/) 下载好之后直接运行jar包 

### 简单上手

**添加线程组**

![mark](http://static.imlgw.top/image/20190529/wBWwVD5chqtc.png?imageslim)

**设置线程个数和配置**

![mark](http://static.imlgw.top/image/20190529/KonWE6pHVdMz.png?imageslim)

Ramp-Up就是多长时间内启动这些线程设置位0就是同时启动。

**设置HTTP请求默认值**

![mark](http://static.imlgw.top/image/20190529/73A4su1jMAWM.png?imageslim)

设置好后再添加具体的请求的时候就不用再写这个了

![mark](http://static.imlgw.top/image/20190529/Uwy5zrWzFsji.png?imageslim)

**添加HTTP请求**

![mark](http://static.imlgw.top/image/20190529/iVbmgkdG7D88.png?imageslim)

这里对我们的秒杀商品列表进行压测。

**添加监听器**

![mark](http://static.imlgw.top/image/20190529/yVm6imNhWYKe.png?imageslim)

这里添加比较常用的聚合报告就可以了

**结果**

![mark](http://static.imlgw.top/image/20190531/C3pwAwyh7BhG.png?imageslim)

这里我们可以需要关注的就是吞吐量这个参数，一开始可能会不太准多测几次。

同时我们也可以用Linux的`top`命令查看当前CPU的利用率。

### 添加自定义参数

![mark](http://static.imlgw.top/image/20190601/VQ9adD8KBfvA.png?imageslim)

![mark](http://static.imlgw.top/image/20190601/j1XB0mewqvJG.png?imageslim)

```java
17362363659,3d3ae96d381d4376b87cb7ebf14aadb6
12012341234,fbb11e35f16b4a54be1315a0a1619193
11012341234,58d63f1d9482472f907829da2ae3b4ff
10012341234,09fe09587b924c49b6db64f763c1ad10
```

**效果**

![mark](http://static.imlgw.top/image/20190601/5up85aDbSTWM.png?imageslim)

### 生成token

方便后面的压测，可以直接写一个工具类生成token供后面的redis使用

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class TokenUtils {

    @Autowired
    private SpikeUserService spikeUserService;

    static String str="0123456789";

    static  Random random = new Random();

    static HttpServletResponse resp;

    @Test
    public  void test() throws IOException {
        FileOutputStream outputStream=new FileOutputStream(new File("D:\\AliyunKey\\config.txt"));
        for (int i=0;i<20000;i++){
            String phone = creatPhone();
            RegisterVo registerVo = new RegisterVo(phone,"123456","user-"+i);
            spikeUserService.register(registerVo);
            //需要在service层token返回出来
            String token= spikeUserService.login(resp, new LoginVo(registerVo.getMobile(), registerVo.getPassword()));
            outputStream.write((phone+","+token+"\n").getBytes());
        }
    }

    public  String creatPhone(){
        String res="1";
        for (int i=0;i<10;i++){
            res+=str.charAt(random.nextInt(10));
        }
        return res;
    }
}
```

### Redis压测

①redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000

100个并发，十万个请求，对常用的一些命令进行测试

![mark](http://static.imlgw.top/image/20190601/oPkioXCFgL8R.png?imageslim)

②redis-benchmark -h 127.0.0.1 -p 6379 -q -d 100

100 bytes payload 

-q 是quiet输出信息较少

③ redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"

对特定的语句压测



##  搭建压测环境

### 命令行压测

其实还是需要借助图形界面来录好jmx文件然后上传到Linux上，然后执行

sh jmeter.sh -n -t Xxx.jmx -l result.jtl

然后再用图形界面导入result.jtl就可以看到结果

> 上面的测试都是在我的开发机(win)上进行的，压测和服务都在本地，结果可能并不准确，这里为了隔离环境我开了了2个虚拟机，一个是部署服务的机器（2G 4核），一个是部署mysql和redis的机器（2G 4核），这里在Linux上运行部署项目有两种方式，一种是打成war包放在tomcat目录下，一种是打成jar包直接运行。

### 环境

✔ 192.168.25.123   Centos6  mysql+redis  2G4核

✔ 192.168.25.4     Centos7   SpikeServer+压测  2.5G 4核

✔~~win10开发机     Jmeter压测 SpikeServer~~ 

✔~~192.168.25.129 Centos7 压测设备 2G4核~~

### SpringBoot打war包

**添加tomcat依赖(编译时依赖)**

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
	<scope>provided</scope>
</dependency>
```

**添加一个maven插件**

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
	<configuration>
    	<failOnMissingWebXml>false</failOnMissingWebXml>
	</configuration>
</plugin>
```

**修改pom打包方式位war**

```java
<packaging>war</packaging>
```

**boot类添加一个方法**

```java
@SpringBootApplication
public class SpikeApplication extends SpringBootServletInitializer {
    public static void main(String[] args) {
        SpringApplication.run(SpikeApplication.class, args);
    }
    /**
     * @param builder
     * @return 打 war包
     */
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(SpikeApplication.class);
    }
}
```

然后在项目目录下执行`mvn clean package`就会在target目录下生成war包，然后将war包拷到tomcat里面就可以直接运行了。

### SpringBoot打jar包

**pom里的打包方式改为jar(默认就是jar)**

```java
<packaging>war</packaging>
```

**添加一个maven插件**

```java
<!--打jar包的插件-->
<plugin>
    <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

**执行mvn clean package**

同上会在target目录下生成一个jar包，jar包内容大致如下

```java
Manifest-Version: 1.0
Implementation-Title: Spike
Implementation-Version: 1.0-SNAPSHOT
Built-By: priva
Implementation-Vendor-Id: top.imlgw
Spring-Boot-Version: 2.1.2.RELEASE
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: top.imlgw.spike.SpikeApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Created-By: Apache Maven 3.5.3
Build-Jdk: 1.8.0_172
Implementation-URL: https://projects.spring.io/spring-boot/#/spring-bo
 ot-starter-parent/Spike
```

如果确少一些信息比如Main-Class和Start-Class，说明jar包打的有问题，运行会报`没有主清单属性`，我一开始没注意，我的`plugins`上层还有个`pluginmanagement`插件根本没加载进来，去掉就行了。

## 开始压测

### 压测商品列表页面

```java
@RequestMapping("/to_list")
public String tolist(Model model,SpikeUser spikeUser) {
    List<GoodsVo> goodsVos = goodsService.goodsVoList();
    model.addAttribute("user", spikeUser);
    model.addAttribute("goodsList",goodsVos);
	return "goods_list";
}
```

这个接口主要就做了一个查询的mysql的操作，没有cookie所以不会去操作redis

### 遇到的问题

一开始直接设置了 5000*10 的并发，然后服务端报了   打开文件过多的错误

```java
java.io.IOException: 打开的文件过多
        at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method) ~[na:1.8.0_171]
        at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422) ~[na:1.8.0_171]
        at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250) ~[na:1.8.0_171]
        at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:448) ~[tomcat-embed-core-9.0.14.jar!/:9.0.14]
        at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:70) ~[tomcat-embed-core-9.0.14.jar!/:9.0.14]
        at org.apache.tomcat.util.net.Acceptor.run(Acceptor.java:95) ~[tomcat-embed-core-9.0.14.jar!/:9.0.14]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_171]
```

google后发现是句柄太少的原因，Linux默认是1024，而我们同时起了5000个线程自然就出问题了。

通过`ulimit -a` 可以查看到当前的最大句柄数`open files` ，这里我们可以通过 `ulimit -n 2048`临时的设置一个较大的值，但是重启后就会失效。

```java
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 14707
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 14707
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

这里最好是直接修改  /etc/security/limits.conf

```java
*  soft nofile  32768
*  hard nofile 65536
```

就可以将文件句柄限制统一改成软32768，硬65536。配置文件最前面的是指domain，设置为星号代表全局，另外你也可以针对不同的用户做出不同的限制。

注意：这个当中的硬限制是实际的限制，而软限制，是warnning限制，只会做出warning，其实ulimit命令本身就有分软硬设置，加-H就是硬，加-S就是软

修改后记得重启才会生效 [参考资料](https://www.cnblogs.com/ibook360/archive/2012/05/11/2495405.html) 

一开始是打算直接用win开发机做压测的，但是发现在进程开大了之后老是跑不完，跑一半就停了（可能是内存给小了），而且数据出入也比较大，然后改用秒杀服务的那条机器来压测，一开始只增大了跑秒杀服务的虚拟机，发现还是会有异常，然后我把mysql和redis的虚拟机也调大了就没报异常了，但是在压测的时候秒杀服务的虚拟机cpu飙到了 `9.0+`，4核的机子，cpu飙到这么高就有点问题了一般来说应该维持在 `4*0.7` 左右。

![mark](http://static.imlgw.top/image/20190604/LedMcRvfSXeb.png?imageslim)

![mark](http://static.imlgw.top/image/20190604/UbcTCaTGiEJ9.png?imageslim)

可以看到平均等待时间都在2s以上。

为了更准确的模拟，我又开了一台~~1G2核~~ 2G4核的虚拟机专门来做压测（8G内存吃不消了）。

![mark](http://static.imlgw.top/image/20190604/pr4yMAE2Jamf.png?imageslim)

这里用top观察了两台虚拟机的情况发现mysql的那台机器负载一直很低，SpikeServer那台机器（1G双核）负载一路飙到6.0+。吞吐率也明显的下降了，这里我连续测试了两次都是400多。

### 最终配置

经过一上午的折腾，我决定还是值利用两台你虚拟机，一台跑SpikeServer和压测，另外一台跑mysql和redis，再启动一台成本太大了，这里主要根据这个做一个标准量，后期优化后拿来对比

#### 结果

![mark](http://static.imlgw.top/image/20190604/IEGxtWU8pujI.png?imageslim)

后面在调整机器或连续测试了5，6次 在5000的并发下QPS大概是1000左右的样子，小于1000。

### 压测Redis查询的性能

上面的goods_list实际上只对mysql进行了一个查询操作，而mysql的并发量并不大。

下面我们单独对redis做一下压测，看下系统的QPS（这里）

![mark](http://static.imlgw.top/image/20190604/h2NOJszwC7Lo.png?imageslim)

#### 结果

一开测试忘了调大redis链接池的大小，一直跑不出来，后来改大之后测了4，5次，同样的5000并发10次，QPS大概在3000左右

![mark](http://static.imlgw.top/image/20190606/58L5ieKaxR7a.png?imageslim)

可以说是相当快了，而且`top`观察redis那台机器发现负载依然很低，说明这点并发确实对redis来说是小意思，前面其实也单独对redis用它自带的压测工具测试过，大概每秒10 0000的GET是没问题的

### 重头戏—压测do_spike接口

```java
 @RequestMapping("/do_spike")
 public String do_spike(Model model, SpikeUser spikeUser, @RequestParam("goodsId") long goodsId) {
        if (spikeUser==null) { //没有登录
            return "login";
        }
        //检查库存
        GoodsVo goodsVo= goodsService.getGoodsVoByGoodsId(goodsId);
        int stock=goodsVo.getStockCount(); //这里拿的秒杀商品里面的库存,不是商品里面的库存
        if(stock<=0){
            model.addAttribute("failMsg",CodeMsg.STOCK_EMPTY);
            return "spike_fail";
        }
        //看是否重复秒杀
        SpikeOrder spikeOrder=spikeService.getGoodsVoByUserIdAndGoodsId(spikeUser.getId(), goodsId);
        if(spikeOrder!=null){
            model.addAttribute("failMsg",CodeMsg.SKIPE_REPEAT);
            return "spike_fail";
        }
        OrderInfo orderInfo=spikeService.doSpike(spikeUser.getId(),goodsVo);
        model.addAttribute("orderInfo",orderInfo);
        model.addAttribute("goods",goodsVo);
        return "order_detail";
 }
```

步骤都跟上面一样，不过要多加一个商品id的参数，这里依然是5000的并发10次，其实这里测出来的结果和上面的商品列表差不太多，差不多950左右QPS，毕竟这里有判断库存的操作，一旦小于0之后就不会对mysql再进行操作，进行复杂的**减库存**和**生成订单**操作

#### 超卖问题

本来只有10件商品，硬生生给减成了负数😂

![mark](http://static.imlgw.top/image/20190606/m5RhIzmdXRV1.png?imageslim)

可以看到有16个人秒杀到了这个商品这显然是不合理的

![mark](http://static.imlgw.top/image/20190606/nJmnvKvVzAHt.png?imageslim)

这个问题会在后面的文章中提出解决方案，这一篇主要熟悉下压测。