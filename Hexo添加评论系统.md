---
title: Hexo 添加其他的评论系统
tags:
  - Hexo
date: 2018/7/1
categories:
  - Hexo
abbrlink: bbbb4402
---

## 最近换了一个 Hexo 的主题，也就是现在的这款，但是支持的评论系统两个已经关了，还有一个会被墙。所以就自己来加了一个

### 首先注册并登录 LiveRe
登陆注册过程就不多说了
   选择 city 版的安装，会得到一段代码 
### 在个人博客中加入 LiveRe 代码

首先去如路径：hexo_bolg/themes/your-theme/layout/_partial/post 下创建 livere.ejs 代码。livere.ejs 的内容就是上一步中获取的代码：

```
<!-- 来必力 City 版安装代码 -->
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
<noscript> 为正常使用来必力评论功能请激活 JavaScript</noscript>
</div>
<!-- City 版安装代码已完成 -->
```

然后修改路径：hexo_bolg/themes/your-theme/layout/_partial 下的 article.ejs 文件，在`<% if (!index && post.comments){ %>` 代码块下添加如下代码：

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

~~此时 LiveRe 已经添加 OK 了，重新部署你的博客然后刷新页面就可以看到博客中添加好了 LiveRe 评论~~
换了 Valine
