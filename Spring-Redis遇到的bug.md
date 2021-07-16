---
title: Spring-Redis遇到的bug
tags:
  - Bug
  - Redis
  - Spring
categories:
  - 踩坑记录
date: 2018/9/24
abbrlink: bea4831e
---

##   两个小bug记录一下
1.  Spring-data-redis和jedis整合的版本问题报错如下：
```java
严重: Exception sending context initialized event to listener instance of class org.springframework.web.context.ContextLoaderListener
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'redisTemplate' defined in URL [jar:file:/E:/repository/com/pyg/pyg-common/0.0.1-SNAPSHOT/pyg-common-0.0.1-SNAPSHOT.jar!/spring/applicationContext-redis.xml]: Invocation of init method failed; nested exception is java.lang.NoSuchMethodError: org.springframework.core.serializer.support.DeserializingConverter.<init>(Ljava/lang/ClassLoader;)V
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1578)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:545)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:482)
at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:305)
at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:301)
at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:196)
at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:772)
at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:834)
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:537)
at org.springframework.web.context.ContextLoader.configureAndRefreshWebApplicationContext(ContextLoader.java:446)
at org.springframework.web.context.ContextLoader.initWebApplicationContext(ContextLoader.java:328)
at org.springframework.web.context.ContextLoaderListener.contextInitialized(ContextLoaderListener.java:107)
at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4939)
at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5434)
at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1559)
at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1549)
at java.util.concurrent.FutureTask.run(FutureTask.java:266)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at java.lang.Thread.run(Thread.java:748)
Caused by: java.lang.NoSuchMethodError: org.springframework.core.serializer.support.DeserializingConverter.<init>(Ljava/lang/ClassLoader;)V
at org.springframework.data.redis.serializer.JdkSerializationRedisSerializer.<init>(JdkSerializationRedisSerializer.java:53)
at org.springframework.data.redis.core.RedisTemplate.afterPropertiesSet(RedisTemplate.java:117)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1637)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1574)
... 21 more
```

 报错原因：Spring-data-reids 和 jedis的版本冲突（应该也和Spring的版本有关系,别人跟我一样的包都可以运行）。
我测试成功的版本：
<jedis.version>2.6.2</jedis.version>
<spring.version>4.2.0.RELEASE</spring.version>
<spring-data-redis.version>1.4.2.RELEASE</spring.version>

2. Spring加载配置文件的问题
因为同时配置了MySql和Redis的配置文件而且不是同一个工程所以不是同时初始化然后出现了以下的问题。
同个模块中如果出现多个context:property-placeholder ，location properties文件后， 运行时出现Could not resolve placeholder 'key' in string value${key}。原因是在加载第一个context:property-placeholder时 会扫描所有的bean，而有的bean里面出现第二个 context:property-placeholder引入的properties的占位符${key}， 此时还没有加载第二个property-placeholder，所以解析不了${key}。
解决办法一，可以将通过模块的多个property-placeholder合并为一个，将初始化放在一起。
方法二，添加ignore-unresolvable="true "，这样可以在加载第一个property-placeholder时出现解析不了的占位符进行忽略掉

ps：目前的计划是先把品优购这个项目做完，然后搞一搞微信小程序之类的开发，然后开始着手研究源码，对Spring的源码非常好奇有种特别想了解她的欲望hahahahahaha....  看源码肯定会涉及到设计模式也顺便学一学，再就是数据结构和算法，这学期也正在学也不用着急跟着老师的进度再加自己的自学补充，这个东西也不是说今天学完了就会了的东西，大二上学期主要的打算就是这个了，还有就是不能挂科！！！！,再往后就是打算学下安卓和安卓逆向，不过暂时只是个打算😄。再往后就是大三了，还没打算到那个时候，想想还真快，不知不觉就一年过去了。Come on , add oil ! ! ! @imlgw

