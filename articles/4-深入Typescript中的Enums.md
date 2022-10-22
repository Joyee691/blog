# 深入 Typescript 中的 Enums

> 全文共 3676 字，建议阅读时间 35 mins

## 前言

枚举（enumaration, aka enum）一直被认为是管理一系列相关联常量，**提升代码可读性**的一个工具。Typescript 更新的 `enum` 类型突破了类型层面的限制，在**运行时**为 Javascript 补充上了这块空白。在 TS 的手册中是这么描述 `enum` 的：

> Enums are one of the few features TypeScript has which is not a type-level extension of JavaScript.

但我在看 [Effective Typescript](https://www.amazon.com/Effective-TypeScript-Specific-Ways-Improve/dp/1492053740) 与 [Programming Typescript](https://www.amazon.com/Programming-TypeScript-Making-JavaScript-Applications/dp/1492037656) 这两本书的时候，看到两个作者不约而同的对这个类型的部分用法提出了警告，Dan Vanderkam 甚至在 Item 53 建议读者不要使用这个类型并提出了用 `|` 来取代字符枚举的方法。

本文将从基本用法着手，逐步回答两个问题：

- 该用 enum 吗？
- 该怎么用 enum？

<br />

## 基本用法

### 数字枚举

假设今天我们跟后端同学约定好了一组查询请求的内部响应状态码：

```typescript
{
  0: Success,
  1: MissingParameter，
  2: InvalidToken
}
```

然后一个同学写了如下的响应处理函数：

```typescript
const responseHandler = (status: number): void => {
  switch (status) {
    case 0:
      // do something...
      break;
    case 1: 
      // do something else...
      break;
    case 2:
      // do something else...
      break;
    default:
      // handle undefined status
  }
}
```

运行起来没啥毛病，但是下一个维护的同学估计人傻了……

如果我们用了 enum 之后，代码会变成这样子：

```typescript
enum ResponseStatus {
  Success = 0,
  MissingParameter = 1,
  InvalidToken = 2
}

const responseHandler = (status: number): void => {
  // 之后会解释为什么能用解构赋值
  const {Success, MissingParameter, InvalidToken} = ResponseStatus;
  
  switch (status) {
    case Success:
      // do something...
      break;
    case MissingParameter: 
      // do something else...
      break;
    case InvalidToken:
      // do something else...
      break;
    default:
      // handle undefined status
  }
}
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewDKAXAhls8UA3gFBRQZgDGVChAvFAAwA0ZUAsgJbzxcgBzAAo4ATjgjAswUVEYBGNuQCSIAG44ANlwAmAFWQBrUHKgAmEgF8SJKqnhYoohCjTAAEjhA7NM0wAoHPAIALihwCAAjGQBKMLVkXTkAPmJ2AHp0qEBpOUA4FUAseUBjyMApxMAuOUABuUAJOUBfgMAKV0LAEPNAaVjAHgV2OzRHIkoaOhZOHj5BEXFJaVF+1Q1tfSNQS1M4JHtMXHx4AG52dngAdy4sKgALKEC1ghi08nIqHHQKalpeEPZrqEyoHWQoeGQxo-4AgAdCDXtdIs4cIYtm9bvduLxASMJFIZGEweQPl8fn8pADBFBgJp0CCgRioBDgFCYdc4cAoFMtLoDMYQC83pistjfv9AYTicBSeTKdSwTpgAAzHBgTRYdkcj5HLw+elgbyS-jAHQ-c7wdjWSxAA)

这样代码的可读性是不是直接一个飞跃性进步😎

其中的枚举类型 `ResponseStatus` 因为枚举值都是数字，被称为**数字枚举**（Numeric enums）

#### 数字枚举初始化

```typescript
// 显式初始化
enum ResponseStatus {
  Success = 0,
  MissingParameter = 1,
  InvalidToken = 2
}

// 隐式初始化（初始值从 0 开始递增）
enum ResponseStatus {
  Success,	// 0
  MissingParameter,	// 1
  InvalidToken	// 2
}

// 如果显式初始化一个指定数字，则下面的枚举值会接着递增，上面的还是默认从 0 递增
enum ResponseStatus {
  Success,	// 0
  MissingParameter = 100,	// 100
  InvalidToken		// 101
}

// 当然也可以用运算符求值
enum ResponseStatus {
  Success = 1<<0,	// 0b1
  MissingParameter = 1<<1,	// 0b10
  InvalidToken		// 3
}

// 用表达式1
enum ResponseStatus {
  Success,	// 0
  MissingParameter,	// 1
  InvalidToken = Math.random()	// random number
}

// 用表达式2: 因为 ts 判断不了 invalidToken 的值，所以直接报错了
enum ResponseStatus {
  Success,	// 0
  MissingParameter = Math.random(),	// random number
  InvalidToken	// Error: Enum member must have initializer.
}
```

### 字符枚举

上述的数字枚举提供了很好的代码可读性，但是如果今天需要调试的话，数字枚举提供不了比较有价值的信息

让我们继续上一个例子，只是这次需要把错误原因打印到 `console` 面板：

```typescript
enum ResponseStatus {
  Success = 0,
  MissingParameter = 1,
  InvalidToken = 2
}

// 字符枚举
enum ResponseMsg {
  Success = 'Success',
  MissingParameter = 'Missing Parameter',
  InvalidToken = 'Invalid Token',
  UnknownErr = 'Unknown Error'
}

const responseHandler = (status: number): void => {
  const {Success, MissingParameter, InvalidToken} = ResponseStatus;
  
  switch (status) {
    case Success:
      console.log(ResponseMsg.Success);
      break;
    case MissingParameter: 
      console.log(ResponseMsg.MissingParameter);
      break;
    case InvalidToken:
      console.log(ResponseMsg.InvalidToken);
      break;
    default:
      console.log(ResponseMsg.UnknownErr);
  }
}

responseHandler(0); // "Success"
responseHandler(1); // "Missing Parameter"
responseHandler(2); // "Invalid Token"
responseHandler(3); // "Unknown Error"
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewDKAXAhls8UA3gFBRQZgDGVChAvFAAwA0ZUAsgJbzxcgBzAAo4ATjgjAswUVEYBGNuQCSIAG44ANlwAmAFWQBrUHKgAmEgF8SJAPS2ogdW1AZN6As80B8ciVCRYCFGmAOeAFidkoaOlMAcnDaXiilTh4+QRFxSWlZRijuXn4QtIkpGQT2VQ1tfSMTbPKtXSgDYxBS8gBVEEMQZAB3EABRUSyoKI6u3pAoQdFkUSirGypUeCwoUT9l4AAJHBAdTRlTAAoVvAIALihwCAAjGQBKS7VkBvoAPlDyJbRVoli6FhJPKpMRFTKAuqVJqgSymOBITbYM7wADc7HY8B6XCwVAAFlATrh8PB7p9yFAqDh0BRqHF4Od2OSKctkAcAHSaZACI7w-zoIICNn-Xj3NFM8g3dY4Qxi8mU6m5FLCUEZGSXRlylnsznc3mbAVsxX5Qqq0SijUSqUyjXy4BQSG6aEgBni5loVnADlcnkbAIGh1VZrm12S4DS2XkHTAABmODAmiwLvF33gHq9ut9-OCbLG3T602DUGs1hI6wRAR2ewOoiOTFFUHsUAARML4E3S5ntrt9jIjvJ642m0bBFATcVRO2y3yu1Xe2YBw4mwHGtUQJPO5WezWAMwL5u5iZTIazJtAA)

可以看到，字符枚举不止增加了代码的可读性，也集中了所有调试信息，提升了代码的可维护性。

#### 字符枚举初始化

```typescript
// 正常的字符初始化：必须把所有初始值都安排上
enum ResponseMsg {
  Success = 'Success',
  MissingParameter = 'Missing Parameter',
  InvalidToken = 'Invalid Token',
  UnknownErr = 'Unknown Error'
}

// 字符枚举必须初始化全部的值
enum ResponseMsg {
  Success = 'Success',
  MissingParameter,		// Error: Enum member must have initializer.
  InvalidToken = 'Invalid Token',
  UnknownErr = 'Unknown Error'
}

// 初始化可以用模版字符串，但是不允许使用变量
const foo = 'Unknown Error'
enum ResponseMsg {
  Success = 'Success',
  MissingParameter = `Missing Parameter`,		// 可以使用 Template String 形式
  InvalidToken = 'Invalid Token',
  UnknownErr = `${foo}`		// Error: Computed values are not permitted in an enum with string valued members.
}
```

### 混合枚举（不推荐）

混合枚举，顾名思义，就是把数字与字符的枚举混合在一起：

```typescript
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = "YES",
}
```

一般而言，除非真的同时需要数字枚举在运行时的特性（稍后解释）以及字符枚举的信息，不然**不建议使用混合枚举**

混合枚举更像是一种实现上的副产物（后面讲到如何将 enum 转化为 js 代码的时候会讲到）

<br />

## JS 版本的枚举实现

### 数字枚举

前面有讲到，Typescript 在运行时层面为 Javascript 填补了枚举这个功能的空白。

但是，V8 解释的还是 Javascript 语言本身，所以 TS 的编译器必须要先把 TS 代码 “翻译” 为等价的 JS 才能够被 V8 进一步解释运行。

让我们回到数字枚举的例子：

```typescript
enum ResponseStatus {
  Success,
  MissingParameter = 100,
  InvalidToken
}
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewDKAXAhls8UA3gFBRQZgDGVC8ANGVALICW88rIA5gAo4AnHBGBZgAqAF4oARgAMcxuQCSIAG44ANqwAmAFWQBrUCQC+QA)

经过编译器的翻译，JS 代码是这样子的：

```js
"use strict";
var ResponseStatus;
(function (ResponseStatus) {
  // 默认从 0 开始赋值
    ResponseStatus[ResponseStatus["Success"] = 0] = "Success";
    ResponseStatus[ResponseStatus["MissingParameter"] = 100] = "MissingParameter";
  // 因为上一项被显式赋值了，所以这一项默认从上一项开始递增
    ResponseStatus[ResponseStatus["InvalidToken"] = 101] = "InvalidToken";
})(ResponseStatus || (ResponseStatus = {}));
```

可以看到，将 enum 翻译为 JS 总共有 3 个步骤：

1. 首先声明一个变量 `ResponseStatus`
2. 然后在调用 [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) 时用一个空对象初始化 `ResponseStatus` 
3. 接下来针对每一个枚举项，它先创建了 `枚举项 -> 数字` 的映射，再创建了 `数字 -> 枚举项` 的映射
4. 对于没有显式初始化的枚举项，会从 0 开始赋值；如果已经有部分被赋值了，那么之后的值默认由上一项开始递增

这个就是为什么数字枚举支持**双向映射**的原因。

<br />

具体而言，数字枚举支持以下方式调用：

```typescript
// 双向映射
ResponseStatus.Success;	// 0
ResponseStatus[0];	// 'Success'
```

### 字符枚举

依照惯例，我们还是来看看之前的例子：

```typescript
enum ResponseMsg {
  Success = 'Success',
  MissingParameter = 'Missing Parameter',
  InvalidToken = 'Invalid Token',
  UnknownErr = 'Unknown Error'
}
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewCy8DmUDeAUFFAMpgDG5C8UAvFAORmXUMA0RUGAlvPNyGwAFAIYAnERGAAXYGLqMefAblESpsse04BJEADcRAG24ATACrIA1qAUM9hk6aiWbIbcQCqIKyGQB3EABRMXl6Bm9fAJAoELFkLQIAXyA)

经过编译器的编译，会变成这样：

```js
"use strict";
var ResponseMsg;
(function (ResponseMsg) {
    ResponseMsg["Success"] = "Success";
    ResponseMsg["MissingParameter"] = "Missing Parameter";
    ResponseMsg["InvalidToken"] = "Invalid Token";
    ResponseMsg["UnknownErr"] = "Unknown Error";
})(ResponseMsg || (ResponseMsg = {}));
```

大致还是相同的，区别在于函数体内：它只创建了 `枚举项 -> 字符串` 的映射。

所以**字符枚举是不支持双向映射的**。

我们再来看看，当我们不初始化字符枚举的时候会发生什么事：

```typescript
enum ResponseMsg {
  Success = 'Success',
  MissingParameter,		// 故意把初始化值删掉了
  InvalidToken = 'Invalid Token',
  UnknownErr = 'Unknown Error'
}

"use strict";
var ResponseMsg;
(function (ResponseMsg) {
    ResponseMsg["success"] = "Success";
  	// 这里被默认给了一个 undefined
    ResponseMsg[ResponseMsg["MissingParameter"] = void 0] = "MissingParameter";
    ResponseMsg["InvalidToken"] = "Invalid Token";
    ResponseMsg["UnknownErr"] = "Unknown Error";
})(ResponseMsg || (ResponseMsg = {}));
```

[Playground](https://www.typescriptlang.org/play?ssl=6&ssc=2&pln=1&pc=1&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewCy8DmUDeAUFFAMpgDG5C8UAvFAORmXUMA0RUGAlvPNyGwAFAIYAnERGAAXYGLYBIBQHplUQKKmgeENAUUaBcJUDTmoDRlQDwKgAiVAkcaAwuU4BJEADcRAG24ATACrIA1qDqM7ji6uUJ4+IOycAKogXiDIAO4gAKJiYn4M0bEJIFApYshiDAQAvkA)

可以看到，没有被显式初始化的字符枚举对象被默认给了一个 `void 0`

所以编译器贴心的给我们报了错

### 混合枚举

我们来看一个混合枚举的例子：

```typescript
enum ResponseMsg {
  Success = 'Success',
  MissingParameter = 0,
  InvalidToken = 'Invalid Token',
  UnknownErr	// 故意不初始化
}

// "use strict";
var ResponseMsg;
(function (ResponseMsg) {
    ResponseMsg["Success"] = "Success";
    ResponseMsg[ResponseMsg["MissingParameter"] = 0] = "MissingParameter";
    ResponseMsg["InvalidToken"] = "Invalid Token";
  // 就算是混合枚举，没有显式初始化也会默认给一个 undefined
    ResponseMsg[ResponseMsg["UnknownErr"] = void 0] = "unknownErr"
})(ResponseMsg || (ResponseMsg = {}));
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewCy8DmUDeAUFFAMpgDG5C8UAvFAORmXUMA0RUGAlvPNyGwAFAIYAnERGAAXYGLpQADB2IBJEADcRAG24ATACrIA1qAUN1W3XqhHTIdpwCqIYyGQB3EAFExYgJAA9IFQgKKmgPCGgLBygLhKgNOagGjKBAC+QA)

可以观察到几点：

- 枚举值是数字的还是会编译成双向映射
- 对于枚举值是字符的还是只有单向映射
- 对于没有初始化的值会默认给一个 `undefined`

### 总结

1. 对于枚举，JS 会创建一个对象，并为其建立映射
2. 对于数字枚举，JS 会为其建立双向映射
3. 对于字符枚举，JS 只会建立 `枚举项 -> 字符` 的单向映射
4. 初始化数字枚举的时候，会自动为没有显式初始化的枚举值从 0 开始依次赋值；如果已经有部分被赋值了，那么之后的值默认由上一项开始递增
5. 对于**含有字符枚举值**的枚举，会默认给没有初始化的值 `void 0`，并由 TS 编译器负责报错

<br />

## 最佳实践

要了解使用枚举的最佳实践，首先要先知道它有哪些特性与缺陷：

### 同名 enum 的合并

就像 interface 一样，enum 支持将两个同名的枚举自动合并，考虑到如下例子：

```typescript
enum ResponseStatus {
  Success = 0,
  MissingParameter = 1,
  InvalidToken = 2
}

enum ResponseStatus {
  PermissionDenied = 3
}

// JS 版本：
"use strict";
var ResponseStatus;
(function (ResponseStatus) {
    ResponseStatus[ResponseStatus["Success"] = 0] = "Success";
    ResponseStatus[ResponseStatus["MissingParameter"] = 1] = "MissingParameter";
    ResponseStatus[ResponseStatus["InvalidToken"] = 2] = "InvalidToken";
})(ResponseStatus || (ResponseStatus = {}));
(function (ResponseStatus) {
    ResponseStatus[ResponseStatus["PermissionDenied"] = 3] = "PermissionDenied";
})(ResponseStatus || (ResponseStatus = {}));
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewDKAXAhls8UA3gFBRQZgDGVChAvFAAwA0ZUAsgJbzxcgBzAAo4ATjgjAswUVEYBGNuQCSIAG44ANlwAmAFWQBrUHKgAmEgF8SJUJFgIUaTLnyFS5ITIg8+qACKgXMA6pgDMVkA)

可以看到，最后一行巧妙的用了一个或运算，避免了 `ResponseStatus` 被重新初始化，从而可以将同名枚举融合。

<br />

虽然这个特性对开发者非常友好，但是还是有两个问题的，看如下例子：

```typescript
enum ResponseStatus {
  Success = 0,
  MissingParameter = 1,
  InvalidToken = 2
}

enum ResponseStatus {
  PermissionDenied,		// 故意不初始化
  Success = 200,		// Error: Duplicate identifier 'success'.
}

// JS 版本
"use strict";
var ResponseStatus;
(function (ResponseStatus) {
    ResponseStatus[ResponseStatus["Success"] = 0] = "Success";
    ResponseStatus[ResponseStatus["MissingParameter"] = 1] = "MissingParameter";
    ResponseStatus[ResponseStatus["InvalidToken"] = 2] = "InvalidToken";
})(ResponseStatus || (ResponseStatus = {}));
(function (ResponseStatus) {
  // 默认还是从 0 开始，会导致双向映射失败
    ResponseStatus[ResponseStatus["PermissionDenied"] = 0] = "PermissionDenied";
    ResponseStatus[ResponseStatus["Success"] = 200] = "Success";
})(ResponseStatus || (ResponseStatus = {}));


console.log(ResponseStatus);
// {
//   "0": "PermissionDenied",
//   "1": "MissingParameter",
//   "2": "InvalidToken",
//   "200": "Success",
//   "Success": 200,
//   "MissingParameter": 1,
//   "InvalidToken": 2,
//   "PermissionDenied": 0
// } 
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewDKAXAhls8UA3gFBRQZgDGVChAvFAAwA0ZUAsgJbzxcgBzAAo4ATjgjAswUVEYBGNuQCSIAG44ANlwAmAFWQBrUHKgAmEgF8SJUJFgIUaTLnyFS5ITIg8+qACKgXMA6LACQYQD0kVCAoqaA8IaAsHKAuEqA05qAaMrslDR0pmZMrBHRUACioqLIogBcUP5giNpUeMBQuqBYXABmwbIA5PDUtLy9AHRWQA)

- 第一个问题是：**重复的枚举项会导致报错**

​	这个问题相对不那么严重，因为 TS 在编译的时候就直接给出了明确的报错提示

- 第二个问题是：如果不给枚举初始值的话，按照之前的编译规则，会**自动给新的枚举项从 0 开始的枚举值**

​	这样的后果就是**双向映射被覆盖**（当然字符枚举就不会有这个问题了）

​	我们也可以从最后的 console 看到 0 的映射被后来声明的 PermissionDenied 覆盖了

​	解决这个问题的方法就是：**每次都记得给枚举项显式赋值**，不要依赖默认赋值

### 双向映射

数字枚举的双向映射可以让我们轻松做到如下操作：

```typescript
enum ResponseStatus {
  Success = 0,
  MissingParameter = 1,
  InvalidToken = 2
}

let curStatus = ResponseStatus.Success;   // 0
console.log(ResponseStatus[curStatus]);   // "Success"
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewDKAXAhls8UA3gFBRQZgDGVChAvFAAwA0ZUAsgJbzxcgBzAAo4ATjgjAswUVEYBGNuQCSIAG44ANlwAmAFWQBrUHKgAmEgF8SJTVKhUwo7HgKm4SVOhf54AOkoaOgBucigAenDmEiovZDs-TWQBAAoPFDRMXF8AbUdnbIIAXQBKUPJIqAAiQNpeKqA)

反之字符枚举就没办法：

```typescript
enum ResponseMsg {
  Success = 'Success',
  MissingParameter = 'Missing Parameter',
  InvalidToken = 'Invalid Token',
  UnknownErr = 'Unknown Error'
}

let message = ResponseMsg.MissingParameter;
console.log(ResponseMsg[message]);  // Error: Property 'Missing Parameter' does not exist on type 'typeof ResponseMsg'
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewCy8DmUDeAUFFAMpgDG5C8UAvFAORmXUMA0RUGAlvPNyGwAFAIYAnERGAAXYGLqMefAblESpsse04BJEADcRAG24ATACrIA1qAUM9hk6aiWbIbcQCqIKyGQB3EABRMXl6Bm9fAJAoELFkLQIAXwICIxkoKT4RbGAFOCRUdCxsADolfkE1SRk5AG4CciLkdNKjZGwACgKUNEwcAG0s+BzgAF0ASjriAHoZ2NCEgC4oIXjEOWkAT0VeStVxGs0GKFNkBCg-aShgAA9ea9QobY3GF+BkADNYBF7inAYQA)

<br />

看起来数字枚举会更灵活一点，但是更灵活真的是好事吗？看如下例子：

```typescript
enum ResponseStatus {
  Success = 0,
  MissingParameter = 1,
  InvalidToken = 2
}

function responseStatusHandler(status: ResponseStatus): void {
  // do something...
}
responseStatusHandler(ResponseStatus.Success);  // 符合预期
responseStatusHandler(0); // ok？？？
responseStatusHandler(6); // ok？？？

// -------------------------------------------------------------

enum ResponseMsg {
  Success = 'Success',
  MissingParameter = 'Missing Parameter',
  InvalidToken = 'Invalid Token',
  UnknownErr = 'Unknown Error'
}

function msgReporter(msg: ResponseMsg): void {
    console.log(msg)
}

msgReporter(ResponseMsg.Success);
msgReporter(0);   // Argument of type '0' is not assignable to parameter of type 'ResponseMsg'.

```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewDKAXAhls8UA3gFBRQZgDGVChAvFAAwA0ZUAsgJbzxcgBzAAo4ATjgjAswUVEYBGNuQCSIAG44ANlwAmAFWQBrUHKgAmEgF8SJAGZgQVLF1RRRCFGky588ABI4IDqaMgAU8D4EAFywHqjo2HgEAJQxasi6xOwA9NlQOshQ8MiSWAAW-AIAdDVWJO5I8d5J-oHBYXCNXom+VZQ0dMkA3OS5UIBk3oAQKoAhGYD45vVx3ZGtQSGioUzDUGNGgPj-+wtdCcsBq2EAbFs7hvu7NmMAtE-PL69v7x+fX282oJCxR2AHHgAiy5H6tF4pgA5BC6NClJweHxBCJxKUZDDuLxKlA0RIpDIEexVBptPojCZGNDSVpMgZjCBieQAKogQwgZAAdxAAFFRLJqWyOdyQFB+aJkKJoXU7A4nC4xRAQXAUKJpOtlQIYp1POhgQJUlB0plSORyFR4sgQlVNMgBKEtclZVrVVKNaFdU0DX1qJD4MMSK7gGqPZsRqM8gBBUQCSCgLBQZC2KBYACeiGAUGhTGhUB4UE5iZwOIEIBwACMQqnCogxASNUmU+nM9mvV4DdCqkA)

可以看到，上面的 `responseStatusHandler` 明明想要的是一个枚举类型，但是使用 `numer` 类型的 0 和 6 都成功通过 TS 的校验了（其中 6 甚至还不是枚举内的值）。

反观下面的 `msgReporter` 成功阻止了非枚举类型的传入。

综上所述：**如果你想用 TS 校验枚举类型，最好使用字符枚举**

当然，如果你嫌用枚举麻烦（跨模块使用函数如果函数想要枚举值作为形参则必须手动引入对应枚举），并且愿意牺牲一点代码可读性的话，完全**可以使用联合 `|` 来取代字符枚举**：

```typescript
enum ResponseMsg {
  Success = 'Success',
  MissingParameter = 'Missing Parameter',
  InvalidToken = 'Invalid Token',
  UnknownErr = 'Unknown Error'
}

// replace with..
type ResponseMsg = 'Success' | 'MissingParameter' | 'InvalidToken' | 'UnknownErr';
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/KYOwrgtgBASsDOAHA9iewCy8DmUDeAUFFAMpgDG5C8UAvFAORmXUMA0RUGAlvPNyGwAFAIYAnERGAAXYGLqMefAblESpsse04BJEADcRAG24ATACrIA1qAUM9hk6aiWbIbcQCqIKyGQB3EABRMXl6Bm9fAJAoELFkLQIAXwICAHo0qDFgRCMRKih-bmkACwA6MoJpAE9EYFgEFDRMHDtmKj4GKAAfRV5+QTVJGTku3vsDYzNXUDHGSL9AuIYAbiA)

### const enum

虽然枚举为我们的代码提供了不错的可读性，但是它还是产生了不少映射以及 IIFE。在大量使用的时候可能会造成额外的运行时间代价。

于是 const enum 就应运而生了：

```typescript
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}
 
let directions = [
  Direction.Up,
  Direction.Down,
  Direction.Left,
  Direction.Right,
];

// JS 版本
"use strict";
let directions = [
    0 /* Up */,
    1 /* Down */,
    2 /* Left */,
    3 /* Right */,
];
```

[Playground](https://www.typescriptlang.org/play?&preserveConstEnums=true#code/MYewdgzgLgBApmArgWxgEQJYCc7Ch8GAbwCgYYBVABwBoz0QB3MO8gGTgDMpWYAlDAHMAFjxIBfMiQA2cWABNsufOAgwAvDADa9TDjwEwAOmq89yw0bRMWupQfBGO3M-ZXGBIsQF0A3EA)

可以看出来，const enum 为我们做的事情很简单：

1. 它不再生成一个对象，也不建立映射关系

2. 它单纯的把枚举值内联到了所有有用到的地方

看起来似乎完美的解决了上述问题，但是真的有这么简单吗？

<br />

我们知道，const enum 在编译时会进行处理，而且 TS 独有的特性让我们可以在编译时完成 const enum 的内联，且在运行时导入其他含有同名 const enum 的枚举。这个时候，这两个本应该互相融合的枚举会变成两个不同的值。这个可能导致很多预期之外的 bug。

为了避免上述问题，有两个方案可以参考：

- **避免使用 const enum，或者设置 `preserveConstEnums `为 `true`**。后者会照常产生 IIFE 与对应映射。

- **只在自己能控制的程序中使用它**，并且不要把它往外发布或让其他人能够当成库导入。

### 总结

- 同名的 enum 会像 interface 一样自动合并，但是重复的枚举项会导致报错。

- 为了避免同名 enum 自动合并的时候导致双向映射被覆盖，最好对每个枚举值都显式赋值。

- 数字枚举的双向映射特性可能会给枚举类型的校验带来麻烦，如果想用 TS 校验枚举类型，最好使用字符枚举。

- 可以使用联合 `|` 来取代字符枚举。
- const enum 会通过内联的方式替换枚举值，可以减少普通枚举的开销。
- 把含有 const enum 的代码库导入的时候可能会导致预期之外的 bug，解决方式是设置编译选项 `preserveConstEnums `为 `true`。
- 如果一定要使用 const enum 的话，只在自己能控制的程序中使用它。

## Reference

https://juejin.cn/post/6844904112669065224#heading-3

https://www.php.cn/js-tutorial-479202.html

https://jkchao.github.io/typescript-book-chinese/typings/enums.html#%E6%95%B0%E5%AD%97%E7%B1%BB%E5%9E%8B%E6%9E%9A%E4%B8%BE%E4%B8%8E%E6%95%B0%E5%AD%97%E7%B1%BB%E5%9E%8B

https://medium.com/enjoy-life-enjoy-coding/typescript-%E5%96%84%E7%94%A8-enum-%E6%8F%90%E9%AB%98%E7%A8%8B%E5%BC%8F%E7%9A%84%E5%8F%AF%E8%AE%80%E6%80%A7-%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95-feat-javascript-b20d6bbbfe00

[Programming Typescript by Boris Cherny](https://www.amazon.com/Programming-TypeScript-Making-JavaScript-Applications/dp/1492037656)

[Effective TypeScript by Dan Vanderkam](https://www.amazon.com/Effective-TypeScript-Specific-Ways-Improve-ebook/dp/B07Z8HRZZ3)