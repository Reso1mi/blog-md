---
title: Spring-Security 遇到的小问题
tags:
  - Bug
  - Spring
categories:
  - 踩坑记录
date: 2018/11/26
abbrlink: 2dc72c90
---
## Spring Security 配置 intercept-url 的小问题

文档： http://www.fengfly.com/document/springsecurity3/core-web-filters.html

```java
<bean id="filterInvocationInterceptor"
     class="org.springframework.security.intercept.web.FilterSecurityInterceptor">
  <property name="authenticationManager" ref="authenticationManager"/>
  <property name="accessDecisionManager" ref="accessDecisionManager"/>
  <property name="runAsManager" ref="runAsManager"/>
  <property name="securityMetadataSource">
    <security:filter-security-metadata-source path-type="regex">
      <security:intercept-url pattern="\A/secure/super/.*\Z" access="ROLE_WE_DONT_HAVE"/>
      <security:intercept-url pattern="\A/secure/.*\" access="ROLE_SUPERVISOR,ROLE_TELLER"/>
    </security:filter-security-metadata-source>
  </property>
</bean>        
```

模式总是根据他们定义的顺序进行执行。因此很重要的是，把更确定的模式定义到列表的上面。 这会反映在你上面的例子中，更确定的/secure/super/模式放在没那么确定的 /secure/模式的上面。如果它们被反转了。/secure/会一直
 被匹配，/secure/super/就永远也不会执行。

----

```java
<http use-expressions="false" entry-point-ref="casProcessingFilterEntryPoint">
		<intercept-url pattern="/**" access="ROLE_USER"/>
        <!--经过 SpringSecurity 的环境上下文，偶然发现一个小坑-->
        <intercept-url pattern="/cart/*.do" access="IS_AUTHENTICATED_ANONYMOUSLY"/>
        <csrf disabled="true"/>
        <!-- custom-filter 为过滤器， position 表示将过滤器放在指定的位置上，before 表示放在指定位置之前  ，after 表示放在指定的位置之后  -->
        <custom-filter ref="casAuthenticationFilter"  position="CAS_FILTER" />
        <custom-filter ref="requestSingleLogoutFilter" before="LOGOUT_FILTER"/>
        <custom-filter ref="singleLogoutFilter" before="CAS_FILTER"/>
    </http>
```
一个小问题，就是大范围在前面会覆盖后面的小范围所以像我上面那么写/cart/*.do 就不会执行相当于没写，起不到作用。