---
title: 
  - dubbo+zookeeper简单实现分布式
tags:
   [分布式,淘淘商城]
categories:
   [Web]
date: 2018/5/20 9:00
---
# 分布式
- 其实我也是在做淘淘的时候才接触到分布式，也只是简单的了解了下，关于分布式的好处也不多说了.
- 实际应用
这里实际应用就是淘淘商城 由于淘淘商城是基于soa(面向服务)的架构，表现层和服务层是不同的工程。所以要实现商品列表查询需要两个系统之间进行通信。
  ps: 在淘淘里面用的是dubbo+zookeeper这只是远程通信的一种方式常见的有三种
  1.Webservice：效率不高基于soap协议。项目中不推荐使用。
  2.使用restful形式的服务：http+json。很多项目中应用。如果服务太多，服务之间调用关系混乱，需要治疗服务。
  3.使用dubbo。使用rpc协议进行远程调用，直接使用socket通信。传输效率高，并且可以统计出系统之间的调用关系、调用次数。
- dubbo
  Dubbo就是**资源调度和治理中心**的管理工具。
 dubbo的架构
  ![image](http://p0.cdn.img9.top/ipfs/QmNt5aeNWC6PSPY6Dak7p39H1tHY6y9u6ppxhNFoh8PUFn?0.png)
**节点角色说明：**

  **Provider:** 暴露服务的服务提供方。

  **Consumer:** 调用远程服务的服务消费方。

  **Registry:** 服务注册与发现的注册中心。

  **Monitor:** 统计服务的调用次调和调用时间的监控中心。

  **Container:** 服务运行容器
**流程分析：**
  0\. 服务容器负责启动，加载，运行服务提供者。

  1\. 服务提供者在启动时，向注册中心注册自己提供的服务。

  2\. 服务消费者在启动时，向注册中心订阅自己所需的服务。

  3\. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

  4\. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

  5\. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。
**如何使用**
 Dubbo采用全Spring配置方式，透明化接入应用，对应用没有任何API侵入，只需用Spring加载Dubbo的配置即可，Dubbo基于Spring的Schema扩展进行加载。

  **单一工程中spring的配置**
```java
 <bean id="xxxService" class="com.xxx.XxxServiceImpl" />
  <bean id="xxxAction" class="com.xxx.XxxAction">
  <property name="xxxService" ref="xxxService" />
  </bean>
```
**远程服务：**

  在本地服务的基础上，只需做简单配置，即可完成远程化：
  将上面的local.xml配置拆分成两份，将服务定义部分放在服务提供方remote-provider.xml，将服务引用部    分放在服务消费方remote-consumer.xml。
  并在提供方增加暴露服务配置<dubbo:service>，在消费方增加引用服务配置<dubbo:reference>。

  发布服务：

```java
<!-- 和本地服务一样实现远程服务 -->

  <bean id="xxxService" class="com.xxx.XxxServiceImpl" />

  <!-- 增加暴露远程服务配置 -->

  <dubbo:service interface="com.xxx.XxxService" ref="xxxService" />

   调用服务：
<!-- 增加引用远程服务配置 -->

  <dubbo:reference id="xxxService" interface="com.xxx.XxxService" />

  <!-- 和本地服务一样使用远程服务 -->
  <bean id="xxxAction" class="com.xxx.XxxAction">
  <property name="xxxService" ref="xxxService" />
  </bean>

```

- 淘淘中的dubbo
![image](http://p1.cdn.img9.top/ipfs/QmRPvtoW2JoEZKuXFExYrusKhDt7kM5iSk4JWeXHBLfMVA?1.png)
简单解释下
   上面的springmvc是web层的配置下面的service是服务层的配置
**service层发布服务**
```java
<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181" />
```
这个是注册中心 淘淘采用的是zookeeper address就是zookeeper的地址我的zookeper安装在本机上所以就是127.0.0.1后面的是zookeper暴露的端口号默认2181  首先在service层发布服务发布的时候指明接口类和 实现类 这里*实现类的首字母要小写*或者你也可以自己定义一个bean然后指定id
*<!-- 声明需要暴露的服务接口 -->
	<dubbo:service interface="com.taoshop.service.ItemService"
		ref="itemServiceImpl" />*
*<!-- 用dubbo协议在20880端口暴露服务 -->
	<dubbo:protocol name="dubbo" port="20880" />*
这个就是暴露端口的服务 这里还要注意一点就是有多个应用发布服务这里端口需要改。  
**服务发布web层引用服务**
```java
<dubbo:reference interface="com.taoshop.service.ItemService" id="itemService" />
```
这样就完成了服务的发布和引用然后就可以启动项目测试了 这里还要注意启动的顺序
先启动zookeeper --> 再启动service层 -->再启动web 层 主要是要先启动zookeepe
- 最后贴一下maven依赖吧
```java
<!--Dubbo -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>dubbo</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework</groupId>
					<artifactId>spring</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.jboss.netty</groupId>
					<artifactId>netty</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
		</dependency>
```
- 关于分布式确实还了解的不够，远程通讯webservice也还没学后面要慢慢花点时间来学习下 还是挺有意思的