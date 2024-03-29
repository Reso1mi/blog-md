---
title: 七牛云免费额度开启 HTTPS 代理
tags:
  - Nginx
categories:
  - Web
date: 2021/2/22
abbrlink: 1fab87bb
---

## 起因
最近给自己的博客上了 https，而图床一直用的七牛云的免费额度，但是七牛云的免费额度是不支持 HTTPS 流量的，~~作为资深白嫖怪当然首先是考虑如何白嫖了~~

## 解决方案
因为要上全站 https，所以我们站点是不允许发送 http 请求的，http 的外链资源都无法加载。其实解决方案也很简单，我们再套一层 https 的代理就行了。

我们七牛云的 cdn 加速域名是 [static.imlgw.top]()，我们再申请一个子域名 [qiniu.imlgw.top]()，然后给这个子域名添加一条 A 记录指向我们的服务器，然后再给这个域名申请一套 ssl 证书上传到服务器上，然后就是对 Nginx 的配置了

```sh
# 配置 https 代理
server {
    listen        443 ssl;
    # https 跳板
    server_name  qiniu.imlgw.top;
    # 证书
    ssl_certificate  /usr/local/cert/qiniu.imlgw.top.pem;
    ssl_certificate_key  /usr/local/cert/qiniu.imlgw.top.key;
    location / {
        # 七牛云 CDN 域名
        proxy_pass http://static.imlgw.top;
    }
}
```
然后我们就可以通过 [https://qiniu.imlgw.top/abc.jpg](https://qiniu.imlgw.top/abc.jpg)
 代理访问 [http://static.imlgw.top/abc.jpg](http://static.imlgw.top/abc.jpg)，进而继续白嫖七牛云的免费额度🤣

 ## 进阶

实际上这种方案我弄好了后并没有采用，主要原因如下：

1. 这样代理之后图片都会通过我的服务器中转，七牛云 cdn 加速其实就没用了，而我的服务器是个 1Mbps 的小水管，图片的加载速度会变慢
2. `MPic`上传图片获取链接后还需要手动的去修改链接为 qiniu...，事情变复杂了，没有那么方便了
3. 这样代理之后，之前所有的文章图片都需要将链接替换成`qiniu.imlgw.top`，虽然文章也不多，写个脚本一下就替换了，但是终归是有点麻烦

其实这种方案有一个更好的用途：可以做到**无缝换源** 。我们可以**同步**我们的多个对象存储源，当我们其中一个发生故障的时候（欠费）我们可以修改 Nginx 的配置，将其代理到另一个源这样我们文章的外链不需要做任何修改依然可以正常访问

## 结语
最终还是选择了付费，其实本来是打算采用阿里云 OSS 做付费图床的，但是阿里云的计费规则我实在是有点摸不透，空间很便宜 9 元 40G/Year，但是流量的计算有点复杂，而且要做图床替换也是个麻烦事，得写个通用脚本出来，用七牛云的话只要升级了 https，然后开启强制 https，文章中的链接都不需要修改就可以直接访问（我这里最新的 Chrome 会自动把 https 站点的 http 请求替换成 https，但是其他的浏览器就不知道了，所以我们直接开启强制 https 就行了）

七牛云的我大概看了下，计费规则还是很清晰的，免费的额度是：10G 空间，10G 的 CDN-HTTP 流量。使用了 CDN 加速后就不计算外网流出的流量费用。所以我需要做的就是将 http 的 cdn 流量升级成 https，价格如下
![](https://i.loli.net/2021/02/23/HDhXPIkGSvmRuK4.png) 我充了一点点钱，先按量付费用着看，看看一年能不能把我冲的这点钱花完😂（立个 flag 一年后再来看），毕竟是个人博客，除了自己也没啥人会看，如果流量大了可以考虑卖资源包，有打折，会便宜一点。
