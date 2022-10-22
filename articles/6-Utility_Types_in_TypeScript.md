# Utility Types in TypeScript

> 全文共 2495 字，建议阅读时间 25 min

在 Typescript 中，有很多工具类型帮助我们加快开发流程。

为了避免知其然而不知其所以然，我打算写这篇文章针对 [官网](https://www.typescriptlang.org/docs/handbook/utility-types.html) 列出的工具类一一实现并解释其应用场景。

所有的源代码都可以在 [TS 代码仓库](https://github.com/microsoft/TypeScript)中的 `lib/lib.es5.d.ts` 文件找到

## Table of Content

* [Partial&lt;Type&gt;](#partialtype)
* [Required&lt;Type&gt;](#requiredtype)
* [Readonly&lt;Type&gt;](#readonlytype)
* [Record&lt;Keys, Type&gt;](#recordkeys-type)
* [Pick&lt;Type, Keys&gt;](#picktype-keys)
* [Exclude&lt;UnionType, ExcludedMembers&gt;](#excludeuniontype-excludedmembers)
* [Omit&lt;Type, Keys&gt;](#omittype-keys)
* [Extract&lt;Type, Union&gt;](#extracttype-union)
* [NonNullable&lt;Type&gt;](#nonnullabletype)
* [Parameters&lt;Type&gt;](#parameterstype)
* [ConstructorParameters&lt;Type&gt;](#constructorparameterstype)
* [ReturnType&lt;Type&gt;](#returntypetype)
* [InstanceType&lt;Type&gt;](#instancetypetype)
* [ThisParameterType&lt;Type&gt;](#thisparametertypetype)
* [OmitThisParameter&lt;Type&gt;](#omitthisparametertype)
* [ThisType&lt;Type&gt;](#thistypetype)
* [Intrinsic String Manipulation Types](#intrinsic-string-manipulation-types)

<!-- Created by https://github.com/ekalinin/github-markdown-toc -->



## `Partial<Type>`

### 用途

 将传入 `Type` 的属性**全部设置为可选**，当我们需要一个**类型的子集**的时候这是不二选择

### 实现

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P]
};
```

说明：

- `in`：用来遍历所有可枚举值
- `keyof`：返回一个对象的所有键值
- `P in keyof T`：找出并遍历类型 T 的所有键值
- `?`：说明这个键值是可选的（optional）
- `T[P]`：类型 T 中 P 属性的类型



## `Required<Type>`

### 用途

与 [Partial](#`Partial<Type>`) 相反，将传入的 `Type` 属性**全部设置为必选项**

### 实现

```typescript
type Required<T> = {
	[P in keyof T]-?: T[P]
}
```

 说明：

- `-?`：移除了 `?` 的可选声明，相对地，`+?` 等于 `?`



## `Readonly<Type>`

### 用途

将传入的属性全部变为只读选项，比如 `Object.freeze()` 的类型可以写成：

```typescript
function freeze<T>(obj: T): Readonly<T>;
```

### 实现

```typescript
type Readonly<T> = {
	readonly [P in keyof T]: T[P];
}
```

说明：

- 针对每一个传入的对象属性增加 `readonly` 
- `readonly` 只能用在 `interface` 与 `class` 的属性，以及 `array`，`tuple` 类型
- 被 `readonly` 标记的属性或类型只能在声明或者类的构造函数中赋值



## `Record<Keys, Type>`

### 用途

将 `Keys` 中的所有属性转化为 `Type` 类型，增加开发效率，比如：

```typescript
interface StudentInfo {
  id: number;
  gender: 'M' | 'F';
  class: string;
}
type StudentName = 'Emma' | 'Olivia' | 'Ava';

type StudentWithInfo = Record<StudentName, StudentInfo>;
// => {
//     Emma: StudentInfo;
//     Olivia: StudentInfo;
//     Ava: StudentInfo;
// }
```



### 实现

```typescript
type Record<K extends keyof any, T> = {
  [P in K]: T;
}
```

 说明：

- 首先 `keyof any` 返回的是能够被当成 key 的类型，也就是 `number`， `string`，`symbol`
- `extends` 关键字限制了 `K` 的类型只能是能被当成 key 的类型
- 接下来就是把 `K` 的所有值都声明为 `T` 类型



## `Pick<Type, Keys>`

### 用途

从 `Type` 中挑选键为 `Key` 的属性，比如：

```typescript
interface StudentInfo {
  id: number;
  gender: 'M' | 'F';
  class: string;
}
type StudentClass = Pick<StudentInfo, 'id' | 'class'>;
// => {
//     id: number;
//     class: string;
// }
```



### 实现

```typescript
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}
```

 说明：

- 首先 `K extends keyof T` 限制了 `K` 只能是 `T` 的 key
- 然后只声明 key 为 K 的类型



## `Exclude<UnionType, ExcludedMembers>`

### 用途

从 `UnionType` 中找出 `ExcludedMembers` 里没有的元素（也就是从 `UnionType` 中排除 `ExcludedMembers` ）

比如：

```typescript
type T = Exclude<1 | 2, 1 | 3>
// => 2
```

### 实现

```typescript
type Exclude<U, E> = U extends E ? never : U;
```

 说明：

- 首先是 `extends` 关键字，它的作用是**检测类型 U 是否可以赋值给类型 E**，如果可以赋值则表达式为真

- 其次，`extends` 关键字在满足两个条件下，会使用分配条件类型（Distributive Conditional Types）

  - `extends` 前面是一个泛型
  - `extends` 前面的泛型传入了一个联合类型
  - 具体看 [Doc](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types)，下面是一个例子

  ```typescript
  // 1. 正常用法
  type A = 'x' extends 'x' ? string : number; // string
  type B = 'x' | 'y' extends 'x' ? string : number; // number
  
  // 2. 泛型 + 联合类型
  type P<T> = T extends 'x' ? string : number;
  type C = P<'x' | 'y'> // string | number
  
  // 实际上第二种方法被解释成了这样...
  // ('x' extends 'x' ? string : number) | ('y' extends 'x' ? string : number)
  ```

- 最后，找出了 U 中不能被赋值给 E 的元素



## `Omit<Type, Keys>`

### 用途

与 [Pick](#`Pick<Type, Keys>`) 相反，从 `Type` 中剔除掉键为 `Key` 的属性，比如：

```typescript
interface StudentInfo {
  id: number;
  gender: 'M' | 'F';
  class: string;
}
type StudentClass = Omit<StudentInfo, 'gender'>;
// => {
//     id: number;
//     class: string;
// }
```



### 实现

```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>
```

 说明：

- 选取了不在 K 中的 T 的键



## `Extract<Type, Union>`

### 用途

与 [Exclude](#`Exclude<UnionType, ExcludedMembers>`) 相反，从 `Type` 中提取出 `Union` 也有的类型

### 实现

```typescript
type Extract<T, U> = T extends U ? T : never;
```

 说明：

- 找出 T 中可以赋值给 U 的类型返回它们的联集



## `NonNullable<Type>`

### 用途

  从 `Type` 中剔除 `undefined`  和 `null`

### 实现

```typescript
type NonNullable<T> = T extends (undefined | null) ? never : T;
// 或者
type NonNullable<T> = Extract<T, undefined | null>
```



## `Parameters<Type>`

### 用途

将 `Type` 函数的参数构造成一个 tuple 类型返回，如果 `Type` 不能被赋值为一个函数，则返回 `never`

举个例子：

```typescript
function log(msg: string){}
type T = Parameters<typeof log>
// => [msg: string]
```

### 实现

```typescript
type Parameters1<T extends ((...args: any) => any)> = T extends ((...args: infer P) => any) ? P : never;
```

 说明：

- `infer` 关键字表示类型推断，它可以将一个当前未知的类型推断为某个暂时的类型变量（可以理解为将一个未知的变量类型剥离出来）
- 具体可以看看 [Doc](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types)



## `ConstructorParameters<Type>`

### 用途

将 `Type` 构造函数的参数构造成一个 tuple 类型返回，如果 `Type` 不能被赋值为一个构造函数，则返回 `never`

举个例子：

```typescript
class Log {
    constructor(msg:number){}
}
type T = ConstructorParameters<typeof Log>;
// => [msg: number]
```

### 实现

```typescript
type ConstructorParameters<T extends abstract new(...args: any) => any> = 
  T extends abstract new(...args: infer P) => any ? P : never;
```

 说明：

- 首先，构造函数的类型是 `new(...args: any) => any`
- 紧接着，将构造函数声明为 `abstract` 是为了兼容性：普通构造函数可以被赋值为抽象构造函数，反之不行



## `ReturnType<Type>`

### 用途

返回 `Type` 函数的返回类型，如果 `Type` 不能被赋值为一个函数，则返回 `any`

### 实现

```typescript
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

说明：

- 之所以默认返回一个 `any` 是因为函数的返回类型不像参数是一个数组的形式，而可以是任意类型的

## `InstanceType<Type>`

### 用途

返回 `Type` 构造函数的实例类型，如果 `Type` 不能被赋值为一个构造函数，则返回 `any`

### 实现

```typescript
type InstanceType<T extends abstract new (...arge: any) => any> = 
  T extends abstract new (...args: any) => infer R ? R : any;
```



## `ThisParameterType<Type>`

### 用途

返回 `Type` 函数的 `this` 参数类型，如果 `Type` 函数没有 `this` 参数则返回 `unknown`

### 实现

```typescript
type ThisParameterType<T> = T extends (this: infer U, args: any[])=>any ? U : unknown;
```



## `OmitThisParameter<Type>`

### 用途

移除 `Type` 函数的 `this` 参数并返回一个新的函数（如果有函数类型重载将会以最后一个为该新函数的类型）

如果 `Type` 不是一个函数类型或者 `Type` 函数没有 `this` 参数，则将返回 `Type` 本身

注意⚠️：如果 `Type` 函数的 `this` 参数类型为 `any` 则不会被删除（虽然也不会有人这么无聊就是了……吧）

### 实现

```typescript
type OmitThisParameter<T> = unknown extends ThisParameterType<T>
	? T
	: T extends (...args: infer A) => infer R
	? (...args: A) => R
	: T;
```

 说明：

- 首先 `unknown` 只能被赋值给 `unknown` 或者 `any`（这也解释了上面的注意⚠️），所以第一个判断为真有三种可能，且都返回原类型：
  1. T 是一个不含 `this` 参数的函数类型
  2. T 是一个包含 `this` 参数的函数类型，但是 this 参数被声明为 `any`
  3. T 干脆就不是一个函数类型
- 进入第二句条件判断表示 T 是一个有 `this` 为参数的函数（且 `this` 类型不为 `any`），那么我们要做的就是把 `this` 忽略，提取出其余参数与返回值，构造新的函数类型（这一步也把函数的类型重载消除掉了）



## `ThisType<Type>`

### 用途

只能用于对象字面量中，用来标记该上下文的 `this` 对象类型

注意⚠️：必须把编译选项 `noImplicitThis` 打开

### 实现

```typescript
interface ThisType<T> {}
```

 说明：

- 首先，它只能用在对象字面量中，不然就是一个空的 interface
- 再来，它的用法是告诉 TS 编译器，我这个对象的 this 类型为 T
- 举个例子：

```typescript
type ObjectDescriptor<D, M> = {
  data?: D;
  // 如果这里没有对 this 对象类型作声明, 下面的方法调用 this.x 的时候就会报错
  methods?: M & ThisType<D & M>; // Type of 'this' in methods is D & M
};

// 这个函数是为了打平 data 跟 method，将它们放到同一个对象中
function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
  let data: object = desc.data || {};
  let methods: object = desc.methods || {};
  return { ...data, ...methods } as D & M;
}

let obj = makeObject({
  data: { x: 0, y: 0 },
  methods: {
    moveBy(dx: number, dy: number) {
      this.x += dx; // Strongly typed this
      this.y += dy; // Strongly typed this
    },
  },
});

obj.x = 10;
obj.y = 20;
obj.moveBy(5, 5);
```



## Intrinsic String Manipulation Types

接下来的这一个部分，被称为内置字符操作类型。

随着 Typescript 4.1 对模版字面量类型的引入，随之而来的有四个新的工具类型，这四个工具类型都是由编译器实现的，所以在类型层面看不到底层实现。

<br />

这四种工具类型以及各自的用途如下：

```typescript
// 将类型 S 转换成全大写的类型 S
type Uppercase<S extends string> = intrinsic; 

// 将类型 S 转换成全小写的类型 S
type Lowercase<S extends string> = intrinsic;

// 将类型 S 转换成首字母大写的类型 S
type Capitalize<S extends string> = intrinsic;

// 将类型 S 转换成首字母小写的类型 S
type Uncapitalize<S extends string> = intrinsic;

// intrinsic 是一个关键字，会告诉 tsc 编译器这些实现是 TS 内置的
```

这四个工具在 tsc 中的实现如下：

```typescript
// lib/tsc.js
function applyStringMapping(symbol, str) {
  switch (intrinsicTypeKinds.get(symbol.escapedName)) {
    case 0: return str.toUpperCase();
    case 1: return str.toLowerCase();
    case 2: return str.charAt(0).toUpperCase() + str.slice(1);
    case 3: return str.charAt(0).toLowerCase() + str.slice(1);
  }
  return str;
}

// intrinsicTypeKinds 刚好对应了工具类型
var intrinsicTypeKinds = new ts.Map(ts.getEntries({
  Uppercase: 0,
  Lowercase: 1,
  Capitalize: 2,
  Uncapitalize: 3
}));
```



## Reference

https://github.dev/microsoft/TypeScript/blob/main/lib/lib.es5.d.ts

https://www.typescriptlang.org/docs/handbook/utility-types.html

https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html#intrinsic-string-manipulation-types

https://segmentfault.com/a/1190000040690347

https://segmentfault.com/a/1190000040719575

