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
如果f只是普通函数，在为变量generator