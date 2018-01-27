---
title: JS函数参数值传递和引用传递 
tags: 前端,JS
---
## 概念
**按值传递**：最常用的求值策略，函数的形参是被调用时所传实参的副本。修改形参的值并不影响形参。
**按引用传递**：实际上是对实参引用变量的复制，导致实参、形参都指向同一个对象实体。形参的改变会同时改变实参的值。
**形参**：在定义函数名和函数体时候使用的参数，目的是用来接收调用该函数时传递的参数。
**实参**：在调用时传递给函数的参数。

##### 按值传递
```javascript
function add(num){
	num++;
	return num;
}

var count = 0;
var result = add(count);    // 按值传递 num=count
console.log(count);         //  还是0
console.log(result);   		// 变为1
```
传递完后两个变量各不相干

##### 按引用传递
```javascript
function setName(obj){
	obj.name = "joe";
}
var person = new Object();
setName(person);
console.log(person.name) // joe
```
当var person = new Object();时
![][1]
当函数setName(person);时，下图可以表示全局变量person和局部变量obj的关系
![][2]
以上代码中创建一个对象，并将其保存在变量person中。然后，这个变量被传递到setName(obj)函数中之后就复制给了obj。在这个函数内部，obj和person引用的是同一个对象。换句话说，即使ECMAScript说这个变量是按值传递的，但obj也会按引用来访问同一个对象。于是，在函数内部为obj添加name属性后，函数外部的person也将有所反应；因为这时的person和obj指向同一个堆内存地址。所以，很多人错误的认为：在局部作用域中修改的对象会在全局对象中反映出来，就说明参数是按引用传递的。

```javascript
function setName(obj){
	obj.name = 'joe';
	obj = new Object(); // 改变obj的指向，此时obj指向一个新的内存地址，不再和person指向同一个
	obj.name = 'wyj';
}
var person = new Object();
setName(person);
console.log(person.name); // joe
```
当创建obj对象obj = new Object();时
![][3]
这个例子与前一个唯一的区别，就是setName()函数中添加了两行代码：obj = new Object();用来改变obj的指向。obj.name = 'wyj'; 用来给新创建的obj添加属性。如果是按引用传递的，那么person就会自动被修改为指向新创建的obj的内存地址，则person的name属性值被修改为wyj。但是，当访问person.name时，显示的结果为joe。这说明即使在函数内部修改了参数的值，但原始的引用仍然保持未变。实际上，当在函数内部重写obj时，这个变量引用的就是一个局部对象了。而这个局部对象会在函数执行完后被立即销毁。



  [1]: ./images/1516983531378.jpg
  [2]: ./images/1516983600958.jpg
  [3]: ./images/1516985452259.jpg