---
title: Spring 请求参数获取的几种方式（转）
tags:
  - Spring
  - 转载
categories:
  - Web
date: 2018/5/17
abbrlink: 81b3fc53
---
# [springmvc 请求参数获取的几种方法](http://www.cnblogs.com/xiaoxi/p/5695783.html)
**1、直接把表单的参数写在 Controller 相应的方法的形参中，适用于 get 方式提交，不适用于 post 方式提交。**

```java 
    /** * 1. 直接把表单的参数写在 Controller 相应的方法的形参中
      * @param username
     * @param password
     * @return
     */ @RequestMapping("/addUser1") public String addUser1(String username,String password) {
        System.out.println("username is:"+username);
        System.out.println("password is:"+password); return "demo/index";
    }
```
url 形式：[http://localhost/SSMDemo/demo/addUser1?username=lixiaoxi&password=111111](http://localhost/SSMDemo/demo/addUser1?username=lixiaoxi&password=111111) 提交的参数需要和 Controller 方法中的入参名称一致。

**2、通过 HttpServletRequest 接收，post 方式和 get 方式都可以。**

```java
   /** 2、通过 HttpServletRequest 接收
    * @param request
     * @return
     */ @RequestMapping("/addUser2") public String addUser2(HttpServletRequest request) {
        String username=request.getParameter("username");
        String password=request.getParameter("password");
        System.out.println("username is:"+username);
        System.out.println("password is:"+password); return "demo/index";
    }
```

**3、通过一个 bean 来接收，post 方式和 get 方式都可以。**
(1) 建立一个和表单中参数对应的 bean

```java
package demo.model; public class UserModel { private String username; private String password; public String getUsername() { return username;
    } public void setUsername(String username) { this.username = username;
    } public String getPassword() { return password;
    } public void setPassword(String password) { this.password = password;
    }

}
```

(2) 用这个 bean 来封装接收的参数

```java
 /** * 3、通过一个 bean 来接收
      * @param user
     * @return
     */ @RequestMapping("/addUser3") public String addUser3(UserModel user) {
        System.out.println("username is:"+user.getUsername());
        System.out.println("password is:"+user.getPassword()); return "demo/index";
    }
```
**4、通过@PathVariable 获取路径中的参数**

```java
/*** 4、通过@PathVariable 获取路径中的参数
      * @param username
     * @param password
     * @return
     */ @RequestMapping(value="/addUser4/{username}/{password}",method=RequestMethod.GET) public String addUser4(@PathVariable String username,@PathVariable String password) {
        System.out.println("username is:"+username);
        System.out.println("password is:"+password); return "demo/index";
    }
```
例如，访问 [http://localhost/SSMDemo/demo/addUser4/lixiaoxi/111111](http://localhost/SSMDemo/demo/addUser4/lixiaoxi/111111) 路径时，则自动将 URL 中模板变量{username}和{password}绑定到通过@PathVariable 注解的同名参数上，即入参后 username=lixiaoxi、password=111111。
**5、使用@ModelAttribute 注解获取 POST 请求的 FORM 表单数据**
Jsp 表单如下：
```java
<form action ="<%=request.getContextPath()%>/demo/addUser5" method="post"> 用户名：&nbsp;<input type="text" name="username"/><br/> 密&nbsp;&nbsp; 码：&nbsp;<input type="password" name="password"/><br/>
     <input type="submit" value="提交"/> 
     <input type="reset" value="重置"/> 
</form> 
```
Java Controller 如下：

```java 
/** * 5、使用@ModelAttribute 注解获取 POST 请求的 FORM 表单数据
     * @param user
     * @return
     */ @RequestMapping(value="/addUser5",method=RequestMethod.POST) public String addUser5(@ModelAttribute("user") UserModel user) {
        System.out.println("username is:"+user.getUsername());
        System.out.println("password is:"+user.getPassword()); return "demo/index";
    } 
```

 

**6、用注解@RequestParam 绑定请求参数到方法入参**
当请求参数 username 不存在时会有异常发生，可以通过设置属性 required=false 解决，例如：@RequestParam(value="username", required=false)

```java 
/** * 6、用注解@RequestParam 绑定请求参数到方法入参
      * @param username
     * @param password
     * @return
     */ @RequestMapping(value="/addUser6",method=RequestMethod.GET) public String addUser6(@RequestParam("username") String username,@RequestParam("password") String password) {
        System.out.println("username is:"+username);
        System.out.println("password is:"+password); return "demo/index";
    }
```