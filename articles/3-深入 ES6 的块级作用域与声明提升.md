# 深入 ES6 的块级作用域与声明提升

> 全文共 3038 字，建议阅读时间 20 mins

众所周知，ES6 的 let 与 const 给了 Javascript 之前没有的块作用域，但是却很少人深入其中探讨块级作用域背后的概念，以及变量提升的一些细节，下面记录了我深入了解的过程。

*如果已经了解 var 与 let、const 区别的同学可以直接跳到 [正文](#深入块级作用域与声明提升)*

## 背景知识——var 与 let、const 的区别

### 作用域

var 的作用域存在于**函数级**，或**全局级**：

```js
for(var i=0;i<10;i++) {
  // do something
}
console.log(i);	// 10
```

let、const 的作用域是**块级**的：

```js	
for(let i=0;i<10;i++) {
  // do something
}
console.log(i);	// ReferenceError: i is not defined
```

### 变量声明提升

使用 var 声明的变量会被提升声明至作用域的头：

```js
console.log(a);	// undefined
var a = 1;

// 只有声明会被提升，上述代码可以看成：
var a;
console.log(a);
a = 1;
```

使用 let、const 的变量不会被提升声明：

```js
console.log(typeof a);	// ReferenceError: a is not defined
let a = 1;
```

### 重复声明

var 允许同一个变量被重复声明：

```js
var a = 1;
var a = 2;
console.log(a);	// 2
```

let、const 重复声明变量会导致报错：

```js
let a = 1;
let a = 2;	// SyntaxError: Identifier 'a' has already been declared
```

### 全局作用域的绑定

在全局作用域下使用 var 会绑定全局变量（在浏览器下是 window）：

```js
var a = 1;
console.log(window.a);	// 1

// 但是这个特性可能导致 window 下一些 writable 为 true 的属性被覆写：
var RegExp = 1;
console.log(window.RegExp);	// 1
```

反之 let、const 不会有这个问题：

```js
var a = 1;
console.log(window.a);	// undefined
```

当然，这并不是说 var 的这个特性就是不好的，如果你有跨 window 或者 frame 之间调用的需求，使用 var 也是一个选项。至于是不是最优解就超出本章的范围了。

### let 与 const 的区别

使用 let 声明的变量可以被更新引用：

```js
let a = 1;
a = 2;
console.log(a); // 2
```

反之，用 const 声明的变量不允许修改引用（但可以修改引用下的值）：

```js
const a = 1;
a = 2;	// TypeError: Assignment to constant variable.
  
const b = {
  foo: 1
}
b.foo = 2;
console.log(b.foo);		// 2
b = {};		// TypeError: Assignment to constant variable.
```



## 深入块级作用域与声明提升

### 暂时性死区 TDZ

介绍 TDZ 之前，我们先来看几个例子：

```js
// 例子1: 声明 let 前使用
{
  console.log(a);		// ReferenceError: Cannot access 'a' before initialization
  let a = 1;
}

// 例子2: 声明 const 前使用
{
  console.log(b);		// ReferenceError: Cannot access 'b' before initialization
  const b = 2;
}

// 例子3: 使用未声明变量
{
  console.log(c);		// ReferenceError: c is not defined
}
```

细心的同学应该已经发现了，例子 1 跟例子 2 的报错跟例子 3 是不一样的。可以看到，如果照着我们之前的思想，let 跟 const 是不会被声明提升的话，那么报错应该是 `xxx is not defined`，但是却报了一个 `Cannot access xxx before initialization`，换句话说，变量已经存在了，只是 “未初始化” 罢了，为什么呢？

<br />

要回答这个问题，首先要回答**为什么 var 会有变量声明提升？**

> 学过编译原理的同学应该知道，在一个语言被编译的时候，会先经过一系列的 `词法分析`，`语法分析` 等等的步骤，最后生成一个 `抽象语法树 AST`。而 JS 的引擎在生成 AST 的时候，会给变量声明保留一些空间（讲白话点就是分配内存）。对于 var 来说，JS 引擎在分配空间的时候会顺便给它赋值为 `undefined`。这个就是声明提升的由来。

<br />

回到 let 与 const 的例子，现在我们可以回答之前的问题了：

>  let 与 const 也被**声明提升**了，只是区别在于，他们被分配完空间之后没有被初始化为 `undefined`

我们知道，以 C++ 为例，声明了一个变量而没有初始化是一个多么危险的事情，严重点甚至可以造成内存溢出，所以这也是 JS 引擎在上述情况直接报错的原因了。

<br />

有了上述背景知识之后，我们可以来解释所谓的 TDZ 到底是什么了：

> TDZ 就是从 JS 引擎初始化作用域为 let/const 的变量分配完空间之后，到尚未进行词法绑定（let/const 语句实际上被执行完）的这段时间。

从结果上来说，我们可以这么认为：

```js	
{	// TDZ starts at beginning of scope
  const foo = () => console.log(a);	// ok
  let a = 1;	// TDZ ends for variable "a"
  foo();		// 1
}
```

可以看到，我们的变量 `a` 直到第 3 行运行结束才被真正的赋值完毕，而在 TDZ 之后执行的语句不会报错。

<br />

也正是有 TDZ 这个特性，运算符 `typeof` 在使用了 let 的时候将变得不再安全：

```js
// 例子1: 使用未声明的变量只会返回 undefined
{
	console.log(typeof foo);	// undefined
}

// 例子2: 使用 let 声明前的变量会直接报错
{
  console.log(typeof bar);		// ReferenceError
  let bar;
}
```

### for 循环中的 let 和 const

要解释循环中 let 与 const 是怎么维护块级作用域的，就必须请出一个老生常谈的面试题了：

```js
var funcs = []
for(var i = 0;i<3;i++) {
  funcs[i] = function(){
    console.log(i)
  }
}
funcs[0]();	// 3
console.log(i);	// 3
```

显而易见，会造成这个的原因是因为被 var 声明的 i 被声明提升到了循环之外，进而“污染”了闭包中的 i

解决方案就是给它套一层函数级作用域：

```js
var funcs = []
for(var i = 0;i<3;i++) {
  funcs[i] = (function(i){
      return function(){
          console.log(i)
      }
  })(i)
}
funcs[0]();	// 0
console.log(i);	// 3
```

当然这个问题在 ES6 之后有一个更好的解决方案：

```js
var funcs = []
for(let i = 0;i<3;i++) {
  funcs[i] = function(){
    console.log(i)
  }
}
funcs[0]();	// 0
console.log(i);	// ReferenceError: i is not defined
```

让我们通过 chrome devTool 来看一下这个循环背后 JS 引擎为我们做了什么事吧：

1. 进入循环后，引擎会创建一个 block 来保存变量 i 并初始化

![3-1](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-1.jpg)

2. 再来它会判断 i 的值是否符合条件

![3-2](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-2.jpg)

3. 如果符合条件则进入并执行循环语句

![3-3](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-3.jpg)

4. 循环语句执行完之后执行更新表达式更新 block 的变量 i

![3-4](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-4.jpg)

5. 然后继续步骤 2，直到条件不符合终止循环，删除 block

![3-5](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-5.jpg)

![3-6](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-6.jpg)

综上所述，可以得到第一个结论：**引擎进入 for 循环的时候会为 let 声明的变量创建一个块作用域 block**

<br />

但是事情真的有这么简单吗？我们来考虑如下例子：

```js
var funcs = []
debugger;
for(let i = 0;i<2; i++) {
  // 在循环内部声明一个 i
  let i = 0;
  funcs[i] = function(){
    console.log(i)
  }
}
funcs[1]();	// TypeError: funcs[1] is not a function
console.log(i);
```

可以看到，循环内部声明的变量 i 不仅没有因为重复声明而报错，也没有更改循环的 i 变量导致死循环，为什么呢？

我们还是通过 devTool 看一下：

![3-7](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-7.jpg)

创建了两个 block😮

再来看一下一次循环结束时会变成怎样：

![3-8](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-8.jpg)

可以看到，包含循环内部的变量 i 的 block 被删除了

至此，我们可以得到第二个结论：**引擎进入 for 循环的时候会为循环计数器创建一个 block，并且会为每一个循环体分别创建不同的 block，这些循环体的 block 生命周期由单次循环开始，到单次循环结束**

<br />

到了这里，我们还剩下两个个问题：这两个 block 之间的关系是什么呢？进入 for 循环创建的第一个 block 的生命周期又是什么呢？

为了搞清楚这个问题，我们来看下一个例子：

```js
var funcs = []
debugger;
// 来一个 const 的循环计数器
for(const i = 0;i<2; i++) {  // TypeError: Assignment to constant variable
  let i = 0;
  funcs[i] = function(){
    console.log(i)
  }
}
funcs[0]();
console.log(i);
```

可以预见的，这段代码在第 4 行报了一个错，我们打的断点也明确的指示了报错的语句：

![3-9](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-9.jpg)

到现在，我们又收集了两个线索：

1. 更新表达式确实更改了 const i 的引用（所以才导致了报错）
2. 由上个例子第 10 行报错来看，循环体内部的变量 i 屏蔽了循环计数器的 i

综上，我们得到第三个结论：**进入 for 循环创建的第一个 block 生命周期为整个循环，且循环体的 block 存在整个循环的 block 内部**

如果被文字绕晕的同学可以看一下伪代码：

```js
// for(let i = 0;i < 2;i++) { let i = 1; }
{  // for 循环创建的 block
  let i=0;
  { // for 循环单次循环体创建的 block
    let i = 1;
  }
  i++;
  if(i < 2) goto line4;
}
```

### for-in 和 for-of

让我们来看看 for-in 循环的时候发生了什么吧～有如下例子：

```js
let arr = [1,2,3,4];
debugger;
for(const num in arr) {
    console.log(num);
}
// 0
// 1
// 2
// 3
```

按照惯例，我们看看 devTools 给我们提供了什么有价值信息

1. 首先是进入循环，给循环变量 num 创建了一个 block，并且先给了一个 undefined

![3-10](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-10.jpg)

2. 接下来就是用 arr 的键给 num 赋值并且执行循环体

![3-11](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-11.jpg)

3. 一次循环结束，看看 num 变成了什么？

![3-12](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/3-12.jpg)

4. 并且顺利进入了下一次循环直到结束并顺利打印了

结论：**for-in 的循环不会修改循环变量的引用，并且会在每次循环分别创建 block**

上述例子伪代码比较类似：

```js
{
  const num = 0;
  console.log(num);
}
{
  const num = 1;
  console.log(num);
}
{
  const num = 2;
  console.log(num);
}
{
  const num = 3;
  console.log(num);
}
```

for-of 也是一样的，感兴趣的同学可以自己试一试

### 声明提升的优先级

在 [暂时性死区 TDZ](#暂时性死区 TDZ) 我们讲了 var 与 let、const 为什么会有声明提升。但是在 JS 中，除了这三个之外，function 跟 class 也能享受声明提升的待遇。下面我们用个例子来聊一下 var、function、class 声明提升的优先级。（实际上是比较 var 与 function，class 只是 function 的语法糖）

```js
function foo() {
	return bar;
	var bar = 1;
	function bar() {}
}
console.log(typeof foo());	// function
```

可以看到，在同一个作用域下，function 声明提前的优先级是大于 var 的。

（可以这么理解：当你声明了一个任意的变量，那么理论上来说它可以是任何东西，但是如果声明了一个 function，那么它会有 Function 的一些原型，相当于类型范围被缩窄了，那肯定是 function 会优先一点）

## 总结

- 使用 `let`、`const` 声明变量会产生 TDZ，在 TDZ 内使用声明的变量会导致报错
- 在 `for` 循环中使用 `let`、`const` 变量会产生嵌套的两个 block，其中内部的 block 会随着单次循环的结束而被销毁，随着下一次循环的开始被创建；外部 block 的生命周期则是整个循环
- `for-in` 和 `for-of` 循环会在每次循环都创建一个单独的 block
- `var`、`function`、`class`、`let`、`const` 声明都会发生声明提升，只有 `let`、`const` 会产生 TDZ
- 使用 `function` 的变量声明提升优先级会高于 `var`
- 一般而言，声明一般变量建议优先使用 `const`，如果有需要修改变量的引用则使用 `let`，最后除非你知道其中的区别并且需要再使用 `var`

## Reference

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz

https://developer.mozilla.org/en-US/docs/Glossary/Hoisting#variable_hoisting

https://blog.csdn.net/weixin_44691608/article/details/106766444

https://github.com/mqyqingfeng/Blog/issues/82

https://segmentfault.com/a/1190000015603779

https://segmentfault.com/q/1010000007541743

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Loops_and_iteration

https://segmentfault.com/q/1010000000600770