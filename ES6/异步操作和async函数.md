---
title: 异步操作和async函数
tags: 前端
---
### 关于异步
ES6诞生前，异步编程的方法大概有下面4面
* 回调函数
* 事件监听
* 发布\订阅
* Promise对象

ES7中的async函数更是给出了异步编程的终极解决办法。

## 基本概念
#### 异步
所谓“异步”，简单说就是一个任务分成两段，先执行第一段，然后转而执行其他任务，等做好准备再回头执行第二段。
比如，有一个任务是读取文件进行操作，任务的第一段是向操作系统发出请求，要求读取文件。然后，程序执行其他任务，等到操作系统返回文件，再接着执行任务的第二段（处理文件）。这种不连续的执行，就叫做异步。
相应地，连续的执行就叫作同步。由于是连续执行，不能插入其他任务，所以操作系统从硬盘读取文件的这段时间，程序只能等待。

#### 回调函数
JS对异步的实现就是回调函数。所谓回调函数，就是把任务的第二段单独写在一个函数中，等到重新执行该任务时直接调用这个函数。callback直译过来就是“重新调用”

读取文件例子：
```javascript
fs.readFile('xxoo', function(err, data){
	if(err){
		throw err;
	}
	console.log(data);
})
```
上面代码中。readFile函数的第二个参数就是回调函数，也就是任务的第二段。等到操作系统返回了xxoo这个文件以后，回调函数才会执行。

一个有趣的问题，为什么Node.js约定回调函数的第一个参数必须是错误对象err（如果没有错误，该参数就是null），原因就是执行分成两段，在这两段之间抛出的错误程序无法捕获，只能当作参数传入第二段。

#### Promise
回调函数本身并没有问题，问题出在多重嵌套，会出现回调函数地狱
Promise就是为了解决这个问题而提出的。它不是新的语法功能，而是一个新的写法，允许将回调函数的横向加载改成纵向加载。采用Promise，连续读取多个文件的写法如下：
```javascript
var readFile = require('fs-readFile-promise');
readFile(fileA)
.then(function(data){
	console.log(data.toString);
})
.then(function(){
	return readFile(fileB);
})
.then(function(data){
	console.log(data.toString());
})
.then(function(err){
	console.log(err);
});
```
上面的代码中使用了fs-readfile-promise模块，其作用是返回一个Promise版本的readFile函数。Promise提供then方法加载回调函数，catch方法捕捉执行过程中抛出的错误。

Promise的最大问题是代码冗余，原来的任务被Promise包装了一下，不管什么操作，一眼看去都是一堆then，原来语义变得不是很清楚
有木有更嗨的写法呢？

## Generator函数
#### 协程
传统的编程语言早已有异步编程的解决方案（尤其是多任务的解决方案）。其中有一种叫做“协程”，意思是多个线程互相协作，完成异步任务。
协程有点像函数，又有点像线程。其运行流程大致如下。
* 1、协程A开始执行
* 2、协程A执行到一半，暂停，执行权转移到协程B
* 3、（一段时间后）协程B交还执行权
* 4、协程A恢复执行
上面的协程A就是异步任务，因为它分成两段执行。
举例来说，读取文件的协程写法如下。
```javascript
fucntion asyncJob() {
	// ...其他代码
	var f = yield readFile(fileA);
	// ...其他代码
}
```
上面的函数asyncJob是一个协程，他的奥妙在于其中的yield。他表示执行到此处执行权将交给其他协程。也就是说，yield命令是异步两个阶段的分界线。
协程遇到yield命令就暂停，等到执行权返回，再从暂停的地方继续往后执行。他的最大优点就是代码的写法非常像同步操作，如果去除yield命令，则完全一样。

#### Generator函数的概念
Generator函数是协程在ES6中的实现，最大的特点就是可以交出函数的执行权（即暂停执行）。
整个Generator函数就是一个封装的异步任务，或者说是异步任务的容器。异步操作需要暂停的地方，都用yield语句注明。Generator函数的执行方法如下。
```javascript
function* gen(x){
	var y = yield x + 2;
	return y;
}
var g = gen(1);
g.next()// {value: 3,         done: false}
g.next()// {value: undefined, done: true}
```
上面的代码中，调用Generator函数会返回一个内部指针（及遍历器）g。这是Generator函数不同于普通函数的另一个地方，即执行它不会返回结果，返回的是指针对象。调用指针g的next方法，会移动内部指针（即执行异步任务的第一段），指向第一个遇到的yield语句，上例中是到 x + 2为止。
换言之，next方法的作用是分阶段执行Generator函数。每次调用next方法，会返回一个对象，表示当前阶段的信息（value属性和done属性）。value属性是yield语句后表达式的值，表示当前阶段的值；done属性是一个布尔值，表示Generator函数是否执行完毕，即是否还有下一个阶段。

#### Generator函数的数据交换和错误处理
Generator函数可以暂停执行和恢复执行，这是它能封装异步任务的根本原因。除此之外它还有两个特性使它可作为异步编程的完整解决方案：函数体内外的数据交换和错误处理机制。
next方法返回值的value属性，是Generator函数向外输出数据；next方法还可以接受参数，这是向Generator函数体内输入数据。
```javascript
function* gen(x){
	var y = yield x + 2;
	return y;
}
var g = gen(1);
g.next()  // { value: 3, done: false }
g.next(2) // { value: 2, done: true }
```
第二个next参数2，这个参数可以传入Generator函数，作为上个阶段异步任务的返回结果被函数体内的变量y接收。因此，这一步的value属性返回的就是2（变量y的值）。
Generator函数内部还可以部署错误处理代码，捕获函数体外抛出的错误。
```javascript
function* gen(x){
	try{
		var y = yield x + 2;
	}catch(e){
		console.log(e);
	}
	return y;
}
var g = gen(1);
g.next();
g.throw('出错了');
// 出错了
```
上面的最后一行，Generator函数体外使用指针对象的throw方法抛出的错误，可以被函数体内的try...catch代码块捕获。这意味着，出错的代码与处理错误的代码实现了时间和空间上的分离，这对于异步编程来说无疑是很重要的。

#### 异步任务的封装
如何使用Generator函数执行一个真实的异步任务
```javascript
var fetch = require('node-fetch');
function* gen(){
	var url = "https://xxoo";
	var result = yield fetch(url);
	console.log(result.xxoo);
}
```
Generator函数封装了一个异步操作，先读取一个远程接口，然后从JSON格式的数据解析信息。就像前面说的，这段代码非常像同步操作，只是加上了yield命令。

执行方法如下
```javascript
var g = gen();
var result = g.next();

result.value.then(function(data){
	return data.json();
}).then(function(data){
	g.next(data);
})
```
首先执行Generator函数获取遍历器对象，然后使用next方法（第二行）执行异步任务的第一阶段。由于Fetch模块返回的是一个Promise对象，因此要用then方法调用下一个next方法。
可以看到，虽然Generator函数将异步操作表示得很简洁，但是流程管理（即何时执行第一阶段，何时执行第二阶段）却不方便。

## Thunk函数
#### 参数的求值策略
```javascript
var x = 1;
function f(m){
	return m * 2;
}
f(x + 5)  // 12
```
1、先定义函数f
2、然后向他传入表达式x+5
f(x + 5)传值调用时等同于f(6)

**“传值调用”**
进入函数体前就计算x+5的值，再将这个值传入函数f。

**“传名调用”**
直接将表达式x+5传入函数体，只在用到它时求值。

#### Thunk函数的含义
编译器的“传名调用”实现往往是先将参数放到一个临时函数中，再将这个临时函数传入函数体。这个临时函数就叫做Thunk函数。
```javascript
function f(m){
	return m * 2;
}
f(x + 5);
//等同于
var thunk = function(){
	return x + 5;
};
function f(thunk){
	return thunk() * 2;
}
```
代码中，函数f的参数x+5被一个函数替换了。凡是用到原参数的地方，对Thunk函数求值即可。这就是Thunk函数的定义，他是“传名调用”的一种实现策略，用来替换某个表达式。

#### JS语言的Thunk函数
JS是传值调用，他的Thunk函数含义有所不同。在JS语言中，Thunk函数替换的不是表达式，而是多参数函数，它将其替换成单参数的版本，且只接受回调函数作为函数。
```javascript
// 正常的readFile 多参数
fs.readFile(fileName, callback);

// Thunk版本的readFile 单参数
var readFileThunk = Thunk(fileName);
readFileThunk(callback);
var Thunk = function(fileName){
	return function(callback){
		return fs.readFile(fileName, callback);
	};
};
```
代码中，fs模块的readFile方法是一个多参数函数，两个参数分别为文件名和回调函数。经过转换器处理，它变成了一个单参数函数，只接受回调函数作为参数。这个单参数版本，就叫做Thunk函数。

任何函数，**只要参数有回调函数，就能写成Thunk函数的形式**。下面是一个简单的Thunk函数转换器。
```javascript
var Thunk = function(fn){
	return function(){
		var args = Array.prototype.slice.call(arguments);
		return function(callback){
			args.push(callback);
			return fn.apply(this, args);
		}
	};
};
```
使用上面转换器生成fs.readFile的Thunk函数如下
```javascript
var readFileThunk = Thunk(fs.readFile);
readFileThunk(fileA)(callback);
```

#### Thunkify模块
用于**生产环节**的转换器，建议使用Thunkify模块
用法：
```javascript
var thunkify = require('thunkify');
var fs = require('fs');
var read = thunkify(fs.readFile);
read('package.json')(function(err, str){
	// .....
});
```

#### Generator函数的流程管理
Thunk函数可以用于Generator函数的自动流程管理。
读取文件为例，下面的Generator函数封装了两个异步操作。
```javascript
var fs = require('fs');
var thunkify = require('thunkify');
var readFile = thunkify(fs.readFile);
var gen = function* (){
	var r1 = yield readFile('xx');
	console.log(r1.toString());
	var r2 = yield readFile('oo');
	console.log(r2.toString());
};
```
代码中，yield命令用于将程序的执行权移出Generator函数。就需要一种方法，将执行权再交还给Generator函数。

#### Thunk函数的自动流程管理
Thunk函数真正的威力在于可以自动执行Generator函数。下面就是一个基于Thunk函数的Generator执行器
```javascript
function run(fn){
	var gen = fn();
	function next(err, data){
		var result = gen.next(data); // 先next
		if(result.done){   // 判断结束
			return;
		}
		result.value(next); // 没结束 next函数传入Thunk函数
	}
	next();  // 递归执行
}
run(gen);
```
上面的run函数就是一个Generator函数的自动执行器。内部的next函数就是Thunk的回调函数。next函数先将指针移到Generator函数的下一步（gen.next方法），然后判断Generator函数是否结束（result.done属性），如果没有结束，就将next函数再传入Thunk函数（result.value属性），否则直接退出。

有了这个执行器，执行Generator函数就方便多了。不管有多少个异步操作，直接传入run函数即可。当然，前提是每一个异步操作都要是Thunk函数。也就是说，跟yield命令后面的必须是Thunk函数。
```javascript
var gen = function* (){
	var f1 = yield readFile('fileA');
	var f2 = yield readFile('fileB');
	// ...
	var fn = yield readFile('fileN');
};
run(gen);
```
上面的代码中，函数gen封装了n个异步的读取文件操作，只要执行run函数，这些操作就会自动完成。这样一来，异步操作不仅可以写的像同步操作，而且一行代码就可以执行。
Thunk函数并不是Generator函数自动执行的唯一方案。因为自动执行的关键是，必须有一种机制自动控制Generator函数的流程，接收和交换程序的执行权。回调函数可以做到这一点，Promise对象也可以做到这一点。

## co模块
co模块用于Generator函数的自动执行。
```javascript
var gen = function* (){
	var f1 = yield readFile('xx');
	var f2 = yield readFile('oo');
	console.log(f1.toString());
	console.log(f2.toString());
};
```
co模块可以让你不用编写Generator函数的执行权
```javascript
var co = require('co');
co(gen);
```
上面的代码中，Generator函数只要传入co函数就会自动执行。
co函数返回一个Promise对象，因此可以用then方法添加回调函数。
```javascript
co(gen).then(function(){
	console.log('Generator函数执行完成');
})
```
上面的代码中，等到Generator函数执行结束，就会输出一行提示。

## async函数
#### 含义
ES7提供了async函数，使得异步操作变得更加方便。async函数是什么？一句话，async函数就是Generator函数的语法糖。
Generator函数，依次读两个文件
```javascript
var fs = require('fs')
var readFile = function(fileName){
	return new Promise(function (resolve, reject){
		fs.readFile(fileName, function(error, data){
			if(error){
				reject(error);
			}
			resolve(data)
		});
	});
};
var gen = function* (){
	var f1 = yield readFile('xx');
	var f2 = yield readFile('oo');
	console.log(f1.toSrting());
	console.log(f2.toSrting());
}

// 写成async函数如下
var asyncReadFile = async function(){
	var f1 = await readFile('xx');
	var f2 = await readFile('oo');
	console.log(f1.toString());
	console.log(f2.toString());
}
```
比较会发现，async函数就是将Generator函数的星号替换成async，将yield替换成await，仅此而已。
async函数对Generator函数的改进体现在以下4点。
1、**内置执行器**。Generator函数的执行必须靠执行器，所以才有co模块，而async函数自带执行器。也就是说，async函数的执行与普通函数一模一样，只要一行
var result = asyncReadFile();
2、上面代码调用了asyncReadFile函数，然后它就**会自动执行**，输出最后结果。完全不像Generator函数，需要调用next函数的执行与普通函数一模一样，只要一行
3、更好的语义。async和await比起星号和yield，语义更清楚。**async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果**。
4、更广的适用性。co模块约定，yield命令后面只能是Thunk函数或Promise对象，而async函数的**await命令后面可以是Promise对象和原始类型的值**（数值、字符串和布尔值，但这是等同于同步操作）
5、**返回值是Promise**。async函数的返回值是Promise对象，这比Generator函数的返回值是Iterator对象方便多了。可以用then方法指定下一步的操作。

进一步，async函数完全可以看作由多个异步操作包装成的一个Promise对象，而await命令就是内部then命令的语法糖。

#### async函数的实现
async函数的实现就是将Generator函数和自动执行器包装在一个函数中。
```javascript
async function(){
	// ...
}
// 等同于
function fn(args){
	return spawn(fucntion* (){
		// ...
	});
}
```
所有的async函数都可以写成上面的第二种形式，其中spawn函数就是自动执行器

#### async函数的用法
同Generator函数一样，async函数返回一个Promise对象，可以使用then方法添加回调函数。当函数执行时，一旦遇到await就会先返回，等到触发的异步操作完成，再接着执行函数体内后面的语句。
```javascript
async function getStockPriceByName(name){
	var symbol = await getStockSymbol(name);
	var stockPrice = await getStockPrice(symbol);
	return stockPrice;
}
getStockPriceByName('goog').then(function(result){
	console.log(result);
});
```
代码是一个获取股票报价的函数，函数前的async关键字表明该函数内部有异步操作。调用时，会立即返回一个Promise对象。

下例指定了多少秒后返回一个值：
```javascript
function timeout(ms){
	return new Promise((resolve) => {
		setTimeout(resolve, ms);
	});
}
async function asyncPrint(value, ms){
	await timeout(ms);
	console.log(value);
}
asyncPrint('hello world', 50);
```

#### 注意点
await命令后面的Promise对象，运行结果可能是Rejected，所以最好把await命令放在try...catch代码块中
```javascript
async function myFunction(){
	try{
		await somethingThatReturnsAPromise();
	}catch(err){
		console.log(err);
	}
}

// 另一种写法

async function myFunction(){
	await somethingThatReturnsAPromise().catch(function(err){
		console.log(err);
	});
}
```
await命令只能用在async函数中，在普通函数中会报错

#### 与Promise、Generator的比较
一个DOM操作的动画效果，前一个开始后一个结束。
Promise的写法
```javascript
function animate(elem, animations){
	// 变量ret用来保存上一个动画的返回值
	var ret = null;
	// 新建一个空的返回值
	var p = Promise.resolve();
	// 使用then方法添加所有动画
	for(var anim in animations){
		p = p.then(function(val){
			ret = val;
			return anim(elem);
		})
	}
	// 返回一个部署了错误捕捉机制的Promise
	return p.catch(function(e){
		// 忽略错误，继续执行
	}).then(function(){
		return ret;
	});
}
```
比回调有很大改进，但是都是Promise的API，操作本身的语义不容易看出来
接着是Generator函数的写法：
```javascript
function animate(elem, animations){
	return spawn(function* (){
		var ret = null;
		try{
			for(var anim of animations){
				ret = yield anim(elem);
			}
		}catch(e){
			// 忽略错误继续执行
		}
		return ret;
	});
}
```
代码使用Generator函数遍历了每个动画，语义比Promise写法更清晰，用户定义的操作全部出现在spawn函数的内部。问题在于必须有一个任务运行自动执行Generator函数，上面的spawn函数就是自动执行器，它返回一个Promise对象，而且保证yield语句后面的表达式必须返回一个Promise。
async函数的写法
```javascript
async function animate(ele, animations){
	var ret = null;
	try{
		for(var anim of animations){
			ret = await anim(elem);
		}
	}catch(e){
		// 忽略错误 继续执行
	}
	return ret;
}
```
这个太好了