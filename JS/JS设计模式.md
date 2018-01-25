---
title: JS设计模式
tags: JavaScript,前端
---

## 单利模式
> 在应用单例模式时，生成单例的类必须保证**只有一个实例**的存在，很多时候整个系统只需要拥有一个全局对象，才有利于协调系统整体的行为。比如在整个系统的配置文件中，配置数据有一个单例对象进行统一读取和修改，其他对象需要配置数据的时候也统一通过该单例对象来获取配置数据，这样就可以简化复杂环境下的配置管理。

> 单例模式的思路是：一个类能返回一个对象的引用（并且永远是同一个）和一个获得该实例的方法。那么当我们调用这个方法时，如果类持有的引用不为空就返回该引用，否者就创建该类的实例，并且将实例引用赋值给该类保持的那个引用再返回。同时将该类的构造函数定义为私有方法，避免其他函数使用该构造函数来实例化对象，只通过该类的静态方法来得到该类的唯一实例。

#### 实现1：对象字面量
```javascript
var mySingleton = {
    property1: "something",
    property2: "something else",
    method1: function(){
        console.log('hello world');
    }
}
var m1 = mySingleton;
var m2 = mySingleton;
```
#### 实现2：构造函数
```javascript
var singleton = (function(){
	var instantiated;
	function init(){
		// 定义单利代码
		return {
			publicMethod:function(){
				console.log('hello world');
			},
			publicProperty: 'test'
		};
	}
	return {
		getInstance: function(){
			if(!instantiated){
				instantiated = init();
			}
			return instantiated;
		}
	}
})();
singleton.getInstance().publicMethod();
```
## 观察者模式
又叫做观察者模式。一个发布一个订阅
#### 优点：
1、支持简单的广播通信，当对象状态发生改变时，会自动通知已经订阅过的对象
2、发布者与订阅者耦合性降低，发布者只管发布一条消息出去，它不关心这条消息如何被订阅者使用，同时，订阅者只监听发布者的事件名，只要发布者的事件名不变，它不管发布者如何改变
#### 缺点：
1、创建创建订阅者需要消耗时间和内存
2、过度使用使维护性低
#### 实现
1、想好谁是发布者
2、在发布者中添加一个缓存列表，用来存放回调函数来通知订阅者
3、发布消息，发布者遍历缓存列表，依次触发存放的回调函数
```javascript
// 封装一个
    var Event = (function(){
      var list = {};
      
      var listen = function(key, fn){
        if(!list[key]){
          list[key] = [];
        }
        list[key].push(fn);
      };

      var trigger = function(){
        var key = Array.prototype.shift.call(arguments);
        var fns = list[key];
        if(!fns || fns.length === 0){
          return false;
        }
        for(var i=0, fn; fn=fns[i]; i++){
          fn.apply(this, arguments);
        }
      };

      var remove = function(key, fn){
        var fns = list[key];
        if(!fns){
          return false;
        }
        if(!fn){
          fns && (fns.length = 0);
        }else{
          for(var i=fns.length-1; i>=0; i--){
            var _fn = fns[i];
            if(_fn === fn){
              fns.splice(i, 1);
            }
          }
        }
      };

      return {
        listen: listen,
        trigger: trigger,
        remove: remove
      }
    })();

    // 测试代码
    Event.listen("color", function(size){
      console.log("尺码为："+size);
    });
    Event.trigger("color", 42);
```
