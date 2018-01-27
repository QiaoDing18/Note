---
title: Get和Post区别
tags: 计算机网络,Get,Post
---
## 一些理解
1、Get后退按钮/刷新无害，Post数据**会被重新提交**（浏览器应该告知用户数据会被重新提交）
2、Get书签可收藏，Post为书签不可收藏
3、Get能被**缓存**，Post不能缓存
4、Get编码类型application/x-www-form-url，Post编码类型encodedapplication/x-www-form-urlencoded 或 multipart/form-data。为二进制数据使用多重编码
5、Get历史参数保留在浏览器历史中，Post参数不会保存在浏览器历史中
6、Get对数据**长度有限制**，当数据发送时，Get方法向URL添加数据，URL的长度是受限制的（URL的最大长度是2048个字符）,Post无限制
7、Get只允许ASCⅡ字符，Post没有限制，也允许二进制数据
8、Get不安全，发送的数据是URL的一部分。Post安全一点，因为参数不会被保存在浏览器历史或者Web服务器日志中。
9、Get产生**一个TCP数据包**，Post产生**2个TCP数据包**
  Get时，浏览器会把http header和data一并发送过去，服务器响应200，
  Post时，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok(返回数据)
  
## jsonp
同源策略：域名、协议、端口
```javascript
 oBtn.onclick = function() {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4 && xhr.status == 200) {
            alert( xhr.responseText );
        }
    };
    xhr.open('get', 'https://api.douban.com/v2/book/search?q=javascript&count=1', true);
    xhr.send(); 
};
```
![会报错][1]

但是img的src（获取图片），link的href（获取css），script的src（获取javascript）这三个都不符合同源策略，它们可以跨域获取数据。
jsonp是动态创建script标签，然后利用script的src不受同源策略约束来跨域获取数据
jsonp由两部分组成：回调函数和数据。回调函数是当相应到来时应该在页面中调用的函数。回调函数的名字一般是在请求中指定的。而数据就是传入回调函数中的json格式数据
```javascript
var script = document.createElement("script");
script.src = "xxoo?canshu=1&result=ok&callback=myCallbackName";
document.body.insertBefore(script, document.body.firstChild);
```
在页面中，返回的json作为参数传入回调函数
```javascript
function myCallbackName(result){
	console.log(result);
}
```
传递一个callback参数给跨域服务端，然后跨域服务端返回数据时会将这个callback参数作为函数名来包裹住json数据即可。

##### 例子
在跨域服务器上xx.js:
```javascript
localHandler({"result":"远程js带来的数据"})
```
本地：
```javascript
<script>
	var localHandler = function(data){
		alert("本地得到远程的数据是：" + data.result);
	}
</script>
<script src="跨域服务器/xxoo.js"></script>
```
**本地定义一个函数，引入外部js，外部的js里调用这个函数**
但是服务器端不知道本地函数的名字

node后台可以动态获取函数名，然后返回
```javascript
var express = require('express');
var router = express.Router();
router.get('/xxoo', function(req, res, next){
	var _callback = req.query.callback;
	var _data = {json};
	if(_callback){
		res.type('text/javascript'); //告诉浏览器这是js代码
		res.send(_callback + "()")
	}
})
```
通过获取请求的url中callback=xx的拼接信息动态的获取到函数名称
或者直接用res.jsnop();



  [1]: ./images/1517050675842.jpg