---
title: Iterator和for...of循环
tags: 前端,ES6
---

### Iterator 遍历器
遍历器（Iterator）为各种不同的数据结构提供统一的访问机制。任何数据结构，只要部署Iterator接口，就可以完成遍历操作。
>Iterator作用：
>1、为各种数据结构提供一个统一的、简便的访问接口
>2、使数据结构的成员能够按某种次序排列
>3、ES6创造了一种新的遍历命令——for...of循环，Iterator接口主要共for...of消费

Ierator遍历过程：
1、创建一个指针对象，指向当前数据结构的起始位置。遍历器对象本质上就是一个指针对象

2、第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员

3、第二次调用指针对象的next方法，指针就指向数据结构的第二个成员

4、不断调用指针对象的next方法，直到它指向数据结构的结束位置。

每一次调用next方法，都会返回数据结构当前成员的信息。具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束

```javascript
let it = makeIterator(['a', 'b']);
it.next(); //value:a,done:false
it.next(); //value:b,done:false
it.next(); //value:undefined,done:true
function makeIterator(array){
	var nextIndex = 0;
	return {
		next: function(){
			return nextIndex < array.length ?
			{value: array[nextIndex++], done: false} :
			{value: undefined, done: true}
		}
	}
}
```
函数返回一个遍历器对象（即指针对象）

ES6中有些数据结构具备原生的Iterator接口（比如数组），不用任何处理就可以被for..of循环遍历，有些就不行（比如对象）。原因在于，这些数据结构原生部署了**Symbol.iterator**属性。凡部署了Symbol.iterator属性的数据结构，就称为部署了遍历器接口。调用接口就会返回遍历器对象。
ES6中，数组、Set、Map具有原生数据结构
Symbol.iterator本身是一个表达式，返回Symbol对象的iterator属性，这是一个预定义好的、类型为Symbol的特殊值，所以要放到方括号中。
```javascript
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();

iter.next() //{value: 'a', done: false};
iter.next() //{value: 'b', done: false};
iter.next() //{value: 'c', done: false};
iter.next() //{value: undefined, done: true};
```

### 调用Iterator接口的场合
**有一些场合会默认调用Iterator接口(即Symbol.iterator方法)**
#### 1、解构赋值
对数组和Set解雇进行解构赋值时，会默认调用Symbol.iterator方法
```javascript
let set = new Set().add('a').add('b').add('c');
let [x, y] = set;  //x='a' y='b'
let [first, ...rest] = set;   //first='a' second=['b', 'c']
```

#### 2、扩展运算符
扩展运算符(...)也会调用默认的Iterator接口
```javascript
var str = "hello";
[...str] //['h', 'e', 'l', 'l', 'o']

let arr = ['b', 'c'];
['a', ...arr, 'd']   //['a', 'b', 'c', 'd']
```
上面的扩展运算符内部就调用了Iterator接口
**实际上，这提供了一种机制，可以将任何部署了Iterator接口的数据结构转化为数组。也就是说，只要某个数据结构部署了Iterator接口，就可以对它使用扩展运算符，将其转为数组。**
```javascript
let arr = [...iterable];
```

#### 3、yield*
yield\*  后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口
```javascript
let generator = function* () {
	yield 1;
	yield* [2, 3, 4];
	yield 5;
};

var iterator = generator();
iterator.next();  //{ value: 1, done: false }
iterator.next();  //{ value: 2, done: false }
iterator.next();  //{ value: 3, done: false }
iterator.next();  //{ value: 4, done: false }
iterator.next();  //{ value: 5, done: false }
iterator.next();  //{ value: 6, done: false }
iterator.next();  //{ value: undefined, done: true }
```
#### 4、其他场合
由于数组的遍历会调用遍历器接口，所以任何接受任何数组作为参数的场合其实都调用了遍历器接口。
* for...of
* Array.from()
* Map()、Set()、WeakMap()和WeakSet()（比如new Map([['a', 1], ['b', 2]])）
* Promise.all()
* Promise.race()

### **for...of**
**一个数据结构只要部署了Symbol.iterator属性，就被视为具有Iterator接口，就可以用for...of循环遍历它的成员。**

#### 1、数组
数组原生具备Iterator接口，for...of循环本质上就算是调用这个接口产生的遍历器
>```javascript
>const arr = ['red', 'green', 'blue'];
>let iterator = arr[Symbol.iterator]();
>for(let v of arr){
>  console.log(v);   // red green  blue
>}
>for(let v of iterator){
>  console.log(v);   // red green blue
>}
>```
以上两种方法等价 
JS原有的for...in循环，只能获得对象的键名，不能直接获取键值。ES6提供for...of循环，允许遍历获得键值
```javascript
var arr = ['a', 'b', 'c', 'd'];
for(let a in arr){
	console.log(a);  //0 1 2 3  键名
}
for(let a of arr){
	console.log(a);  //a b c d  键值
}
```
2、Set和Map结构
Set和Map结构也原生具有Iterator接口，可以直接使用for...of循环
（1）遍历的顺序是按照各个成员被添加进数据结构的顺序
（2）Set结构遍历时返回的是一个值，而Map结构遍历时返回的是一个数组，该数组的两个成员分别为当前Map成员的键名和键值。

3、计算生成的数据结构
有些数据结构是现有数据结构基础上计算生成的，比如以下3个方法调用后
* entries()返回遍历器对象，用于遍历【键名， 键值】组成的**数组**
* keys()返回一个遍历器对象，用于遍历所有的**键名**
* values()返回一个遍历器对象，用于遍历所有的**键值**

4、类似数组的对象
比如字符串、DOM NodeList对象、arguments对象
但并不是所有类似数组的对象都具有Iterator接口。一个简便的方法是使用Array.from方法将其转化为数组
```javascript
let arrayLike = {length: 2, 0: 'a', 1: 'b'};
for(let x of arrayLike){
	console.log(x);
}                  // 报错
for(let x of Array.from(arrayLike)){
	console.log(x);
}
```

5、对象
对于普通的对象，for...of结构不能直接使用，会报错，必须部署了接口才能使用。但是，这样的情况下，for...in循环依然可以用于遍历键名。
```javascript
var es6 = {
	edition: 6,
	committee: "TC39",
	standard: "ECMA-262"
};

for(e in es6){
	console.log(e);
}// edition ommittee standard

for(e of es6){
	console.log(e);
}// error es6 is not iterable
```
解决办法：
1、使用Object.keys方法将对象的键名生成一个数组，然后遍历这个数组
```javascript
for(var key of Object.keys(someObj)){
	console.log(key + ":" + someObj[key])
}
```
2、在对象上部署Iterator接口的代码，可以直接赋值给其他对象的Symbol.iterator属性
```javascript
// 让for...of循环遍历jQuery对象
jQuery.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator]
```
3、使用Generator函数将对象重新包装
```javascript
function* entries(obj){
	for(let key of Object.keys(obj)){
		yield [key, obj[key]];
	}
}
for(let [key, value] of entries(obj)){
	console.log(key, "->", value);
}// a->1  b->2  c->3
```
>for...in的缺点
>1、数组的键名是数字，但是for...in循环是以字符串作为键名，“0”、“1”、“2”
>2、for...in循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键
>3、某些情况下 ，for...in循环主要是**遍历对象而设计的，不适合遍历数组**

>for...of的优点
>1、有着同for...in一样的简介语法，但是没有for...in那些缺点
>2、不同于forEach方法，它可以与break、continue、和return配合使用
>3、提供了遍历所有数据结构的统一操作接口