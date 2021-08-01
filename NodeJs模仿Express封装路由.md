---
title: NodeJS 模仿 Express 封装路由
tags:
  - NodeJS
  - ES6
  - JavaScript
categories:
  - Web
date: 2018/12/3
cover: 'https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/timg%5B1%5D.jpg'
abbrlink: 3b46123d
---

## NodeJS 模仿 Express 封装路由

_最近才开始学 NodeJs 也不知道为啥就突然对这个很感兴趣，（可能 Java 写烦了😄)，感觉用这个开发还是挺快的，而且性能也很好，借此也了解下函数式编程的特点和异步编程的思想。_

- ### 没封装前
  原生的就差不多是这样的。

```java
	/* 
	    Node.js 未封装 1.0
	*/
	const http = require("http");
	const fs = require("fs");
	const path = require("path");
	const querystring = require("querystring"); //json 转换
	const scores = require("./StudentScore.json");
	const template = require("art-template");
	http.createServer((req, resp) => {
	    if (req.url.startsWith("/query") && req.method == 'GET') {
	        //这里可以采用模板为了和下面的对比一下
	        fs.readFile(path.join(__dirname, "querypage.html"), (err, content) => {
	            if (err) {
	                resp.writeHead(500, {
	                    'Content-Type': 'text/plain;charset=utf8'
	                });
	                resp.end('服务器错误');
	            }
	            resp.end(content);
	        });
	    } else
	    if (req.url.startsWith("/scores")) {
	        let pdata = '';
	        //事件绑定
	        //获取数据（id）
	        req.on('data', (ck) => {
	            pdata += ck;
	        });
	        //在这里返回
	        req.on('end', () => {
	            let obj = querystring.parse(pdata); //将参数 的字符串转换成 对象
	            let result = scores[obj.stunum];
	            let content = template(path.join(__dirname, "scores.art"), result);
	            resp.end(content);
	        });
	    }
	}).listen(9999, () => {
	    console.log('Server is runing on 9999');
	});
```

- Express 的方式
通过 const app=express(); 获得一个 app 的对象后面就通过这个来操作
![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/D~KS%28D1%5B%5DUS5TBZPO7KTJ88.png)
- 手动封装

```java
	var url = require('url');
	
	//封装方法改变 res  绑定 res.send()
	function changeRes(res) {
	
	    res.send = function(data) {
	
	        res.writeHead(200, {
	            "Content-Type": "text/html;charset=utf-8"
	        });
	
	        res.end(data);
	    }
	}
	
	//定义暴露的模块  return 里面定义的模块
	var Server = function() {
	
	    var G = this; /*全局变量*/
	
	    //处理 get 和 post 请求
	    this._get = {};
	
	    this._post = {};
	
	    var app = function(req, res) {
	
	        changeRes(res);
	
	        //获取路由
	        var pathname = url.parse(req.url).pathname;
	        if (!pathname.endsWith('/')) {
	            pathname = pathname + '/';
	        }
	
	        //获取请求的方式 get  post
	        var method = req.method.toLowerCase();
	
	        if (G['_' + method][pathname]) {
	
	            if (method == 'post') { /*执行 post 请求*/
	
	                var postStr = '';
	                req.on('data', function(chunk) {
	
	                    postStr += chunk;
	                })
	                req.on('end', function(err, chunk) {
	                    //添加请求属性
	                    req.myBody = postStr; /*表示拿到 post 的值*/
	                    /*执行方法*/
	                    G['_' + method][pathname](req, res);
	                })
	            } else { /*执行 get 请求*/
	                G['_' + method][pathname](req, res); /*执行方法*/
	            }
	        } else {
	            res.end('no router');
	        }
	    }
	
	    //下面的都是为了做注册的操作
	    app.get = function(string, callback) {
	        if (!string.endsWith('/')) {
	            string = string + '/';
	        }
	        if (!string.startsWith('/')) {
	            string = '/' + string;
	
	        }
	
	        //    /login/
	        G._get[string] = callback;
	
	    }
	
	    app.post = function(string, callback) {
	        if (!string.endsWith('/')) {
	            string = string + '/';
	        }
	        if (!string.startsWith('/')) {
	            string = '/' + string;
	
	        }
	        //    /login/
	        G._post[string] = callback;
	
	        //G._post['dologin']=function(req,res){
	        //
	        //}
	    }
	
	    return app;
	
	}
	
	module.exports = Server();
```

- 封装后
```java
	const http = require("http");
	const url = require("url");
	//引入自定义的路由模块
	const myApp = require("./model/express-route.js");
	
	//使用自定义的模块
	http.createServer(myApp).listen(9999, () => {
	    console.log('Running 9999');
	});
	
	myApp.get("/express", (req, resp) => {
	    resp.send("模仿 Express 封装路由");
	    console.log(req);
	});
	myApp.post("/postExpress", (req, resp) => {
	    resp.send("模仿 Express 封装路由");
	    console.log(req.myBody);
	});
```

---
通过这个体会下 Express 的封装
