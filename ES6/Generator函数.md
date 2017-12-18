---
title: Generator函数
tags: 前端,ES6
---

## 简介
>1、Generator函数是ES6提供的一种异步编程方案
>2、从语法上，首先可以把它理解成一个状态机，封装了多个内部状态
>3、Generator函数还是一个遍历器对象生成函数，返回遍历器对象
>4、Generator函数有两个特征：1、function命令与函数名之间有一个星号；2、函数体内部使用yield语句定义不同的内部状态（“yield”在英语里的意思是“产出”）

```javascript
function* helloWorldGenerator(){
	yield 'hello';
	yield 'world';
	return 'ending';
}
var hw = helloWorldGenerator();
```
定义了一个Generator函数——helloWorldGenerator，它内部有两个yield语句“hello”和“world”，即该函数有三个状态：hello、world和return语句（结束执行）
在调用Generator函数后，**该函数并不执行**，**返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是遍历器对象**。
下一步必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或从函数上一次停下来的地方开始执行，直到遇到下一条yield语句（或return语句）为止。换言之，**Generator函数是分段执行的，yield语句是暂停执行的标记，而next方法可以恢复执行**
```javascript
hw.next() // {value: 'hello', done: false }
hw.next() // {value: 'world', done: false }
hw.next() // {value: 'ending', done: true }
hw.next() // {value: undefined, done: true }
```
ES6没有规定function关键字与函数名之间的星号必须在哪

### yield语句
由于Generator函数返回的遍历器对象只有调用next方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数。yield语句就是暂停标志。
遍历器对象的next方法的运行逻辑如下：
>1、遇到yield语句就暂停执行后面的操作，并将紧跟在yield后的表达式的值作为返回的对象的value属性值
>2、下一次调用next方法时才会继续执行，直到遇到下一个yield语句。
>3、如果没有再遇到新的yield语句，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值作为返回的对象的value属性值。
>4、如果该函数没有return语句，则返回的对象的value属性值为undefined。

要注意的是，yield语句后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行，因此等于为JS提供了手动的“惰性求值”的语法功能。
```javascript
function* gen(){
	yield 123+456;
}
```
上面的代码中，yield后面的表达式123+456不会立即求值，只会在next方法将指针移到这一语句时才求值

Generator函数可以不用yield语句，这时就变成了一个单纯的暂缓执行函数。
```javascript
function* f(){
	console.log('执行了');
}
var generator = f();
setTimeout(function(){
	generator.next();
}, 2000);
```
如果f只是普通函数，在为变量generator赋值时就会执行。但是函数f是一个Generator函数，于是就变成只有调用next方法时才会执行。
要注意yield语句不能再普通函数中，yield语句在表达式中，必须放在括号里

## next方法
next方法可以携带一个参数：
```javascript
function* f(){
	for(var i=0; true; i++){
		var reset = yield i;
		if(reset) {i = -1};
	}
}
var g = f();
g.next() // {value: 0, done: false}
g.next() // {value: 1, done: false}
g.next(true) // {value: 0, done: false}
```
定义了一个可以无限循环的Generator函数f，如果next方法没有参数，每次运行到yield语句，变量reset就会被重置为这个参数，因而i会等于-1，下一轮循环就从-1开始递增。

Generator函数从暂停到恢复运行，其上下文状态是不变的。通过next方法的参数就有办法在Generator函数开始运行后继续向函数提内部注入值。也就是说，可以在Generator函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。
```javascript
function* foo(x){
	var y = 2 * (yield(x + 1));
	var z = yield( y / 3 );
	return (x + y + z);
}
var a = foo(5);
a.next()//Object{value:6, done:false}
a.next()//Object{value:NaN, done:false}
a.next()//Object{value:NaN, done:false}

var b = foo(5);
b.next()//Object{value:6, done:false}
b.next(12)//Object{value:8, done:false}
b.next(13)//Object{value:42, done:false}
```
如果next中没有参数，yield就返回undefined。所以在a第二次调next是y=2\*undefined，所以结果为NaN。
在实例b中，第一次yield返回6。第二次传入12，所以第二次返回的值为2\*12再除以3，所以结果为8.第三次同理
next参数就是上一条yield的返回值，所以第一次yield时不能带有参数。V8引擎会忽略第一次next时的参数，只从第二次使用next方法开始参数才是有效的。从语义上讲，第一个next方法用来启动遍历器对象，所以不用带有参数。如果想要第一次next带有参数，需要在Generator函数外再包一层。
```javascript
function wrapper(generatorFunction){
	return function(...args){
		let generatorObject = generatorFunction(...args);
		generatorObject.next();
		return generatorObject;
	};
}
const wrapped = wrapper(function* () {
	console.log('First input: ${yield}');
	return 'DONE';
});
wrapped().next('hello!');
```
如果Generator函数不用wrapper先包一层，是无法第一次调用next方法就输入参数的。

## for...of
**for...of循环可以自动遍历Generator函数，且此时不再需要调用next方法。**
```javascript
function *foo(){
	yield 1;
	yield 2;
	yield 3;
	yield 4;
	yield 5;
	return 6;
}

for(let v of foo(){
	console.log(v);
})
//1 2 3 4 5
```
for...of遇到done属性为true就会终止，且不包含该返回对象。所以return的6就不包括在for...of中

## Generator与协程
协程是一种程序运行的方式，可以理解成“协作的线程”或“协作的函数”。协程可以用单线程实现，也可以用多线程实现；前者是一种特殊的子线程，后者是一种特殊的线程。

#### 协程与子例程的差异
传统的“子例程”采用堆栈式“先进后出”的概念实现，只有当调用的子函数完全执行完毕，才会结束执行父函数。协程与其不同，多个线程可以并行执行，但只有一个线程处于正在运行状态，其他线程都处于暂停态，线程之间可以交换执行权。也就是说，一个线程执行到一半，可以暂停执行，将执行权交给另一个线程，等到稍后回收执行权时再恢复执行。这种可以并行执行，将执行权交给另一个线程，等到稍后收回执行权时再恢复执行。这种可以并行执行、交换执行权的线程，就成为协程。
从实现上看，在内存中子例程只使用一个栈，而协程是同时存在多个栈，但只有一个栈是在运行态。也就是说，协程是以多占用内存为代价实现多任务的并行运行。

#### 协程与普通线程的差异
不难看出，协程适用于多任务运行的环境。在这个意义上，它与普通的线程很相似，都有自己执行的上下文，可以分享全局变量。他们不同之处在于，同一时间可以有多个线程是抢占式的，到底哪个线程优先得到资源，必须由运行环境决定，但是协程是合作式的，执行权由协程自己分配。
ECMAScript是单线程语言，只能保持一个调用栈。引入协程以后，每个任务可以保持自己的调用栈。这样做的最大好处，就是抛出错误时可以找到原始的调用栈，不至于像异步操作的回调函数那样，一旦出错原始的调用栈早已结束。
Generator函数是ES6对协程的实现，但属于不完全实现。Generator函数被称为“半协程”，意思是只有Generator函数的调用者才能将程序的执行权还给Generator函数。如果是完全实现的协程，任何函数都可以让暂停的协程继续执行。
如果将Generator函数当作协程，完全可以将多个需要互相协作的任务写成Generator函数，它们之间使用yield语句交换控制权。

## 应用
Generator可以暂停函数执行，返回任意表达式的值。这种特点使得Generator有多种应用场景。

#### 异步操作的同步化表达
Generator函数的暂停执行效果，意味着可以把异步操作卸载yield语句里面，等到调用next方法时再执行。所以，Generator函数的一个重要实际意义就是用于**处理异步操作，改写回调函数**
```javascript
function* loadUI () {
	showLoadingScreen();
	yield loadUIDataAsynchronously();
	hideLoadingScreen();
}
var loader = loadUI();
//加载UI
loader.next();
//卸载UI
loader.next();
```
上面的代码表示，第一次调用loadUI函数时，该函数不会执行，仅返回一个遍历器。下一次对该遍历器调用next方法，**则会显示加载界面，并且异步加载数据**。等到数据加载完成，再一次使用next方法，则会隐藏加载界面。可以看到，这种写法的好处是所有加载界面的逻辑都被封装在一个函数中，按部就班非常清晰。

Ajax是典型的异步操作，通过Generator函数部署Ajax操作，可以用同步的方式表达。
```javascript
function* main(){
	var result = yield request("http://xxoo");
	var resp = JSON.parse(result);
	console.log(resp.value);
}
function request(url){
	makeAjaxCall(url, function(response){
		it.next(response);
	});
}
var it = main();
it.next();
```
上面的main函数就是通过AJAX操作获取数据。可以看到，除了多了一个yield，它几乎与同步操作的写法一模一样。注意，makeAjaxCall函数中的next方法必须加上response参数，因为yield语句构成的表达式本身是没有值的，总是等于undefined。

通过Generator实现的逐行读取文本文件。
