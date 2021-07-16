---
title: Java爬虫实战(一) ：爬取斗图社所有gif
tags:
  - 爬虫
  - Java
  - misc
categories:
  - 爬虫
date: 2018/11/30
cover: 'http://static.imlgw.top///20190407/sAP81lBQEv3Q.png?imageslim'
abbrlink: ada76fed
---
## Java爬虫实战(一) ：爬取斗图社所有gif
最近开始玩爬虫 , 还是挺有意思的 , 虽然写爬虫一般都是用Python比较方便，但是也没有必要为了写爬虫再学一门语言 ,虽然也挺简单，但是还是对Java比较习惯，后面可能会学Python但是目前还是先用java写着玩玩。

**目标** 
[斗图社 ]( https://doutushe.com/)  上所有的图片。
![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/%408Q0%601FQB2A2E4D%5D9M8%40VW8.png)

**技术选择**
Jsoup，最开始看见这个我看成了Jsonp。。。我寻思着不是解决跨域的那个么?还能搞爬虫?这么牛掰的么😄
关于Jsoup网上也有很多文档 [参考资料](http://www.open-open.com/jsoup/) 
还有一些其他的技术比如 httpclient+Xpath 还有htmlunit 还有 selenium 等等，等以后学了后再来秀一秀 😁

---

其实我写的第一个爬虫是copy的别人的博客上的，不过它爬取的是京东的，我爬的是淘宝的
我也只是想参考下他的结构，但是我感觉他的有些类没什么实际意义。。然后我就直接自己写了。
结构如下：

- boot ：爬虫的入口
- dao  ：dao
- handle ：封装的处理查询结果集的类（这里没用）
- model ：爬取的数据的模型
- parse  ： 解析html的类
- util     ： Jsoup工具类和dao的工具类和DB模板类

首先建立数据模型DoutuModel

```java
public class DoutuModel {
	    private Long id;
	    private String topic;
	    private String imgUrl;
	    private String title;
	
	    //数据库id自增
	    public DoutuModel(String topic, String imgUrl, String title) {
	        this.topic = topic;
	        this.imgUrl = imgUrl;
	        this.title = title;
	    }
	
	    public String getTopic() {
	        return topic;
	    }
	
	    public void setTopic(String topic) {
	        this.topic = topic;
	    }
	
	    public String getTitle() {
	        return title;
	    }
	
	    public void setTitle(String title) {
	        this.title = title;
	    }
	
	    public String getImgUrl() {
	        return imgUrl;
	    }
	
	    public void setImgUrl(String imgUrl) {
	        this.imgUrl = imgUrl;
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public void setId(Long id) {
	        this.id = id;
	    }
}	
```

**主要记录下parse类**

``` java
public static List<DoutuModel> getData2(String url) throws Exception {
	        //获取的数据，存放在集合中
	        List<DoutuModel> data = new ArrayList();
	        //采用Jsoup解析
	        //String url="https://doutushe.com/portal/article/index/id/5gK";
	        String preurl = "https://doutushe.com";
	        //取到当前页的document
	        //取到内容页的所有图片
	        int page = 1;
	        while (true) {
	            Document doc = JsoupUtils.getHtmlDocument(url);
	            Elements imgList = doc.select("div[class=col-xs-12 col-sm-8 col-lg-9]").select("img.lazy");
	            String topic = doc.select("blockquote>p").text();
	            for (Element imgelement : imgList) {
	                //异步的坏处体现出来了,这个明显是懒加载,要找就找数据源,直接获取src获取不到
	                //String imgUrl= imgelement.attr("src");
	                String imgUrl = imgelement.attr("data-original");
	                String title = imgelement.attr("title");
	                data.add(new DoutuModel(topic, imgUrl, title));
	                //System.out.println(topic + ":" + imgUrl + ":" + title);
	            }
	            Elements pageUrls = doc.select("ul.pager").select("a");
	            //爬一页休息1秒
	            if (page % 10 == 0) {
	                Thread.sleep(1000);
	                System.out.println("第" + (page/10) + "页采集完 , 暂停-------");
	            }
	            //最后一页也有两个按钮。。。看来要多观察页面
	            /*if (pageUrls.size() < 2) {
	                //说明到最后一页了
	                break;
	            }*/
	            url = preurl + pageUrls.get(1).attr("href");
	            if (!url.matches(preurl + "/portal/article/index/id/[a-zA-Z0-9_]*")) {
	                break;
	            }
	            page++;
	        }
	        //返回数据
	        return data;
}
```
其实一开始写的一个版本是从主页面爬的先获取每一页的链接，再获取每一页的主题的链接，再获取每个主题下的图片链接，一个三重for循环，速度确实比较慢。。。

后来发现每一页都有下一页的链接。。。然后就可以直接从页面上爬，两个循环就可以了，但是一开始我判断边界的时候用的是在下面的链接的数量小于2但是一开始爬了好长时间结果报错了。。  然后我去看了下最后一页发现也有两个链接后面一个是全部的链接。。 。

 ![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/AT5%7BUKZN%24M%5B59V%5B1OU0S%5B7N.png)
而且和上面的css是一样的但是和前面的每一页的链接的后缀不一样，所以就直接用正则表达式匹配url的后缀是否匹配
Regex ：https://doutushe.com/portal/article/index/id/[a-zA-Z0-9_]*  后面的就是直接匹配任意视频
而最后一页的全部链接是 ： https://doutushe.com/portal/index/index 所以就匹配不上直接break

**爬图片的小细节** , 一开始没注意，他这个图片是懒加载的，也就是随着页面用js加载的， 直接src获取肯定获取不到的，因为jsoup是不支持异步的，用js操作的东西肯定爬不到。所以只能通过 data-orginal 获取。

github  [仓库地址](https://github.com/imlgw/javaSpiders)

---

**结果**
![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/X%28%7B%60H6YKH2LIW%24%25F2K4U53M.jpg)
4243张图片，后面可以自己写个脚本把图片全部下载到本地或者用迅雷之类的下载工具。

---
后面会尝试下更多爬虫技术像 htmlunit这种支持异步的或者 selenium 这种直接操作浏览器的工具



