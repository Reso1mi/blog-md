---
title: 你有 FreeBug 么？--->Spring-solr
tags:
  - Bug
  - Solr
  - Spring
categories:
  - 踩坑记录
date: 2018/10/5
abbrlink: 8e2bbf7a
---
## FreeBug ? 哎呦，不错喔。
 昨天从上午 10 点开始一直到晚上 11：58 才把那几个 Bug 给解决了，前两个 Bug 确实蛮奇怪的，特别是第一个 Bug , 最后一个 Bug.... 纯属智障。把这几个 Bug 记录下┗|｀O′|┛ 
### ***Bug1***:
![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/9642SBM2%60QFX4AMO1C%5D~A%5BM.png)
意思是没找到 TbItemMapper 的 select 方法还有这个类的其他方法。
估计是 xml 文件没有加载到 classes 路径下，之前一直好好的然后昨天突然抽风了，百度了下叫我把 xml 文件随便的改动下在里面加个空格换行之类的，然后就好了。
### ***Bug2***:
![img9](https://p4.cdn.img9.top/ipfs/Qmcg5dscbhYgod9vdN2SHaxywdaCPVgY28jX4imd53TH6J?4.png)
看着这些个 Bug 真的是一脸懵，写 main 方法执行就一点问题没有，首先是一波百度，说是 jar 包冲突了主要是 HttpClient 的冲突，然后我就尝试了下 idea 的 maven 依赖视图
![img9](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/DJQYP%604SGOCL_VP%28LGN%7B_TN.png )
然后发现 dubbo 和 spring-data-solr 都有 httpclient 包和 httpcore 而且版本不一致，然后果断的把 dubbo 里面的 httpclient 和 httpcore 给 exclusion 了，不看不知道，整个项目的 maven 依赖好乱，有好多依赖冲突，不过没影响使用我就没有去改，怕再改出什么问题。你以为改完之后 bug 就结束了？
### ***Bug3***:
![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/%60Z3GBQ~K%29%7D%60DX%60P%60Q%257%25%7B%25S.png) 在 controller 层疯狂报错，service 层一点问题没有！[oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/NLKUXAM~LPOZ7K%60%28%29D52%40VD.png) 可以，又一个 FreeBug，在纠结了几个小时后看了一眼代码发现
![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/L20FMBQ%28XNRQV6Y6%24ITFEYA.png) 原来这里写掉了东西，返回了一个 solr 的结果集也没有实现序列化返回出去了前台也根本就解析不了难怪会再前台报错。

ps: 之前一直用的 img9.top 的无限图床，前几天突然崩溃了博客的图片都失效了，之前也有一个送的阿里云的 oss 的包所以就想拿那个当图床，这篇文章里面的图片也都是在 oss 上的，img9 虽然也还可以但是是个去中心化的图床不好管理确实难搞，等有时间就用 java 写一个自动化的图床工具玩玩。