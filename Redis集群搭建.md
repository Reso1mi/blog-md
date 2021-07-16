---
title: Redis集群搭建
tags:
  - 集群搭建
  - Redis
  - 淘淘商城
categories:
  - Web
date: 2018/5/27
img: 'https://p1.ssl.qhmsg.com/dr/270_500_/t0147083ec93e20e687.png?size=363x363'
abbrlink: 3e4345d9
---
# Redis集群搭建
# 一、Redis简介

### 1．关于关系型数据库和nosql数据库
   关系型数据库是基于关系表的数据库，最终会将数据持久化到[磁盘]()上，而nosql数据  库是基于特殊的结构，并将数据存储到[内存]()的数据库。从性能上而言，nosql数据库  要优于关系型数据库，从安全性上而言关系型数据库要优于nosql数据库，所以在实  际开发中一个项目中nosql和关系型数据库会一起使用，达到性能和安全性的双保证
### 2 . Redis的安装
这里关于Redis的安装不想多说，实际生产中都是将Reids安装在Linux上的，这里主要是说集群的搭建。
### 3. Redis的使用
   我也是第一次接触Redis，使用其实也没什么说的 [菜鸟教程](http://www.runoob.com/redis/redis-tutorial.html)上都有，挺简单的。
### 3. 集群的搭建
Redis要做集群必须要有至少三个节点否则他的投票机制无法运行
必须要有超过半数的节点都认为某一台Redis机器挂掉了才会认为集群挂掉了
![Redis](http://p3.cdn.img9.top/ipfs/QmX5E7BViadysS6bZLs7Eeu1VSkvCHEEcUrkSVgWBjwNCx?3.png)

这里再看看Redis集群的架构图![Redis](http://p3.cdn.img9.top/ipfs/QmZ6PpYPozPHppvXGRBMY5LgnrsjAqDaT2s1NoUCT8EFuD?3.png)
架构细节:

(1)所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽.

(2)节点的fail是通过集群中超过半数的节点检测失效时才生效.

(3)客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可

(4)redis-cluster把所有的物理节点映射到[0-16383]slot上,cluster 负责维护node<->slot<->value
Redis 集群中内置了 16384 个哈希槽，当需要在 Redis 集群中放置一个 key-value 时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点，所以Redis节点最多也就16384个节点最少3个节点。
- 开始搭建
 这里搭建的是有3个节点的最小的集群为保证集群的高可用每个节点都有一台备份机一共需要6台服务器，就需要开6个虚拟机这里其实做测试没那个必要，搭建一个伪分布式的就可以了，在一台虚拟机上搭建6个Redis实例
- 集群搭建环境
1、使用ruby脚本搭建集群。需要ruby的运行环境。
安装ruby
yum install ruby
yum install rubygems
(我虚拟机yum的时候一直报错无法解析，然后加了DNS后报404很奇怪 然后yum makecache后就好了)
2、安装ruby脚本运行使用的包[redis-3.0.0.gem](https://pan.baidu.com/s/1bW6EUyeEevR5h1cT-g51mQ)。
 gem install redis-3.0.0.gem
这个脚本在你Redis源代码的src目录下redis-trib.rb这个脚本
 3、创建6个Redis实例
直接copy5份编译后的Redis文件然后改下端口 把redis.conf(在Redis源代码下copy过来)里 cluster-enabled yes前注释去掉
 3 、启动6个Redis这里可以写一个shell
```java
cd redis01
./redis-server redis.conf
cd ..
cd redis02
./redis-server redis.conf
cd ..
cd redis03
./redis-server redis.conf
cd ..
cd redis04
./redis-server redis.conf
cd ..
cd redis05
./redis-server redis.conf
cd ..
cd redis06
./redis-server redis.conf
cd ..
```
4、用Ruby搭建集群 只需要这条命令集群就搭建完毕了
./redis-trib.rb create --replicas 1 192.168.25.3:7001 192.168.25.3:7002 192.168.25.3:7003 192.168.25.3:7004 192.168.25.3:7005 192.168.25.3:7006
补 ：这里的ip不要写127.0.0.1 不然连接集群的时候就会报Too many Cluster redirections?错误 亲测。。。我不是写的127.0.0.1我是因为后来虚拟机ip变了然后
replicas后面参数1代表每个节点会有一台备份机所以后面的ip和端口号必须是偶数
### 4. 集群的使用
    redis01/redis-cli -p 7002 -c 
后面的c代表连接的是集群 如果不加c就是单机的 如果存入数据的槽位不对应就会报错当然这是在命令行下的使用，windows下也有一些Redis的客户端。这里主要讲用Java代码连接Redis这就要用到Jedis了跟JDBC那一套差不多 这里直接讲在淘淘中实际是怎么用的吧
这里有两个版本集群版和单机版 平常测试就用单机版所以可以先写一个接口，抽取一些常用的方法

```java
package redis;
public interface JedisClient {

	String set(String key, String value);
	String get(String key);
	Boolean exists(String key);
	Long expire(String key, int seconds);
	Long ttl(String key);
	Long incr(String key);
	Long hset(String key, String field, String value);
	String hget(String key, String field);
	Long hdel(String key, String... field);
}

```
然后写各自的实现类
- 单机版

```java
package redis;
import lombok.Getter;
import lombok.Setter;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

@Getter@Setter
public class JedisClientPool implements JedisClient {
	
	private JedisPool jedisPool;

	@Override
	public String set(String key, String value) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.set(key, value);
		jedis.close();
		return result;
	}

	@Override
	public String get(String key) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.get(key);
		jedis.close();
		return result;
	}

	@Override
	public Boolean exists(String key) {
		Jedis jedis = jedisPool.getResource();
		Boolean result = jedis.exists(key);
		jedis.close();
		return result;
	}

	@Override
	public Long expire(String key, int seconds) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.expire(key, seconds);
		jedis.close();
		return result;
	}

	@Override
	public Long ttl(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.ttl(key);
		jedis.close();
		return result;
	}

	@Override
	public Long incr(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.incr(key);
		jedis.close();
		return result;
	}

	@Override
	public Long hset(String key, String field, String value) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hset(key, field, value);
		jedis.close();
		return result;
	}

	@Override
	public String hget(String key, String field) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.hget(key, field);
		jedis.close();
		return result;
	}

	@Override
	public Long hdel(String key, String... field) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hdel(key, field);
		jedis.close();
		return result;
	}

}

```
- 集群版

```java
package redis;
import lombok.Getter;
import lombok.Setter;
import redis.clients.jedis.JedisCluster;

@Getter@Setter
public class JedisClientCluster implements JedisClient {
	
	private JedisCluster jedisCluster;

	@Override
	public String set(String key, String value) {
		return jedisCluster.set(key, value);
	}

	@Override
	public String get(String key) {
		return jedisCluster.get(key);
	}

	@Override
	public Boolean exists(String key) {
		return jedisCluster.exists(key);
	}

	@Override
	public Long expire(String key, int seconds) {
		return jedisCluster.expire(key, seconds);
	}

	@Override
	public Long ttl(String key) {
		return jedisCluster.ttl(key);
	}

	@Override
	public Long incr(String key) {
		return jedisCluster.incr(key);
	}

	@Override
	public Long hset(String key, String field, String value) {
		return jedisCluster.hset(key, field, value);
	}

	@Override
	public String hget(String key, String field) {
		return jedisCluster.hget(key, field);
	}

	@Override
	public Long hdel(String key, String... field) {
		return jedisCluster.hdel(key, field);
	}

}

```
- 使用
   使用的时候就可以写个bean然后在需要的时候就可以直接注入了

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xsi:schemaLocation="http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
		http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.2.xsd">
	<!--单机版-->
    <!-- 	<bean id="jedisClientPool" class="redis.JedisClientPool">
		<property name="jedisPool" ref="jedisPool" />
	</bean>
	<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
		<constructor-arg name="host" value="192.168.25.4"/>
		<constructor-arg name="port" value="6379"/>
	</bean> -->
	<!--集群版-->
	<bean id="jedisClientCluster" class="redis.JedisClientCluster">
		<property name="jedisCluster" ref="jedisCluster"/>		
	</bean>
	
	<bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
		<constructor-arg name="nodes">
			<set>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.4"/>
					<constructor-arg name="port" value="7001"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.4"/>
					<constructor-arg name="port" value="7002"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.4"/>
					<constructor-arg name="port" value="7003"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.4"/>
					<constructor-arg name="port" value="7004"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.4"/>
					<constructor-arg name="port" value="7005"/>
				</bean>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg name="host" value="192.168.25.4"/>
					<constructor-arg name="port" value="7006"/>
				</bean>
			</set>
		</constructor-arg>			
	</bean>
</beans>
```
这里在集群的时候遇到了问题就是我上面提到的ip的问题不要写127.0.0.1要写实际的内网ip以后如果ip变了就只能重新来一次了，那个命令我试了下执行第二次，报错说我那个节点不为空，**也许**把数据清空了就可以了。
- 测试
```java
	@Test
	public void testRedis() {
		//初始化Spring容器
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-redis.xml");
		//从容器中获取jedisClient对象 (单机版) 
		JedisClient jedisClient=applicationContext.getBean(JedisClient.class);
		jedisClient.set("testRedis", "Hello World");
		System.out.println(jedisClient.get("testRedis"));
	}
```

