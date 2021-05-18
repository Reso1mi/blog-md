---
title: 
  Hexo添加其他的评论系统
tags:
  [Hexo]
date: 2018/7/1
categories:
  [Hexo]
---

## 最近换了一个Hexo的主题，也就是现在的这款，但是支持的评论系统两个已经关了，还有一个会被墙。所以就自己来加了一个

### 首先注册并登录LiveRe
登陆注册过程就不多说了
   选择city版的安装，会得到一段代码 
### 在个人博客中加入LiveRe代码

首先去如路径：hexo_bolg/themes/your-theme/layout/_partial/post下创建livere.ejs代码。livere.ejs的内容就是上一步中获取的代码：

```
<!-- 来必力City版安装代码 -->
<div id="lv-container" data-id="city" data-uid="MTAyMC8zMzM5MC85OTQ2">
    <script type="text/javascript">
   (function(d, s) {
       var j, e = d.getElementsByTagName(s)[0];

       if (typeof LivereTower === 'function') { return; }

       j = d.createElement(s);
       j.src = 'https://cdn-city.livere.com/js/embed.dist.js';
       j.async = true;

       e.parentNode.insertBefore(j, e);
   })(document, 'script');
    </script>
<noscript> 为正常使用来必力评论功能请激活JavaScript</noscript>
</div>
<!-- City版安装代码已完成 -->
```

然后修改路径：hexo_bolg/themes/your-theme/layout/_partial下的article.ejs文件，在`<% if (!index && post.comments){ %>` 代码块下添加如下代码：

```
<% if (!index){ %>
  <% if (post.comments){ %>
  <%- partial('post/livere') %>
  <% } else { %>
    <div class="lv-container"></div>
  <% } %>
<% } %>
```
![](http://p0.cdn.img9.top/ipfs/QmWwSjQj5zqz5mAgEtYjgTBRXRqBapvZMaEzArt8xRqQm2?0.jpg)

~~此时LiveRe已经添加OK了，重新部署你的博客然后刷新页面就可以看到博客中添加好了LiveRe评论~~
换了 Valine


