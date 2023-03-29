# Typescript 中的协变与逆变

## 前言

看完这篇文章，你将会能回答这些问题：

1. 什么是协变（Covariance）？什么是逆变（Contravariance）？什么是双向协变（Bicovariance）？什么是不变（Invarianc）？
2. 为什么要有逆变？
3. method 跟 function property 有什么区别？
4. 函数类型之间可转化的规则是什么？

## 协变

```tsx
interface Animal {
  age: number;
}

interface Dog extends Animal {
  bark(): void;
}

// covariance test
declare let animal: Animal;
declare let dog: Dog;

animal = dog;    // success
dog = animal;    // Error: Property 'bark' is missing in type 'Animal' but required in type 'Dog'.
```

想了解协变，我们可以从一个例子说起：

我们有两个类型，Animal 与 Dog，其中 Dog 继承至 Animal，并且拥有自己的 bark 方法

我们将 dog 赋值给 animal 时，是没问题的，因为 Animal 只关注值是否有 age 这个属性

但是当我们将 animal 赋值给 dog 时，ts 却报错了，这是因为 dog 不仅关注了 age 这个属性，还要求值必须要有 bark 方法，而 animal 并不符合后者，所以 ts 报了错误

简单来说，协变就是**允许子类型转换为父类型**

## 逆变

```tsx
interface Animal {
  age: number;
}

interface Dog extends Animal {
  bark(): void;
}

// Contravariance test - function properties
interface FnPropertyTest<T> {
  test: (s: T) => void;
}

declare let animalFnPropertyTest: FnPropertyTest<Animal>
declare let dogFnPropertyTest: FnPropertyTest<Dog>

// Error: Type 'FnPropertyTest<Dog>' is not assignable to type 'FnPropertyTest<Animal>'.
// Property 'bark' is missing in type 'Animal' but required in type 'Dog'.
animalFnPropertyTest = dogFnPropertyTest;
// success
dogFnPropertyTest = animalFnPropertyTest;
```

同样，我们用一个例子来说明

这个例子使用的还是上面我们用的 Dog 与 Animal，只是区别在于我们将它们放在了函数属性的参数的位置

当我们将有 Dog 属性的方法赋值给有 Animal 属性的方法时，ts 报错了，并且错误的提示是我们无法把 Dog 赋值给 Animal

这个结论似乎与我们协变的结论背道而驰了……吗？

当我们把方法 dogFnPropertyTest 赋值给 animalFnPropertyTest 后，我们的 animalFnPropertyTest 可以被看成如下代码：

```tsx
animalFnPropertyTest = (animal: Animal): void => {
	// Error
	return dogFnPropertyTest(animal);
}
```

在上面的协变中，我们知道：Animal 类型是不能赋值给 Dog 类型的，所以 ts 就报错了

属性方法参数的角度看，Animal（父类型）被成功转换为 Dog（子类型），所以这种现象被称为逆变

简单来说，**逆变就是允许父类型转换为子类型**

需要注意 function property 的参数逆变只有打开了 --strictFunctionTypes 才会生效

## 双向协变

依然是一个熟悉的例子：

```tsx
interface Animal {
  age: number;
}

interface Dog extends Animal {
  bark(): void;
}

// Bicovariance test - methods
interface MethodTest<T> {
  test(s:T): void;
}

declare let animalMethodTest: MethodTest<Animal>;
declare let dogMethodTest: MethodTest<Dog>;

animalMethodTest = dogMethodTest    // success
dogMethodTest = animalMethodTest    // success
```

可以看到，这个例子于上面逆变的例子几乎一样……唯一的区别在于这个例子使用了 method 的方式来定义 test 方法，上个例子用了 function property

而**双向协变，就是允许父子类型相互转换**

为什么会有这个区别呢？

实际上，一开始的 ts 对于函数参数采取的策略都是双向协变，但是考虑到如下的 case：

```tsx
interface Animal {
  age: number;
}

interface Dog extends Animal {
  bark(): void;
}

let animalAge = (animal: Animal) => {
  // do something
}

let dogBark = (dog: Dog) => {
  dog.bark();
}

animalAge = dogBark  // success when strictFunctionTypes is off

const cat: Animal = {
  age: 2,
}

// Error: bark() is undefined
animalAge(cat)
```

我们将 dogBark 的方法赋值给了 animalAge，可以看到在双向协变（关闭 --strictFunctionTypes）的场景，这个赋值操作在 ts 编译时是不会报错的

但是我们在 dogBark 中调用了 Animal 类型没有的方法，所以在运行时会导致 js 的报错

<br />

那么为什么 ts 只让 function property 属性关闭了协变只保留逆变呢？

在 ts 的 [issue]([]()) 中，有一句话：

```
Methods are excluded specifically to ensure generic classes and interfaces (such as Array<T>) continue to mostly relate covariantly.
```

什么意思呢？

我们先来看看 Array<T> 在 ts 中的类型是怎么写的吧：

```typescript
interface Array<T> {
	length: number;
	toString(): string;
	toLocaleString(): string;
	pop(): T | undefined;
	push(...items: T[]): number;
	concat(...items: ConcatArray<T>[]): T[];
	// ......
}
```

可以看到所有的 Array 方法都是用 method 的方式来写的，为的就是维持双向协变的特性

举个例子：

```tsx
const dogArr: Array<Dog> = [] as Dog[];
const animalArr: Array<Animal> = dogArr;  // success
```

我们知道 Dog 类型是可以直接赋值给 Animal 类型的，dogArr 自然也能赋值给 animalArr

现在考虑到 push 这个方法，dogArr 的参数类型是 Dog；animalArr 的参数类型是 Animal

如果按照正常逆变来判断的话，这个操作显然是不允许的，所以只能用双向协变来“放宽”这个限制

## 不变

不变的话就相当简单了，直接看例子

```tsx
interface Dog {
	bark();
}

interface Cat {
	mew();
}

const dog: Dog = {
	bark: () => {
		// bark
	}
}
const cat: Cat = dog;   // Error
```

我们肯定不能把狗当成一直猫然后叫它学猫叫吧 = =

## 函数的场景

通过上面的文章，我们了解到函数的参数判断规则是逆变，除了参数之外，函数还有几个有趣的场景值得我们深入探索一下

### 函数的参数数量不同

参数数量不同的函数之间的赋值规则是怎样的呢？

我们用个例子来了解一下

```tsx
let fn1 = (p1: number, p2: number): void => {}
let fn2 = (p1: number): void => {}

fn1 = fn2;  // success
fn2 = fn1;  // Error: Type '(p1: number, p2: number) => void' is not assignable to type '(p1: number) => void'
```

可以看到**将参数少的函数赋值给参数多的函数是可以的**；反之则不行

究其原因，当我们将参数多的函数 fn1 赋值给 fn2 时，可以看成如下：

```tsx
let fn2 = (p1: number) => {
	return fn1(p1, p2)  // p2 is undefined
}
```

fn1 中可能会使用 fn2 没有的参数 p2，所以这种赋值是有风险的

而将 fn2 赋值给 fn1 时，fn1 的参数 p2 是不会被 fn2 消费的，相当于多传了一个参数，这个是安全的

### 函数的返回值

接下来我们探讨一下函数的返回值对函数之间赋值的影响：

```tsx
interface Animal {
  age: number;
}

interface Dog extends Animal {
  bark(): void;
}

let fn1 = (): Animal => ({age: 2})
let fn2 = (): Dog => ({age: 2, bark: () => {}})

fn1 = fn2;  // success
fn2 = fn1;  // Error: Type '() => Animal' is not assignable to type '() => Dog'.
```

可以看到，fn2 被复制给 fn1 是被允许的，也就是说，**函数的返回值遵循协变的规则**

我们一样来分析一下为什么 fn1 赋值给 fn2 会导致报错：

```tsx
let fn2 = (): Dog => {
	return fn1()
}
```

可以看到，fn1 的返回值是 Animal 类型，但是 fn2 要求返回值是 Dog 类型，我们知道 Animal（父类）是不能被复制给 Dog（子类）的，所以这个类型转换是不安全的

### 函数返回 void 与其他值的转换

还有一种情况值的拿出来讨论，那就是返回 void 的函数与返回非空的函数之间的类型转换规则

```tsx
let fn1 = (): number => { return 1; }
let fn2 = (): void => {}

fn1 = fn2;  // Type '() => void' is not assignable to type '() => number'.
fn2 = fn1;
```

可以看到，**返回 void 的函数是不允许赋值给任何返回非空的函数的**

我们一样来分析一下：

```tsx
let fn1 = (): number => {
	return fn2()
}
```

fn2 返回了一个空值（void），但是 fn1 要求的是一个 number，所以这种类型转换是不安全的

那如果是返回非空值的函数赋值给返回 void 的函数呢？

```tsx
let fn2 = (): void => {
	return fn1()
}
```

fn1 返回了一个 number 类型的值，而 fn2 要求的返回值是 void，表示它不关心函数的返回值，所以函数返回任意的值都被视为是安全的

## 总结

- 协变是允许子类型转换为父类型
- 逆变就是允许父类型转换为子类型
- 双向协变允许父子类型相互转换
- 函数属性（function property）的参数是逆变的（前提是打开 --strictFunctionTypes）
- 方法（method）的参数是双向协变的
- 参数较少的函数可以赋值给参数较多的函数，反之不行
- 函数的返回值是协变的
- 返回非空的函数允许赋值给返回值为 void 的函数，反之则不行