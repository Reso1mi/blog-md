---
title: Nginx 学习
tags:
  - Nginx
categories:
  - Web
date: 2018/5/13
abbrlink: dda0de9c
---
# Nginx 学习笔记
- Nginx 简介
  _Nginx_ (engine x) 是一个高性能的 [HTTP](https://baike.so.com/doc/5366073-5601774.html) 和 [反向代理](https://baike.so.com/doc/5345781-5581226.html) 服务器，也是一个 IMAP/POP3/SMTP[服务器](https://baike.so.com/doc/4487696-4696885.html)。Nginx 是由伊戈尔·赛索耶夫为俄罗斯访问量第二的 Rambler.ru 站点（俄文：Рамблер) 开发的，第一个公开版本 0.1.0 发布于 2004 年 10 月 4 日。
其将源代码以类 BSD 许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。2011 年 6 月 1 日，nginx 1.0.4 发布。
Nginx 是一款 [轻量级](https://baike.so.com/doc/585215-619452.html) 的 [Web](https://baike.so.com/doc/4230501-4432285.html) 服务器/[反向代理](https://baike.so.com/doc/5345781-5581226.html) 服务器及 [电子邮件](https://baike.so.com/doc/928072-980969.html)(IMAP/POP3) 代理服务器，并在一个 BSD-like 协议下发行。其特点是占有内存少，[并发](https://baike.so.com/doc/6916691-7138566.html) 能力强，事实上 nginx 的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用 nginx 网站用户有：百度、京东、[新浪](https://baike.so.com/doc/2204905-2333035.html)、网易、[腾讯](https://baike.so.com/doc/1038695-1098608.html)、淘宝等。
	
- Nginx 主要作用
1. (Http 服务器）处理静态文件
![image](http://p0.cdn.img9.top/ipfs/QmdiKx1JUr7y9TabUQNNqu5dP8AwdsHSHe9QMwa487rsaS?0.png)
像这样配置的话当访问 *localhost:80* 时就会访问到相对当前路径下的 html 文件夹下的 index.html 或者 index.htm
2. 反向代理
    **反向代理** 也是 nginx 用的最多的地方，既然有反向代理那就肯定有 **正向代理** 先来理解下正向代理！[image](https://gss0.baidu.com/7LsWdDW5_xN3otqbppnN2DJv/zhidao/pic/item/a2cc7cd98d1001e96d753f76b10e7bec54e79779.jpg)
正向代理其实就是代理上网，**客户端**发送请求到代理服务器然后代理服务器转交请求给**目标服务器**目标服务器响应给代理服务器代理服务器再响应给**客户端**也就起到了代理的作用
那反向代理是什么呢？
![image](https://gss0.baidu.com/7LsWdDW5_xN3otqbppnN2DJv/zhidao/pic/item/a8014c086e061d95f3f21c4172f40ad162d9ca17.jpg)
正向代理针对的是**客户端**而反向代理正对的是**服务端**
当用户发送一个请求给 ServerB 然后 ServerB 判断用户是什么请求
是请求 Server1 就转发给 Server1... 在生产中前端用一个 Nginx 处理用户的请求再分发到不同的后端服务器。
**配置**
```java
upstream sina{
	server 192.168.125.3:8080;
    }
    server {
        listen       80;
        server_name  www.sina.com.cn;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass   http://sina;
            index  index.html index.htm;
        }

    }

     upstream sohu{
	server 192.168.125.3:8081;
    }
    server {
        listen       80;
        server_name  www.sohu.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
			#交给谁代理
            proxy_pass   http://sohu;
            index  index.html index.htm;
        }

    }
```

像这样配置客户端访问通过一个公网 ip 入口先到达 Nginx 然后转发给对应的服务器当用户访问* www.sohu.com *时就会转发给 http://sohu 这个就是上面的 upstream sohu{...} 交给它处理
3. 负载均衡和容错
	负载均衡就是在反向代理的中间加上的 就是在 upstream 里面加一个 server，最简单的负载均衡就是轮询一人一次刷新就换 (*默认的就是轮询*) 当然你也可以根据服务器性能配置权重权重越大访问到的机会越大

```java
upstream sina{
	server 192.168.125.3:8080;
	#weigth 是权重
	server 192.168.25.148:8082 weight=2;
    }
    server {
        listen       80;
        server_name  www.sina.com.cn;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass   http://sina;
            index  index.html index.htm;
         }
       }
```
由于 Nginx 是我们网站的入口如果 Nginx 挂掉后面的服务就都失效了这时候就可以加备用 Nginx 服务器用 keepalive+Nginx 实现主备！[image](http://p0.cdn.img9.top/ipfs/Qmc3nC82Xsh1QANbUfAdDGUmTL2KJ2bQzoYzge7VHvRRWu?0.png)
