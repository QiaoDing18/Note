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
上面的代码中，p1是一个Promise，3秒之后变为Rejected。p2的状态由p1决定，1秒之后，p2调用resolve方法，但是此时p1的状态还没有改变，因此p2的状态也不改变。又过了2秒，p1变为Rejected，p2也跟着变为Rejected。

## Promise.prototype.then()
Promise实例具有then方法。也就是说，then方法是定义在原型对象Promise.prototype上的。他的作用是为Promise实例添加状态改变时的回调函数。前面说过，then方法的第一个参数是Resolved状态的回调函数，第二个参数（可选）是Rejected状态的回调函数。

then方法返回的是一个新的Promise实例（不是原来那个）。因此可以采用链式写法，即then方法后面再调用另一个then方法
```javascript
getJSON("/post.json").then(function(json){
	return json.post;
}).then(function(post){
	// ...
});
```
上面的代码使用then方法依次指定了两个回调函数。第一个回调函数完成以后，会将返回结果作为参数传入第二个回调函数。
采用链式的then可以指定一组按照次序调用的回调函数。这时，前一个回调函数有可能返回的还是一个Promise对象（即有异步操作），而后一个回调函数就会等待该Promise对象的状态发生变化，在被调用。
```javascript
getJSON("/post/1.json").then(function(post){
	return getJSON(post.commentURL);
}).then(function funcA(comments){
	console.log("Resolved: ", comments);
}, function funcB(err){
	console.log("Rejected: ", err);
});
```
上面的代码中，第一个then方法指定的回调函数返回的是**另一个Promise对象**。这时，第二个then方法指定的回调函数就会等待这个新的Promise对象状态发生变化。如果变为Resolved，就调用funcA；如果状态变为Rejected，就调用funcB。

用箭头函数写上面的代码
```javascript
getJSON("/post/1.json").then(
	post => getJSON(post.commentURL);
).then(
	comments => console.log("Resolved: ", comments);
	err      => console.log("Rejected: ", err);
)
```

## Promise.prototype.catch()
Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数。
```javascript
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```
上面代码中，getJSON方法返回一个 Promise 对象，如果该对象状态变为resolved，则会调用then方法指定的回调函数；如果异步操作抛出错误，状态就会变为rejected，**就会调用catch方法指定的回调函数**，处理这个错误。另外，then方法指定的回调函数，如果运行中抛出错误，也会被catch方法捕获。

### 1、如果Promise状态已经变成Resolved，再抛出错误是无效的。
```javascript
var promise = new Promise(function(resolve, rejected){
	resolve("ok");
	throw new Error("test");
});
promise
	.then(function(value) {console.log(value)} )
	.catch(function(error) {console.log(error)} )
// ok
```
由于Promise在resolve语句后再抛出错误，并不会被捕获，等于没抛出

### 2、Promise对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获位置。错误总会被下一个的catch语句捕获
```javascript
getJSON("/post/1.json").then(function(post){
	return getJSON(post.commentURL);
}).then(function(comments){
	// some code
}).catch(function(error){
	// 处理前面三个Promise产生的错误
})
```
代码中一共有三个Promise对象：一个由getJSON产生，两个由then产生。其中任何一个抛出的错误都会被最后一个catch捕获

### 3、不要在then方法里定义Rejected状态的回调函数（即then产生的第二个参数），而应该总是使用catch
```javascript
promise
	.then(function(data){
		// success
	}, function(err){
		// error
	});
promise
	.then(function(data){
		// suceess
	})
	.catch(function(err){
		// error
	});
```
第二种方法更好，因为前者更接近同步的写法（try/catch）

### 4、跟传统的try/catch不同的是，如果没哟使用catch方法指定错误处理的回调函数Promise对象抛出的错误不会传递到外层代码，即不会有任何反应。
```javascript
var someAsyncThing = function(){
	return new Promise(function(resolve, reject){
		resolve(x + 2); // x没有声明
	});
};
someAsyncThing().then(function(){
	console.log('everything is great');
});
```
someAsyncThing函数产生的Promise对象会报错，但是由于没有指定catch方法，因而这个错误不会被捕获，也不会传递到外层代码，导致运行后**没有结果**
```javascript
var promise = new Promise(function(resolve, reject){
	resolve("ok");
	setTimeout(function() { throw new Error('test') }, 0);
});
promise.then(function(value){console.log(value)});
// ok
// Uncaught Error: test
```
上面的代码中，Promise指定在下一轮“事件循环”再抛出错误，结果由于没有指定使用try...catch语句，就冒泡到最外层，成了未捕获的错误。因此此时Promise的函数体已经运行结束，所以这个错误是在Promise外抛出的。
Node.js有一个专门监听未捕获的Rejected错误。

### 5、catch方法返回的还是一个Promise对象，因此后面还可以接着调用then方法。
```javascript
var someAsyncThing = function(){
	return new Promise(function(resolve, reject){
		resolve(x + 2); // x未声明
	})；
}；
someAsyncThing()
	.catch(function(error){
		console.log(error)
	})
	.then(function(){
		console.log("carry no")
	});
// [ReferenceError: x is not defined]
// carry on
```
代码中运行完catch方法指定的回调函数，会接着运行后面那个then方法指定的回调函数

### 6、如果没有报错，就会跳过catch方法
```javascript
Promise
	.resolve()
	.catch(function(error){
		console.log(error);
	})
	.then(function(){
		console.log('carry on');
	});
	//carry on
```
代码因为没有报错而跳过了catch方法，直接执行了后面的then方法。此时要是then方法里报错，就与前面的catch无关了

### 7、catch方法中还能再抛出错误
```javascript
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};

someAsyncThing().then(function() {
  return someOtherAsyncThing();
}).catch(function(error) {
  console.log('oh no', error);
  // 下面一行会报错，因为 y 没有声明
  y + 2;
}).then(function() {
  console.log('carry on');
});
// oh no [ReferenceError: x is not defined]
```
上面代码中，catch方法抛出一个错误，因为后面没有别的catch方法了，导致这个错误不会被捕获，也不会传递到外层。如果改写一下，结果就不一样了。
```javascript
someAsyncThing().then(function() {
  return someOtherAsyncThing();
}).catch(function(error) {
  console.log('oh no', error);
  // 下面一行会报错，因为y没有声明
  y + 2;
}).catch(function(error) {
  console.log('carry on', error);
});
// oh no [ReferenceError: x is not defined]
// carry on [ReferenceError: y is not defined]
```
上面代码中，第二个catch方法用来捕获，前一个catch方法抛出的错误。

### Promise.all()
Promise.all方法用于将多个Promise实例包装成一个新的Promise实例
```javascript
var p = Promise.all([p1, p2, p3]);
```
代码中，Promise.all方法接受一个数组作为参数，p1、p2、p3都是Promise对象的实例；如果不是，就会先调用Promise.resolve方法，将参数转为Promise实例，再进一步处理。
Promise.all方法的参数不一定是数组，但是必须具有Iterator接口，且返回的每个成员都是Promise实例
p的状态由p1、p2、p3决定，分为两种情况
>（1）只有p1、p2、p3的状态都变为Fulfilled，p的状态才会变为Fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数
>（2）只要p1、p2、p3中有一个被Rejected，p的状态就变成Rejected，此时第一个被Rejected的实例的返回值会传递给p的回调函数
```javascript
// 生成一个Promise对象的数组
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```
