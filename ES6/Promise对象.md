---
title: Promise对象
tags: 前端,ES6
---

## Promise的含义
所谓Promise，就是一个对象，用来传递异步传递消息的操作。
Promise有两种特点：
1、对象的状态不受外界影响。Promise对象代表一个异步操作。**有三种状态：Pending（进行中）、Resolved（已完成，又称Fulfilled）和Rejected（已失败）**。只有异步操作的结果可以决定当前是哪一种状态，人恶化其他操作都无法改变这个状态。
2、一旦状态改变就不再改变。Promise对象的状态改变**只有两种可能：从Pending变为Resolved和从Pending变为Rejected**。只要其中之一发生，状态就会凝固，不再改变。再添加回调函数，也会立即得到这个结果。与事件不同，事件如果错过，再去监听也得不到结果。

有了Promise对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调。Promise对象提供统一接口，使得控制异步操作更加容易。

>Promise的缺点：
>1、无法取消Promise，一旦新建它就会立即执行，无法中途取消。
>2、如果不设置回调函数，Promise内部抛出的错误不会反应到外部。
>3、当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

如果某些时间不断的反复发生，一般来说，使用stream模式是比部署Promise更好的选择。

## Promise基本用法
ES6规定，Promise对象是一个构造函数，用来生成Promise实例
```javascript
var promise = new Promise(function(resolve, reject){
	// some code
	if(/*异步操作成功*/){
		resolve(value);
	}else{
		reject(error);
	}
});
```
romise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。

**resolve**：resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去

**reject**：reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去

Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。
```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```
then方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为resolved时调用，第二个回调函数是Promise对象的状态变为rejected时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数。

```javascript
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
}); // done
```
上面代码中，timeout方法返回一个Promise实例，表示一段时间以后才会发生的结果。过了指定的时间（ms参数）以后，Promise实例的状态变为resolved，就会触发then方法绑定的回调函数。

异步加载图片的例子
```javascript
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    const image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
```
上面代码中，使用Promise包装了一个图片加载的异步操作。如果加载成功，就调用resolve方法，否则就调用reject方法。

下面是一个用Promise对象实现的 Ajax 操作的例子。
```javascript
const getJSON = function(url) {
  const promise = new Promise(function(resolve, reject){
    const handler = function() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    const client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```
上面代码中，getJSON是对 XMLHttpRequest 对象的封装，用于发出一个针对 JSON 数据的 HTTP 请求，并且返回一个Promise对象。需要注意的是，在getJSON内部，resolve函数和reject函数调用时，都带有参数。

如果调用resolve函数和reject函数时带有参数，那么它们的参数会被传递给回调函数。reject函数的参数通常是Error对象的实例，表示抛出的错误；resolve函数的参数除了正常的值以外，还可能是另一个 Promise 实例，比如像下面这样。
```javascript
const p1 = new Promise(function (resolve, reject) {
  // ...
});

const p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```
上面代码中，p1和p2都是 Promise 的实例，但是p2的resolve方法将p1作为参数，即一个异步操作的结果是返回另一个异步操作。

注意，这时p1的状态就会传递给p2，也就是说，p1的状态决定了p2的状态。如果p1的状态是pending，那么p2的回调函数就会**等待p1的状态改变**；如果p1的状态已经是resolved或者rejected，那么p2的回调函数将会**立刻执行**。
```javascript
var p1 = new Promise(function(resolve, reject){
	setTimeout(() => reject(new Error('fail')), 3000);
});
var p2 = new Promise(function(resolve, reject){
	setTimeout(() => resolve(p1), 1000);
});
p2.then(result => console.log(result));
p2.catch(error => console.log(error));
//Error: fail
```

