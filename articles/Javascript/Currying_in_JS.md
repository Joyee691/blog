# Javascript 中的柯里化（Currying）

> 全文共 1969 字，建议阅读时间 15 min

## 什么是柯里化

柯里化（Currying）是一种函数式编程[^1]的技巧，Wikipedia[^2]是这么描述它的：

> **Currying** is the technique of converting a function that takes multiple arguments into a sequence of functions that each takes a single argument.

简单来说，柯里化是一种**将一个接收 N 个参数的函数转变为 N 个只接收一个参数的函数**的技术，也就是：$f(x,y,z) \Rightarrow h(x)(y)(z)$

### 举个例子

假设今天甲方要让你计算一批**固定长宽**的长方体体积：

```js
// 第一种方法：普通函数
function calculateVolume(length, width, height) {
  return length * width * height;
}

calculateVolume(1, 2, 3);
calculateVolume(1, 2, 4);
calculateVolume(1, 2, 5);
calculateVolume(1, 2, 6);
```

这么写显然有些麻烦了，不只是要复制多次，而且如果之后甲方要修改长或者宽的成本会很大，我们可以用柯里化改写如下：

```js
// 第二种方法：柯里化
function calculateVolume(length) {
  return function(width) {
    return function(height) {
      return length * width * height;
    }
  }
}

calculateVolume(1)(2)(3);
calculateVolume(1)(2)(4);
calculateVolume(1)(2)(5);
calculateVolume(1)(2)(6);
```

这时候有些同学可能就会问了，也没什么不同呀🤔。别急，我们可以根据上面再改写一下：

```js
// Partial Application
function calculateVolume(length) {
  return function(width) {
    return function(height) {
      return length * width * height;
    }
  }
}

const calculateVolumeWithFixLenAndHeight = calculateVolume(1)(2);
calculateVolumeWithFixLenAndHeight(3);
calculateVolumeWithFixLenAndHeight(4);
calculateVolumeWithFixLenAndHeight(5);
calculateVolumeWithFixLenAndHeight(6);
```

通过这种方式，我们实现了**参数复用**。

### 柯里化 vs Partial Application

细心的同学可能发现了我在第三段代码写上了 “**Partial Application**” 作为与柯里化的区分，由于国内的帖子很少专门讨论这两个概念的异同，所以我决定在这里花一点篇幅介绍一下。

Partial Application 在 Wikipedia[^3] 上是这么介绍的：

> **Partial application** (or **partial function application**) refers to the process of fixing a number of arguments to a function, producing another function of smaller [arity](https://en.wikipedia.org/wiki/Arity).

简单来说，它是一个将部分参数事先 “绑定” 到函数上，然后返回需要更少参数的函数的方法。Function.prototype.bind()[^4] 方法在原生上支持了 Partial Application。

两者的区别在于：**柯里化负责将有多个参数的函数转变为多个接收一个参数的函数；Partial application 则是负责 “绑定” 一些参数到函数上并返回绑定后的函数。**

## 柯里化的基础

相信用过柯里化的同学都会听过一句话：柯里化是闭包的概念的一种应用。

所以下面我将会用一点篇幅来介绍一下闭包的概念：

### 什么是闭包

根据 Wikipedia[^5]，闭包的定义是：

> A **closure**, also **lexical closure** or **function closure**, is a technique for implementing [lexically scoped](https://en.wikipedia.org/wiki/Lexically_scoped) [name binding](https://en.wikipedia.org/wiki/Name_binding) in a language with [first-class functions](https://en.wikipedia.org/wiki/First-class_function).

简单来说，**闭包是一种函数词法范围绑定的技术**。

#### 举个例子

 ```js
 function foo() { 
   var a = 2;
 
 	function bar() { 
     console.log(a);
 	}
 	return bar; 
 }
 var baz = foo();
 baz(); // 2——闭包的作用
 ```

可以看出来，因为 foo 函数返回了 bar，导致本来应该被回收的作用域没有被回收，而且仍然可以被 baz 函数使用，这就是闭包的作用。

回到柯里化，让我们看看刚刚的例子：

```js
function calculateVolume(length) {
  // 为了方便描述给内部函数命名了一下
  return function getWidth(width) {
    // 为了方便描述给内部函数命名了一下
    return function getHeight(height) {
      return length * width * height;
    }
  }
}
```

我们来解析一下这段函数到底做了什么事：

1. `calculateVolume` 接收了一个参数 `length` 并且返回了一个函数 `getWidth`，这个时候就产生了一个闭包（`getWidth` 取得了 `length` 的访问权）
2. 同理，返回 `getHeight` 也产生了一个闭包（`getHeight` 取得了 `length` 与 `width` 的访问权）
3. 最后，在 `getHeight` 内部，通过闭包取得 `length` 与 `width` 的值，再加上自己的参数 `height` ，把这三者相乘之后返回

## 柯里化的实现

### 参数定长函数的柯里化

```js
function currying(fn) {
	// 获取需要柯里化的函数参数个数
	const argLen = fn.length;
	// 保存迄今为止接收到的参数
	const presetArgs = [].slice.call(arguments, 1);
	return function (...restArgs) {
		// 将原来有的参数与新接收的参数合并
		const allArgs = [...presetArgs, ...restArgs];
		// 如果参数列表长度满足 fn 的需要的话就执行 fn，否则继续
		if (allArgs.length >= argLen) {
			return fn.apply(null, allArgs);
		} else {
			return currying.call(null, fn, ...allArgs);
		}
	};
}

function add(a, b) {
	return a + b;
}
const curriedAdd = currying(add);
let res1 = curriedAdd(1)(2);  // 3
let res2 = curriedAdd(1, 2);  // 3
let res3 = curriedAdd(1, 2, 3); // 3
let res4 = curriedAdd(1);
let res5 = res4(2); // 3
```

### 参数不定长函数的柯里化

其实上面的 currying 函数已经可以满足大部分的应用场景了，但是考虑到如下函数：

```js
function dynamicAdd() {
  return [...arguments].reduce((prev, curr) => {
		return prev + curr;
	}, 0);
}
dynamicAdd.length; // 0
```

上述的函数可以提供运行时的函数参数计算，为了支持这种函数，我们的 currying 需要那么亿点点的修改，主要有三种思路。

#### 加一个参数

```js
// 加一个参数用来让用户自定义参数个数
function currying(fn, ARITY = fn.length) {
	// 这里需要从第三个参数开始收集
	const presetArgs = [].slice.call(arguments, 2);
	return function (...restArgs) {
		const allArgs = [...presetArgs, ...restArgs];
		// 只有参数长度达到要求才执行
		if (allArgs.length >= ARITY) {
      // 把多出来的参数丢掉
			return fn.apply(null, allArgs.slice(0, ARITY));
		} else {
			return currying.call(null, fn, ARITY, ...allArgs);
		}
	};
}

const curriedAdd = currying(dynamicAdd, 2);
let res1 = curriedAdd(1)(2); // 3
let res2 = curriedAdd(1, 2); // 3
let res3 = curriedAdd(1, 2, 3); // 3——多传的参数会被丢掉
let res4 = curriedAdd(1); 
let res5 = res4(2); // 3
```

上面修改后的版本看似完美的完成了任务，但是它对用户实在是不怎么友好，有两个严重问题：

1. 它要求用户多传一个参数（哪怕是不需要指定长度也需要传 `undefined`）
2. 如果用户在用参数不定长函数（比如上述的 `dynamicAdd`）传了一个 `undefined`，可能会导致程序运行不符合预期（以 `dynamicAdd` 为例，总是返回 0）

#### 约定一个范式

```js
function currying(fn) {
	const presetArgs = [].slice.call(arguments, 1);
	return function (...restArgs) {
		const allArgs = [...presetArgs, ...restArgs];
		// 当传入的参数列表为空的之后再执行，否则继续
		if (restArgs.length === 0) {
			return fn.apply(null, allArgs);
		} else {
			return currying.call(null, fn, ...allArgs);
		}
	};
}


const curriedAdd = currying(dynamicAdd, 10);
let res1 = curriedAdd(1); // 11
let res2 = curriedAdd(1, 2); // 13
let res3 = curriedAdd(1, 2, 3); // 16
let res4 = curriedAdd(1); // 11
let res5 = res4(2); // 13

// 跟用户约定以参数为空来表示运行并返回结果
console.log(res1(), res2(), res3(), res4(), res5());
```

上面的版本最大的区别在于第 6 行，判断传入的参数是否为空，如果是则执行运算，否则继续等待更多参数传入。

与上述添加参数的版本相比，这个方法无疑会好的多，但是还是要有一定的沟通成本。

#### 魔改原型

```js
function currying(fn) {
	const presetArgs = [].slice.call(arguments, 1);
	function curried(...restArgs) {
		const allArgs = [...presetArgs, ...restArgs];
		return curry.call(null, fn, ...allArgs);
	}
	// 重写 toString
	curried.toString = function () {
		return fn.apply(null, presetArgs);
	};
	return curried;
}

const curriedAdd = currying(dynamicAdd);
let res1 = curriedAdd(1)(2)(3)(4); // 10
let res2 = curriedAdd(1, 2)(3, 4); // 10
let res3 = res2(5, 6); // 21
console.log(res1 + res3); // 31
```

这个思路源自于 Tsui 大佬的博客[^6]，他魔改了函数原型的 toString 方法，这将使得返回的函数在进行运算的时候会根据抽象的 ToPrimitive 操作隐式调用 toString 方法，从而能被当成数字处理。Respect

## 总结

柯里化的有点可以总结如下：

- 参数复用
- 延迟执行
- 利于管道式编程（Pipeline programming）

##  Reference

[^1]:https://en.wikipedia.org/wiki/Functional_programming
[^2]:https://en.wikipedia.org/wiki/Currying
[^3]:https://en.wikipedia.org/wiki/Partial_application
[^4]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
[^5]:https://en.wikipedia.org/wiki/Closure_(computer_programming)

[^6]:https://juejin.cn/post/6864378349512065038

https://zh.javascript.info/currying-partials

https://ithelp.ithome.com.tw/articles/10195145

https://segmentfault.com/a/1190000021677898

https://juejin.cn/post/6889250555035090951