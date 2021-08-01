---
title: SpringMVC - 处理静态资源 （转）
tags:
  - 技术
  - Spring
  - 转载
categories:
  - Web
date: 2018/5/21
abbrlink: eb5cc3c6
---
## **【1】第一种示例与解决办法**

将 DispatcherServlet 请求映射设置为 / ，将会拦截所有的请求。不能访问静态资源。

**解决办法：**

在 SpringMVC 的配置文件中配置如下标签解决

```java
<mvc:default-servlet-handler/>
```

其 XSD 文档说明如下：

```java
/*配置一个 handler 通过转发请求到 servlet 容器的默认 servlet 来处理静态资源*/
Configures a handler for serving static resources by forwarding to the Servlet container's default Servlet.

/*使用该 handler 将会允许 DispatcherServlet 的 url-pattern 为'/'; 同时使用 servlet 容器的默认 servlet 处理静态资源*/
Use of this handler allows using a "/" mapping with the DispatcherServlet 
while still utilizing the Servlet container to serve static resources. 

/*该 handler 将会转发所有请求到默认 servlet*/
This handler will forward all requests to the default Servlet. 

/*因此将该 handler 的执行顺序放到所有请求处理的最后是非常重要的！！！*/
Therefore it is important that it remains last in the order of all other URL HandlerMappings. 

/*使用<mvc:annotation-driven/>标签或者设置 HandlerMapping instance 的 order 来确保 DefaultServletHttpRequestHandler 的 order 最大。*/
That will be the case if you use the "annotation-driven" element 
or alternatively if you are setting up your customized HandlerMapping instance 
be sure to set its "order" property to a value lower than 
that of the DefaultServletHttpRequestHandler, which is Integer.MAX_VALUE.
```

![image](https://img-blog.csdn.net/20170914152551033?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjA4MDYyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**解释如下：**

`<mvc:default-servlet-handler/>`将在 SpringMVC 的上下文中定义一个 DefaultServletHttpRequestHandler 来处理静态资源（其实就是将请求转发给默认的 servlet)。

一般 WEB 服务器默认的 servlet 的名称为 default。若所使用的 WEB 服务器默认的 Servlet 名称不是 default，则需要通过 default-servlet-name 属性指定！

![image](https://img-blog.csdn.net/20170224143539025?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjA4MDYyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* * *

**不同服务器下默认 servlet 名字对应如下：**

```java
Tomcat, Jetty, JBoss, and GlassFish  默认 Servlet 的名字 -- "default"
Google App Engine 默认 Servlet 的名字 -- "_ah_default"
Resin 默认 Servlet 的名字 -- "resin-file"
WebLogic 默认 Servlet 的名字  -- "FileServlet"
WebSphere  默认 Servlet 的名字 -- "SimpleFileServlet"
```

![这里写图片描述](https://img-blog.csdn.net/20170914154653746?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjA4MDYyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* * *

**【Tips：】**

使用上述配置，你会发现正常的 Controller 跳转失效了！

XSD 说明最后一段话如下：

```java
/*使用<mvc:annotation-driven/>标签或者设置 HandlerMapping instance 的 order 来确保 DefaultServletHttpRequestHandler 的 order 最大。*/
That will be the case if you use the "annotation-driven" element 
or alternatively if you are setting up your customized HandlerMapping instance 
be sure to set its "order" property to a value lower than 
that of the DefaultServletHttpRequestHandler, which is Integer.MAX_VALUE.
```

也就是说，要么配置`<mvc:annotation-driven />`标签，要么手动注册请求映射处理 bean 于 xml 中，并设置 order 属性值，以其实现框架中处理请求映射的 bean 的 order 值小于 DefaultServletHttpRequestHandler 的 order 属性值！！！

常用的解决方式为配置`<mvc:annotation-driven />`标签，详情点击查看 [请求映射失效](http://blog.csdn.net/j080624/article/details/66969987)。

点击查看 [controller 映射失效](http://blog.csdn.net/J080624/article/details/66969987)

* * *

## **【2】第二种示例与解决办法**

解决静态资源的思路是，在 SpringMVC.xml 中，拦截设置为”*.do”，而不是”/”。

这样就不会拦截静态资源的请求。

需要注意的是，如果项目中用到了 shiro 或者其他权限框架。那么需要注意你的 shiro.xml 配置，示例如下：

```java
  <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!-- Shiro 的核心安全接口，这个属性是必须的 -->
        <property name="securityManager" ref="securityManager"/>
        <!-- loginUrl 认证提交地址，如果没有认证将会请求此地址进行认证，请求此地址将由 formAuthenticationFilter 进行表单认证 -->
        <property name="loginUrl" value="/login"/>

        <!-- Shiro 连接约束配置，即过滤链的定义 -->
        <property name="filterChainDefinitions">
            <value>
                <!-- /** = anon 所有 url 都可以匿名访问 -->
                <!-- 对静态资源设置匿名访问 -->
                /images/** = anon
                /js/** = anon
                /styles/** = anon
                <!-- 验证码，可匿名访问 -->
                /validateCode = anon  <!--验证码-->
                /doLogin = anon

                <!-- /** = authc 所有 url 都必须认证通过才可以访问 -->
                /**=authc
                <!--请求 logout，shrio 擦除 sssion-->
                /logout=logout
            </value>
        </property>
    </bean>
```

需要注意的是虽然 SpringMVC 拦截的是。do，但是由于使用了 shiro（或者你的其他权限框架），那么未登录情况下是不能直接访问除 shiro 配置文件里面允许匿名访问的路径之外的静态资源文件。

举个例子，你把静态资源文件放在了项目根目录，但是参考上面配置文件，显然不在匿名访问路径列表之内，所以会提示你先登录，登录之后才可访问项目根目录的静态资源文件。

*   未登录前访问项目根目录下 1.jpg , 跳到登录页面：

![image](https://img-blog.csdn.net/20170518094647021?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjA4MDYyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

*   登录后访问项目根目录下 1.jpg :

![image](https://img-blog.csdn.net/20170518094717006?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjA4MDYyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

另外，建议最好参考 shiro 配置文件，比如 1.jpg 放到 images 文件夹下，那么不用登录就可以直接访问。

![image](https://img-blog.csdn.net/20170518094833869?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjA4MDYyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* * *

## **【3】第三种示例与解决办法**

**通过配置，避免静态资源被拦截，示例如下：**

```java
 <!-- 静态资源访问（不拦截此目录下的东西的访问） -->
    <mvc:resources location="/js/" mapping="/js/**"/>
    <mvc:resources location="/css/" mapping="/css/**"/>
    <mvc:resources location="/images/" mapping="/images/**"/>
    <mvc:resources location="/bootstrap/" mapping="/bootstrap/**"/>
```

该标签的 xsd 说明文档如下：

```
/*配置 handler 为静态资源，如 images，js 和 CSS 文件并进行缓存头优化，以便在 Web 浏览器中高效加载。*/
Configures a handler for serving static resources such as 
images, js, and, css files with cache headers optimized for efficient loading in a web browser. 
/*允许为任何可以通过 spring 处理的路径资源提供服务*/
Allows resources to be served out of any path that is reachable via Spring's Resource handling.
```

注册的 handler 如下：

```java
org.springframework.web.servlet.resource.ResourceHttpRequestHandler
```

**即，该标签注册 ResourceHttpRequestHandler 为静态资源的访问提供服务。**

该 handler 的 javadoc 如下所示：

```java
{@code HttpRequestHandler} that serves static resources in an optimized way according to the guidelines of Page Speed, YSlow, etc.

 * <p>The {@linkplain #setLocations "locations"} property takes a list of Spring
 * {@link Resource} locations from which static resources are allowed to
 * be served by this handler. Resources could be served from a classpath location,
 * e.g. "classpath:/META-INF/public-web-resources/", allowing convenient packaging
 * and serving of resources such as .js, .css, and others in jar files.
```

* * *

## **【4】第四种示例与解决办法**

确切说这里只解决不通过 controller 而直接访问 jsp 的问题。

`<mvc:view-controller/>`直接访问 view-name 对应的 jsp

*   jsp 路径依据视图解析器配置。

```java
    <!-- mvc:view-controller 可使其直接访问路径 -->  
    <mvc:view-controller path="/i18n" view-name="i18n"/>

    <mvc:view-controller path="/i18n2" view-name="i18n2"/>
```
