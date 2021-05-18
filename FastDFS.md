---
title: 
  FastDFS学习
tags:
  [入门,FastDFS,淘淘商城]
categories:
  [运维]
date: 2018/5/16
---
# FastDFS学习笔记
- FastDFS简介
	FastDFS服务端有两个角色：跟踪器（tracker）和存储[节点](https://baike.baidu.com/item/%E8%8A%82%E7%82%B9)（storage）。跟踪器主要做调度工作，在访问上起负载均衡的作用。
存储节点存储文件，完成文件管理的所有功能：就是这样的存储、同步和提供存取接口，FastDFS同时对文件的metadata进行管理。所谓文件的meta data就是文件的相关属性，以键值对（key value）方式表示，如：width=1024，其中的key为width，value为1024。文件metadata是[文件属性](https://baike.baidu.com/item/%E6%96%87%E4%BB%B6%E5%B1%9E%E6%80%A7)列表，可以包含多个键值对。
跟踪器和存储节点都可以由一台或多台服务器构成。跟踪器和存储节点中的服务器均可以随时增加或下线而不会影响线上服务。其中跟踪器中的所有服务器都是对等的，可以根据服务器的压力情况随时增加或减少。
为了支持大容量，存储[节点](https://baike.baidu.com/item/%E8%8A%82%E7%82%B9)（服务器）采用了分卷（或分组）的组织方式。[存储系统](https://baike.baidu.com/item/%E5%AD%98%E5%82%A8%E7%B3%BB%E7%BB%9F)由一个或多个卷组成，卷与卷之间的文件是相互独立的，所有卷的文件容量累加就是整个存储系统中的文件容量。一个卷可以由一台或多台[存储服务器](https://baike.baidu.com/item/%E5%AD%98%E5%82%A8%E6%9C%8D%E5%8A%A1%E5%99%A8)组成，一个卷下的存储服务器中的文件都是相同的，卷中的多台存储服务器起到了[冗余备份](https://baike.baidu.com/item/%E5%86%97%E4%BD%99%E5%A4%87%E4%BB%BD)和负载均衡的作用。
在卷中增加服务器时，同步已有的文件由系统自动完成，同步完成后，系统自动将新增服务器切换到线上提供服务。
当存储空间不足或即将耗尽时，可以动态添加卷。只需要增加一台或多台服务器，并将它们配置为一个新的卷，这样就扩大了[存储系统](https://baike.baidu.com/item/%E5%AD%98%E5%82%A8%E7%B3%BB%E7%BB%9F)的容量。
FastDFS中的文件标识分为两个部分：卷名和文件名，二者缺一不可。
- 大胆分析
这里在淘淘的后台上传图片的时候用到了,确实好用也是c语言开发的 搭配*Nginx*贼方便,确实佩服那些搞**C**的大佬，上面的说法不够直白文字总是苍白的(￣▽￣)~*让我来强行解释一波![image](http://p1.so.qhmsg.com/bdr/_240_/t0183119f547ca0bbc6.png)
上面说到了**FastDFS**主要有两个节点
 1. 储存节点Storage  
顾名思义就是用来储存文件的服务器这个图中的是比较复杂的情况这里有一个stroage 群里又分了很多组每个组里又有很多服务器（这里的服务器里面储存的文件是一样的 会自动同步 方便加机器）
  2. 监控节点Tracker 
这个就跟[zookeper](http://www.so.com/link?m=a0NGOfhwBC5nSFnUTBxpX2uhRysff5w5UsQFEqgCBRrJbwZpybFckcSRdwhuJdbgSApcIIqhL0gq5D6eIbBJ5pN22O2Z0k6ENTFGSjrRpsgETp2Dl)作用有点像先上图
![image](http://p0.cdn.img9.top/ipfs/QmZynH9DgJJbePkn35LjhRTc6xyyWL64jNT6QU6jyBnrXE?0.png
)
这个是上传的过程可以看到客户端要上传图片都是通过Tracker的而储存节点也会定时向Tracker发送状态的信息监控Storage的状态
`可以看出上面Client在上传图片成功后Storage返回了一个file_id然后客户端就会储存这个id那客户端是如何通过这个id访问到这个图片的呢？ ---没错就是Nginx 用Nginx来处理这些静态资源再好不过了`
- 使用
 说了这么多来实际用用看吧  
   1. 首先我们要在虚拟机上安装FastDFS并配置Nginx这个过程比较复杂也不是我们重点关心的问题是运维应该关心的问题这里有一个搭建好的最简单的[FastDFS服务器](https://pan.baidu.com/s/1u5FLtQu71CueAJwq63ji6A)开机就可以直接用服务都是开机自启动的  
  2. 然后我们要有客户端这里FastDFS作者已经写好了JAVA的客户端我们直接拿来用就好了[fastdfs_client的jar包](https://pan.baidu.com/s/1KY5BKUr6f1PlCRR7hyhRSA)
  3. 开始使用吧  

``` java
@Test
	public void testUpload() throws Exception {

		//1、加载配置文件，配置文件中的内容就是tracker服务的地址。
		//配置文件内容：tracker_server=192.168.25.133:22122
		ClientGlobal.init("D:\\JavaDemo\\taoshop-web\\src\\main\\resources\\conf\\client.conf");
		//2、创建一个TrackerClient对象。直接new一个。
		TrackerClient trackerClient =new TrackerClient();
		//3、使用TrackerClient对象创建连接，获得一个TrackerServer对象。
		TrackerServer trackerServer = trackerClient.getConnection();
		//4、创建一个StorageServer的引用，值为null
		StorageServer storageServer =null;
		//5、创建一个StorageClient对象，需要两个参数TrackerServer对象、StorageServer的引用
		StorageClient storageCilent =new StorageClient(trackerServer,storageServer);
		//6、使用StorageCilent上传文件
		String[] strings = storageCilent.upload_file("C:\\Users\\Administrator\\Desktop\\image\\222.jpg","jpg", null);
		for (String string : strings) {
			System.out.println(string);
		}
```
这里最后也就是为了的到StorageServer对象 我们可以写个工具类方便我们上传，将得到的file信息直接放在虚拟机ip/后面
![image](http://p3.so.qhimgs1.com/bdr/_240_/t0151a407cda74a2153.png)成功访问到了这张图片！还是很方便的。
- 在淘淘商城中的运用
   在淘淘中主要是用在后台的图片上传上的 这里用的是ssm的组合SpringMVC上传图片首先要有commons.io和fileupload的jar包还有配置多媒体解析器
在这里我也遇到了一个小问题 他在这里用的是*KingEditor* 上传图片时要求返回的格式是
```  java
  //成功时
  { 
        "error" : 0,
        "url" : "http://www.example.com/path/to/file.ext"
}
//失败时
{
        "error" : 1,
        "message" : "错误信息"
}
```
我直接返回的一个map然后前台解析不出来出现了问题后来用fastJson把map转成json后就好了但是其实传还是传到了图片服务器上去了

- 在做这个的时候还遇到了一些奇怪的问题就是如果修改代码后马上重新启动项目会报错clean一下就好了。