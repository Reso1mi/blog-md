---
title: 
  CAS单点登陆系统Demo
tags: 
  [CAS,SSO]
categories:
  [Web]
date: 2018/11/17
---
##  单点登陆系统 --CAS
- 关于CAS的介绍网上都有。这里主要记录如何使用，如何配置和集成一些框架。
- CAS架构图
![OSS](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/CAS.PNG)
CAS 的 SSO 实现方式可简化理解为： 1 个 Cookie 和 N 个 Session 。 CAS Server 创建 cookie，在所有应用认证时使用，各应用通过创建各自的 Session 来标识用户是否已登录。
用 户在一个应用验证通过后，以后用户在同一浏览器里访问此应用时，客户端应用中的过滤器会在 session 里读取到用户信息，所以就不会去 CAS Server 认证。如果在此浏览器里访问别的 web 应用时，客户端应用中的过滤器在 session 里读取不到用户信息，就会去 CAS Server 的 login 接口认证，但这时CAS Server 会读取到浏览器传来的 cookie （ TGC ），所以 CAS Server 不会要求用户去登录页面登录，只是会根据 service 参数生成一个 Ticket ，然后再和 web 应用做一个验 证 ticket 的交互而已。

### 1. 配置CAS服务端
从上面的架构图也可以大概知道CAS运作的方式,首先配置好CAS的Server端，直接将CAS的war包拷到tomcat的webapp目录下然后启动tomcat自动解压就可以了，这里我设置的tomcat的端口是8888，地址栏输入localhost:8888/cas能看到CAS的登陆界面说明部署成功
*  去除https
	CAS默认使用的是HTTPS协议，如果使用HTTPS协议需要SSL安全证书（需向特定的机构申请和购买） 。如果对安全要求不高或是在开发测试阶段，可使用HTTP协议。我们这里讲解通过修改配置，让CAS使用HTTP协议。

	* 修改cas的WEB-INF/deployerConfigContext.xml找到下面的配置
<bean class="org.jasig.cas.authentication.handler.support.HttpBasedServiceCredentialsAuthenticationHandler"
p:httpClient-ref="httpClient"/>
这里需要增加参数p:requireSecure="false"，requireSecure属性意思为是否需要安全验证，即HTTPS，false为不采用
   * 修改cas的/WEB-INF/spring-configuration/ticketGrantingTicketCookieGenerator.xml
找到下面配置
    <bean id="ticketGrantingTicketCookieGenerator" class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
      p:cookieSecure="true"
      p:cookieMaxAge="-1"
      p:cookieName="CASTGC"
      p:cookiePath="/cas" />
   * 参数p:cookieSecure="true"，同理为HTTPS验证相关，TRUE为采用HTTPS验证，FALSE为不采用https验证。
参数p:cookieMaxAge="-1"，是COOKIE的最大生命周期，-1为无生命周期，即只在当前打开的窗口有效，关闭或重新打开其它窗口，仍会要求验证。可以根据需要修改为大于0的数字，比如3600等，意思是在3600秒内，打开任意窗口，都不需要验证。
		
		我们这里将cookieSecure改为false , cookieMaxAge 改为3600

   * 修改cas的WEB-INF/spring-configuration/warnCookieGenerator.xml
		<bean id="warnCookieGenerator" class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
		p:cookieSecure="true"
		p:cookieMaxAge="-1"
		p:cookieName="CASPRIVACY"
		p:cookiePath="/cas" />
我们这里将cookieSecure改为false , cookieMaxAge 改为3600
- 配置数据源
   - cas有默认的密码但是实际中肯定是要从数据库中查的，所以我们需要配置下数据源
   - 修改cas服务端中web-inf下deployerConfigContext.xml ，添加如下配置
		```java
		<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" 
		 p:driverClass="com.mysql.jdbc.Driver"
		 p:jdbcUrl="jdbc:mysql://127.0.0.1:3306/pinyougoudb?characterEncoding=utf8"
		 p:user="root"
		 p:password="123456" />
		<bean id="passwordEncoder"
		class="org.jasig.cas.authentication.handler.DefaultPasswordEncoder" 
		 c:encodingAlgorithm="MD5"
		 p:characterEncoding="UTF-8" />
		<bean id="dbAuthHandler" 
		 class="org.jasig.cas.adaptors.jdbc.QueryDatabaseAuthenticationHandler"
		 p:dataSource-ref="dataSource"
		 p:sql="select password from tb_user where username = ?"
		 p:passwordEncoder-ref="passwordEncoder"/>
		```
  - 然后在配置文件开始部分找到如下配置
	
	```java
	 <bean id="authenticationManager" class="org.jasig.cas.authentication.PolicyBasedAuthenticationManager">
	        <constructor-arg>
	            <map>               
	                <entry key-ref="proxyAuthenticationHandler" value-ref="proxyPrincipalResolver" />
	                <entry key-ref="primaryAuthenticationHandler" value-ref="primaryPrincipalResolver" />
	            </map>
	        </constructor-arg>      
	        <property name="authenticationPolicy">
	            <bean class="org.jasig.cas.authentication.AnyAuthenticationPolicy" />
	        </property>
	</bean>
	```
	其中 <entry key-ref="primaryAuthenticationHandler" value-ref="primaryPrincipalResolver" />一句是使用固定的用户名和密码，我们在下面可以看到这两个bean ,如果我们使用数据库认证用户名和密码，需要将这句注释掉。
	添加下面这一句配置
	<entry key-ref="dbAuthHandler" value-ref="primaryPrincipalResolver"/>
  - 将三个jar包加入到cas的lib目录下
   ![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/casjar.PNG)
	第一个和第三个大家应该很熟悉了，中间的那个是cas对jdbc的支持包
	



---
### 2. 普通的web项目集成CAS
 普通的web项目也就是没有使用Spring之类的框架而采用web.xml配置的普通项目
* 2.1 为了方便我这里采用maven配置，先添加相应的依赖
		
```java
        <!-- cas -->
        <dependency>
            <groupId>org.jasig.cas.client</groupId>
            <artifactId>cas-client-core</artifactId>
            <version>3.3.3</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>
```
一个是cas客户端的核心包，一个是servlet的包，因为后面会写一些jsp
然后添加tomcat插件我设置的端口为9002
* 2.2 因为是普通的web项目所以配置的方式主要是通过web.xml来配置，直接上配置

	```java
	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xmlns="http://java.sun.com/xml/ns/javaee"
	         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	         version="2.5">
	    <!-- 用于单点退出，该过滤器用于实现单点登出功能，可选配置 -->
	    <listener>
	        <listener-class>org.jasig.cas.client.session.SingleSignOutHttpSessionListener</listener-class>
	    </listener>
	    <!-- 该过滤器用于实现单点登出功能，可选配置。 -->
	    <filter>
	        <filter-name>CAS Single Sign Out Filter</filter-name>
	        <filter-class>org.jasig.cas.client.session.SingleSignOutFilter</filter-class>
	    </filter>
	    <filter-mapping>
	        <filter-name>CAS Single Sign Out Filter</filter-name>
	        <url-pattern>/* </url-pattern>
	    </filter-mapping>
	    <!-- 该过滤器负责用户的认证工作，必须启用它 -->
	    <filter>
	        <filter-name>CASFilter</filter-name>
	        <filter-class>org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>
	        <init-param>
	            <param-name>casServerLoginUrl</param-name>
	            <param-value>http://localhost:8888/cas/login</param-value>
	            <!--这里的server是服务端的IP -->
	        </init-param>
	        <init-param>
	            <param-name>serverName</param-name>
	            <param-value>http://localhost:9002</param-value>
	        </init-param>
	    </filter>
	    <filter-mapping>
	        <filter-name>CASFilter</filter-name>
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
	    <!-- 该过滤器负责对Ticket的校验工作，必须启用它 -->
	    <filter>
	        <filter-name>CAS Validation Filter</filter-name>
	        <filter-class>org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter</filter-class>
	        <init-param>
	            <param-name>casServerUrlPrefix</param-name>
	            <param-value>http://localhost:8888/cas</param-value>
	        </init-param>
	        <init-param>
	            <param-name>serverName</param-name>
	            <param-value>http://localhost:9002</param-value>
	        </init-param>
	    </filter>
	    <filter-mapping>
	        <filter-name>CAS Validation Filter</filter-name>
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
	    <!-- 该过滤器负责实现HttpServletRequest请求的包裹， 比如允许开发者通过HttpServletRequest的getRemoteUser()方法获得SSO登录用户的登录名，可选配置。 -->
	    <filter>
	        <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
	        <filter-class>
	            org.jasig.cas.client.util.*
	        </filter-class>
	    </filter>
	    <filter-mapping>
	        <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
	    <!-- 该过滤器使得开发者可以通过org.jasig.cas.client.util.AssertionHolder来获取用户的登录名。 比如AssertionHolder.getAssertion().getPrincipal().getName()。 -->
	    <filter>
	        <filter-name>CAS Assertion Thread Local Filter</filter-name>
	        <filter-class>org.jasig.cas.client.util.AssertionThreadLocalFilter</filter-class>
	    </filter>
	    <filter-mapping>
	        <filter-name>CAS Assertion Thread Local Filter</filter-name>
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
	</web-app>

	```

  * 用户认证过滤器(必选)**AuthenticationFilter**在上面的配置中需要两个参数一个是casServerLoginUrl顾名思义就是CASserver登陆的地址，比如我的是http://localhost:8888/cas/login ，第二个参数是**serverName** ,因为登陆成功后还是要返回你当前的应用所以需要将你当前的应用的地址传递给CASserver比如我的当前应用 http://localhost:9002/
  * Ticket的校验过滤器(必选)**Cas20ProxyReceivingTicketValidationFilter** 与上面的过滤器类似也需要那两个参数，							  配置了这两个过滤器CAS就能正常运行了
  * 单点登出过滤器 **SingleSignOutFilter**，顾名思义就是用于单点登出  
  * 另外还有两个过滤器都是为了获取登陆名配置的过滤器，配置一个就行。    
#
* 2.3 编写index.jsp(tomcat默认打开的页面)
	
	```java
	<%@ page language="java" contentType="text/html; charset=utf-8"
	         pageEncoding="utf-8"%>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
	    <title>一品优购</title>
	</head>
	<body>
	 采用web.xml配置的普通的web模块
	<%=request.getRemoteUser()%>
	 <a href="http://localhost:8888/cas/logout?service=http://www.imlgw.top">退出登录</a>>
	</body>
	</html>
	
	```
	 如果想让cas退出后跳转到指定的页面而不是CAS默认的页面也需要修改配置
	 修改cas系统的配置文件cas-servlet.xml
	 <bean id="logoutAction" class="org.jasig.cas.web.flow.LogoutAction"
	 p:servicesManager-ref="servicesManager"
	 p:followServiceRedirects="${cas.logout.followServiceRedirects:true}"/>
	将cas.logout.followServiceRedirects改为true后，可以在退出时跳转页面到目标页面
   然后就可以启动cas和你的应用来测试了。
* 2.4 服务端界面改造
   上面测试成功，但是真实的情况肯定不会用cas默认的那个页面做登陆需要改成你自己的登陆界面，然后你当前应用的登陆页面就没用了。
  
	```java
	<!DOCTYPE >
	<%@ page pageEncoding="UTF-8" %>
	<%@ page contentType="text/html; charset=UTF-8" %>
	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
	<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
	<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
	<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
	<html>
		<head>
			<meta charset="UTF-8" />
			<title>Login</title>
			<link rel="stylesheet" type="text/css" href='bs/css/bootstrap.min.css' />
			<script type="text/javascript" src="js/jquery.min.js"></script>
			<script type="text/javascript" src="bs/js/bootstrap.min.js"></script>
			<style>
				img {
					height: 200px;
					border-radius: 200px;
				}
				.panel-title{
					text-align: center;
				}
				.login {
					width: 500px;
					height: 400px;
					position: absolute;
					top: 20%;
					left: 50%;
					margin-left: -250px;
				}
				.sub{
					text-align: center;
				}
	
			</style>
		</head>
	
		<body>
			<div class="panel panel-primary login">
				<div class="panel-heading">
					<div class="panel-title">
						<img src="images/login.jpg" />
					</div>
				</div>
				<div class="panel-body">
					<!--自定义的登陆界面修改 1 -->
					<form:form method="post" id="fm1" commandName="${commandName}" htmlEscape="true" class="sui-form">
					
						<div class="form-group">
							<div class="input-group">
								<span class="input-group-addon" id="basic-addon1"><span class="glyphicon glyphicon-user"></span></span>
								<!--自定义的登陆界面修改 2 -->
								<form:input  class="form-control" aria-describedby="basic-addon1" placeholder="Username"  id="username" size="25" tabindex="1" accesskey="${userNameAccessKey}" path="username" autocomplete="off" htmlEscape="true" />
							</div>
						</div>
						<div class="form-group">
							<div class="input-group">
								<span class="input-group-addon" id="basic-addon1"><span class="glyphicon glyphicon-lock"></span></span>
								<!--自定义的登陆界面修改 3 -->
								<form:password class="form-control" placeholder="password" aria-describedby="basic-addon1" id="password" size="25" tabindex="2" path="password"  accesskey="${passwordAccessKey}" htmlEscape="true" autocomplete="off" />
							</div>
						</div>
						<div class="form-group sub">
							<!--自定义的登陆界面修改 4 登陆框 -->
							<input type="hidden" name="lt" value="${loginTicket}" />
	     	 				<input type="hidden" name="execution" value="${flowExecutionKey}" />
	      					<input type="hidden" name="_eventId" value="submit" />
	      					<input class="btn btn-primary btn-lg" name="submit" accesskey="l" value="登陆" type="submit" />
	      					<!--自定义的登陆界面修改 4 错误提示框 -->
	      					<form:errors path="*" id="msg" cssClass="errors" element="div" htmlEscape="false" />
						</div>
					</form:form>
				</div>
			</div>
	
		</body>
	
	</html>
	```
	可以直接将登陆的页面拷过来然后加上指令再根据cas的登陆页面稍加修改就可以了

---
### 3. Spring项目集成CAS和Spring-security
* pom依赖
	
	```java
	<properties>
	        <spring.version>4.2.0.RELEASE</spring.version>
	    </properties>
	    <dependencies>
	        <!-- Spring -->
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-context</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-beans</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-webmvc</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-jdbc</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-aspects</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-jms</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-context-support</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <!--spring-security-->
	        <dependency>
	            <groupId>org.springframework.security</groupId>
	            <artifactId>spring-security-web</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.security</groupId>
	            <artifactId>spring-security-config</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>javax.servlet</groupId>
	            <artifactId>javax.servlet-api</artifactId>
	            <version>3.1.0</version>
	            <scope>provided</scope>
	        </dependency>
	        <!--集成包-->
	        <dependency>
	            <groupId>org.springframework.security</groupId>
	            <artifactId>spring-security-cas</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <!--cas-->
	        <dependency>
	            <groupId>org.jasig.cas.client</groupId>
	            <artifactId>cas-client-core</artifactId>
	            <version>3.3.3</version>
	            <exclusions>
	                <exclusion>
	                    <groupId>org.slf4j</groupId>
	                    <artifactId>log4j-over-slf4j</artifactId>
	                </exclusion>
	            </exclusions>
	        </dependency>
	    </dependencies>
	```
	添加tomcat插件指定端口为9001
* web.xml配置
```java
	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xmlns="http://java.sun.com/xml/ns/javaee"
	         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	         version="2.5">
	
	    <context-param>
	        <param-name>contextConfigLocation</param-name>
	        <param-value>classpath:spring/spring-*.xml</param-value>
	    </context-param>
	    <listener>
	        <listener-class>
	            org.springframework.web.context.ContextLoaderListener
	        </listener-class>
	    </listener>
	
	    <filter>
	        <filter-name>springSecurityFilterChain</filter-name>
	        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	    </filter>
	    <filter-mapping>
	        <filter-name>springSecurityFilterChain</filter-name>
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
	
	</web-app>
```
主要配置Spring容器和Spring-security的过滤器
- spring-security.xml配置
  其实就是把之前配置在web.xml里面的过滤器采用Spring的方式配置出来

``` java
	<?xml version="1.0" encoding="UTF-8"?>
	<beans:beans xmlns="http://www.springframework.org/schema/security"
	             xmlns:beans="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
							http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
	
	    <http pattern="/out.html" security="none"></http>
	    <!--   entry-point-ref  入口点引用 因为登陆交给cas或者其他的登陆系统,就不会在本系统做登陆需要指定登陆点-->
	    <http use-expressions="false" entry-point-ref="casProcessingFilterEntryPoint">
	        <intercept-url pattern="/**" access="ROLE_USER"/>
	        <csrf disabled="true"/>
	        <!-- custom-filter为过滤器， position 表示将过滤器放在指定的位置上，before表示放在指定位置之前  ，after表示放在指定的位置之后  -->
	        <custom-filter ref="casAuthenticationFilter"  position="CAS_FILTER" />
	        <custom-filter ref="requestSingleLogoutFilter" before="LOGOUT_FILTER"/>
	        <custom-filter ref="singleLogoutFilter" before="CAS_FILTER"/>
	    </http>
	
	    <!-- CAS入口点 开始 -->
	    <beans:bean id="casProcessingFilterEntryPoint" class="org.springframework.security.cas.web.CasAuthenticationEntryPoint">
	        <!-- 单点登录服务器登录URL -->
	        <beans:property name="loginUrl" value="http://localhost:8888/cas/login"/>
	        <beans:property name="serviceProperties" ref="serviceProperties"/>
	    </beans:bean>
	
	    <beans:bean id="serviceProperties" class="org.springframework.security.cas.ServiceProperties">
	        <!--service 配置自身工程的根地址+/login/cas   -->
	        <beans:property name="service" value="http://localhost:9001/login/cas"/>
	    </beans:bean>
	    <!-- CAS入口点 结束 -->
	    <!-- 认证过滤器 开始 -->
	    <beans:bean id="casAuthenticationFilter" class="org.springframework.security.cas.web.CasAuthenticationFilter">
	        <beans:property name="authenticationManager" ref="authenticationManager"/>
	    </beans:bean>
	    <!-- 认证管理器 -->
	    <authentication-manager alias="authenticationManager">
	        <authentication-provider  ref="casAuthenticationProvider">
	        </authentication-provider>
	    </authentication-manager>
	    <!-- 认证提供者 -->
	    <beans:bean id="casAuthenticationProvider"     class="org.springframework.security.cas.authentication.CasAuthenticationProvider">
	        <beans:property name="authenticationUserDetailsService">
	            <beans:bean class="org.springframework.security.core.userdetails.UserDetailsByNameServiceWrapper">
	                <beans:constructor-arg ref="userDetailsService" />
	            </beans:bean>
	        </beans:property>
	        <beans:property name="serviceProperties" ref="serviceProperties"/>
	        <!-- ticketValidator 为票据验证器 -->
	        <beans:property name="ticketValidator">
	            <beans:bean class="org.jasig.cas.client.validation.Cas20ServiceTicketValidator">
	                <beans:constructor-arg index="0" value="http://localhost:8888/cas"/>
	            </beans:bean>
	        </beans:property>
	        <beans:property name="key" value="an_id_for_this_auth_provider_only"/>
	    </beans:bean>
	    <!-- 认证类 -->
	    <beans:bean id="userDetailsService" class="top.imlgw.demo.UserDetailServiceImpl"/>
	
	    <!-- 认证过滤器 结束 -->
	
	
	    <!-- 单点登出  开始  -->
	    <beans:bean id="singleLogoutFilter" class="org.jasig.cas.client.session.SingleSignOutFilter"/>
	    <!--关联两个地址，相当于封装了前面的地址-->
	    <beans:bean id="requestSingleLogoutFilter" class="org.springframework.security.web.authentication.logout.LogoutFilter">
	        <beans:constructor-arg value="http://localhost:8888/cas/logout?service=http://localhost:9001/out.html"/>
	        <beans:constructor-arg>
	            <beans:bean class="org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler"/>
	        </beans:constructor-arg>
	        <beans:property name="filterProcessesUrl" value="/logout/cas"/>
	    </beans:bean>
	    <!-- 单点登出  结束 -->
	
	</beans:beans>
```

- userDetailsService编写
  这是一个认证类是属于Spring-security的，如果不使用CAS那么这个类就是用来验证密码是否正确是否放行的。但是整合了CAS后就不用在里面做认证了，只是为了返回后面的角色集合。

	```java
	package top.imlgw.demo;
	import org.springframework.security.core.GrantedAuthority;
	import org.springframework.security.core.authority.SimpleGrantedAuthority;
	import org.springframework.security.core.userdetails.User;
	import org.springframework.security.core.userdetails.UserDetails;
	import org.springframework.security.core.userdetails.UserDetailsService;
	import org.springframework.security.core.userdetails.UsernameNotFoundException;
	import java.util.ArrayList;
	import java.util.List;
	
	public class UserDetailServiceImpl implements UserDetailsService {
	    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
	        System.out.println("经过认证类");
	        List<GrantedAuthority> authorities=new ArrayList();
	        //最开始把这里的role写错了，然后cas登陆成功后也一直被403 forbid，因为SpringSecurity认证没通过，一直没放行
	        authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
	        //不通过本项目做登陆 所以密码无所谓，在执行这个方法的时候就已经登陆成功了
	        return new User(username,"",authorities);
	    }
	}
	
	```

---
### 4. 测试
启动搭建好的两个服务器，和casServer，然后测试在一个应用登陆后另一个能否进入index页面 .....

---
SpringBoot集成CAS和Spring-security的后面再补充，因为SpringBoot还不太熟悉。