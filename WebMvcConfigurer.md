---
title:
  WebMvcConfigurer小结
tags:
  [SpringMVC]
categories:
	[Web]
date: 2019/5/20
cover: http://static.imlgw.top/image/20190713/8b2b25e9-a8de-b3f4-a810-7814a40a193c.png
---

## 起源

利用Redis做分布式session，因为没有借助Spring-session或者其他的session共享方案，手动处理session的存取，在控制层获取cookie中的数据是较为麻烦，所以希望直接将cookie的数据转化为需要的bean然后绑定到参数中，这里就可以借助**WebMvcConfigurer**来实现这个需求简化代码

## WebMvcConfigurer是干嘛的？

Spring把实现了WebMvcConfigurer接口的bean都看作为SpringMvc的**扩展配置**，如果既想要使用SpringBoot对SpringMvc的自动配置，又想要对自动配置进行扩展，添加一些用户自己的配置，像拦截器，消息转换器或者下文中的参数绑定，只需要写一个实现了WebMvcConfigurer接口的配置类，实现相关方法就能够添加自己的配置了。

> SpringBoot2.0之前也就是Spring5之前可以直接继承**WebMvcConfigurationAdapter**+**@EnableWebMvc**注解来实现上述需求，但是这个方法在之后的版本中弃用了(还可以用但是不太好)，因为jdk8之后的接口中可以有**默认方法**了，所以这个抽象类就并没有存在的意义了

### WebMvcConfigurationSupport

其实还有一种方法就是直接继承这个WebMvcConfigurationSupport，上面的WebMvcConfigurer只是扩展配置，如果直接继承WebMvcConfigurationSupport，那么就可以重写默认的配置，如果对原理不是很清楚的开发者不小心重写错了默认的配置，springmvc可能相关功能就无法生效。

## WebMvcConfigurer内的方法

```java
public interface WebMvcConfigurer {
    default void configurePathMatch(PathMatchConfigurer configurer) {
    }

    default void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    }

    default void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    }

    default void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    }

    default void addFormatters(FormatterRegistry registry) {
    }
	
    //添加拦截器
    default void addInterceptors(InterceptorRegistry registry) {
    }
	//添加资源处理器
    default void addResourceHandlers(ResourceHandlerRegistry registry) {
    }

    default void addCorsMappings(CorsRegistry registry) {
    }
	//视图控制器
    default void addViewControllers(ViewControllerRegistry registry) {
    }

    default void configureViewResolvers(ViewResolverRegistry registry) {
    }
	//添加参数解析器
    default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    }

    default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {
    }
	//消息转换器
    default void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    }

    default void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
    }

    default void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
    }

    default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
    }

    @Nullable
    default Validator getValidator() {
        return null;
    }

    @Nullable
    default MessageCodesResolver getMessageCodesResolver() {
        return null;
    }
}
```

## 注意事项

在**SpringBoot**下自定义的WebMvcConfigurer实现配置类上是不需要添加**@EnableWebMvc**的，因为**SpringBoot**已经实例化了WebMvcConfigurationSupport，如果添加了该注解，默认的WebMvcConfigurationSupport配置类就会失效，mvc默认的配置会失效，也就是以用户定义的为主，一般建议还是不覆盖默认的好。

这点可以从SpringBoot的**WebMvcAutoConfiguration**中看到。（@EnableWebMvc会导入一个WebMvcConfigurationSupport的子类，叫 DelegatingWebMvcConfiguration）。

![mark](http://static.imlgw.top///20190520/p5I6RuWHJPMy.png?imageslim)

当没有WebMvcConfigurationSupport的时候自动配置才会生效

> [官方文档](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-auto-configuration)
>
> If you want to keep Spring Boot MVC features and you want to add additional [MVC configuration](https://docs.spring.io/spring/docs/5.0.4.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, you can declare a `WebMvcRegistrationsAdapter` instance to provide such components.

## 实例

### Controller参数绑定

在SpringMVC里面可以轻松的把表单的数据映射到对应的bean中，但是有时候这并不能满足我们的需求，比如下面的例子。

```java
@RequestMapping("/to_list")
public String tolist(HttpServletResponse response, Model model,
                         @CookieValue(value = SpikeUserService.COOK1_NAME_TOKEN, required = false) String cookie,
                         @RequestParam(value = SpikeUserService.COOK1_NAME_TOKEN, required = false) String param) {
        /*手机浏览器，有可能将cookie放在参数中*/
        if (param == null && cookie == null) {
            return "login";
        }
        String cook = cookie != null ? cookie : param;
        SpikeUser user = spikeUserService.getUserByToken(response, cook);
        System.out.println(user);
        model.addAttribute("user", user);
        return "goods_list";
 }
```

可以看到这里为了获取这个**SpikeUser**对象并不能直接从表单中获取，需要借助cookie然后从redis里面查询，如果下面还有一些其他的controller需要获取这个对象，又要写很多重复的代码。这个时候我们就可以通过上面介绍的WebMvcConfigurer来实现简化代码。

####  重写addArgumentResolvers

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Autowired //注入我们的参数解析器
    SpikeUserArgumentResolver spikeUserArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(spikeUserArgumentResolver);
    }
}
```

这里只需要实现**addArgumentResolvers**就可以了，注意加上**@Configuration**注解将WebConfig托付给Spring，使我们添加的参数解析器生效。

#### 实现HandlerMethodArgumentResolver

```java
@Service
public class SpikeUserArgumentResolver implements HandlerMethodArgumentResolver {

    @Autowired
    SpikeUserService userService;

    public boolean supportsParameter(MethodParameter parameter) {
        Class<?> clazz = parameter.getParameterType();
        //处理SpikeUser类型的
        return clazz==SpikeUser.class;
    }


    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);
        HttpServletResponse response = webRequest.getNativeResponse(HttpServletResponse.class);

        String paramToken = request.getParameter(SpikeUserService.COOK1_NAME_TOKEN);
        String cookieToken = getCookieValue(request, SpikeUserService.COOK1_NAME_TOKEN);
        if(StringUtils.isEmpty(cookieToken) && StringUtils.isEmpty(paramToken)) {
            return null;
        }
        String token = StringUtils.isEmpty(paramToken)?cookieToken:paramToken;
        return userService.getUserByToken(response,token);
    }

    private String getCookieValue(HttpServletRequest request, String cookiName) {
        Cookie[]  cookies = request.getCookies();
        for(Cookie cookie : cookies) {
            if(cookie.getName().equals(cookiName)) {
                return cookie.getValue();
            }
        }
        return null;
    }
}
```

然后就可以直接在在控制器中拿到SpikeUser了，代码变得清爽简洁

```java
	@RequestMapping("/to_list")
    public String tolist(Model model,SpikeUser spikeUser) {
        if (spikeUser==null) {
            return "login";
        }
        model.addAttribute("user", spikeUser);
        return "goods_list";
    }

	//未优化
    @SuppressWarnings("all")
    @RequestMapping("/to_list0")
    @Deprecated
    public String tolist0(HttpServletResponse response, Model model,
                         @CookieValue(value = SpikeUserService.COOK1_NAME_TOKEN, required = false) String cookie,
                         @RequestParam(value = SpikeUserService.COOK1_NAME_TOKEN, required = false) String param) {
                            //手机浏览器，有可能将cookie放在参数中
        System.out.println("cookie:" +cookie);
        System.out.println("param " +param);
        if (param == null && cookie == null) {
            return "login";
        }
        String cook = cookie != null ? cookie : param;
        SpikeUser user = spikeUserService.getUserByToken(response, cook);
        System.out.println(user);
        model.addAttribute("user", user);
        return "goods_list";
    }

```

这也算是对SpringMVC原理的初次接触吧，后面关于框架还是多看源码啊。

关于**Spring-Session**的内容后面用到再来介绍。

### 拦截器

依然是SpringBoot2，所以还是实现的**WebMvcConfigurer**接口

先看下拦截器的执行流程

![来自慕课](http://static.imlgw.top/image/20190609/vUv7FDoickvl.png?imageslim)

#### 自定义注解

```java
package top.imlgw.spike.intercept;

import java.lang.annotation.*;

/**
 * @author imlgw.top
 * @date 2019/6/9 15:13
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface NeedLogin {
    boolean needLogin() default true;
}
```

#### 重写HandlerInterceptor

```java
package top.imlgw.spike.intercept;

import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import top.imlgw.spike.entity.SpikeUser;
import top.imlgw.spike.service.SpikeUserService;
import top.imlgw.spike.utils.UserContext;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author imlgw.top
 * @date 2019/6/8 23:22
 */
@Component
public class LoginIntercept implements HandlerInterceptor {

    @Autowired
    SpikeUserService spikeUserService;

    /**
     * @param request
     * @param response
     * @param handler
     * @return 在登陆前拦截
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if(!(handler instanceof HandlerMethod)){
            return true;
        }
        HandlerMethod hm=(HandlerMethod) handler;
        SpikeUser spikeUser = getSpikeUser(request, response);
        //有的页面不需要登陆(二次登陆)但是需要用户信息(订单页面...)，所以需要先存进去
        UserContext.setUser(spikeUser);
        //获取方法上的注解
        NeedLogin needLogin = hm.getMethodAnnotation(NeedLogin.class);
        if(needLogin==null || ! needLogin.needLogin()){
            //没有注解后者注解为false,就直接放过
            return true;
        }
        //有注解，没登陆
        if(spikeUser==null){
            return false;
        }
        return true;
    }


    /** 视图渲染完毕后调用(收尾工作)
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        //删除ThreadLocal中的User否则会产生错乱
        UserContext.removeUser();
    }

	
    private SpikeUser getSpikeUser(HttpServletRequest request,HttpServletResponse response){
        //拿参数中的token
        String paramToken = request.getParameter(SpikeUserService.COOK_NAME_TOKEN);
        //拿cookie中的token
        String cookieToken = getCookieValue(request, SpikeUserService.COOK_NAME_TOKEN);
        if(StringUtils.isEmpty(cookieToken) && StringUtils.isEmpty(paramToken)) {
            //没登陆cookie为空
            return null;
        }
        String token = StringUtils.isEmpty(paramToken)?cookieToken:paramToken;
        SpikeUser user = spikeUserService.getUserByToken(response, token);
        return user;
    }


    /*
     * 获取cookie中的User
     * */
    private String getCookieValue(HttpServletRequest request, String cookieName) {
        Cookie[]  cookies = request.getCookies();
        if(cookies == null || cookies.length <= 0){
            return null;
        }
        for(Cookie cookie : cookies) {
            if(cookie.getName().equals(cookieName)) {
                return cookie.getValue();
            }
        }
        return null;
    }
}
```

#### WebConfig里面添加拦截器

```java
package top.imlgw.spike.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import top.imlgw.spike.intercept.LoginIntercept;
import java.util.List;

@Configuration
public class WebConfig implements WebMvcConfigurer{
	//自动装配或者 手动创建bean,加到Ioc容器中，否则取不到service
    @Autowired
    SpikeUserArgumentResolver spikeUserArgumentResolver;

    @Autowired
    LoginIntercept loginIntercept;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(spikeUserArgumentResolver);
    }
    

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginIntercept).addPathPatterns("/**").
            excludePathPatterns("/login/*");
    }
}
```

### 视图解析器

不用为了跳转页面而专门写一个**controller**

```java
@Override
public void addViewControllers(ViewControllerRegistry registry) {
    //这里如果是用的模板引擎，就只能是模板引擎template里面的文件
    //这里后面默认指的是static里面的文件，后缀为html
    registry.addViewController("/").setViewName("login");
    registry.addViewController("/goodslist").setViewName("goods_list");
    registry.addViewController("/register").setViewName("register");
}
```

> 其他的以后用到会继续补充

