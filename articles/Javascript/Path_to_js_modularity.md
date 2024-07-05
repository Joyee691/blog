# JS 的模块化之路

## 序

JS 作为一个脚本语言，在开发之初本就是为了执行独立的脚本任务

但是随着前端以及 JS 这个语言的发展，各种运行大量复杂 JS 脚本的程序应运而生

为了管理好这些庞大的程序，模块化的思想也被引入前端之中

本文从 JS 模块化的历程中，选出一些最具代表性的模块化方案介绍，保证看完能对 JS 的理解更加深入

## 为什么需要模块化

在介绍模块化方案之前，我们先回到一个最初的问题：**如果没有模块化，会有哪些问题？**

### 命名冲突

让我们先来看一个例子，我们有两个 js 文件，分别定义了两个同名的函数：

```js
// a.js
function sayHi(creator) {
  console.log('hi! ', creator)
}
```

```js
// b.js
function sayHi(creator) {
  console.log('Welcome! ', creator)
}
```

然后我们的页面同时引入了这两个文件：

```html
<!DOCTYPE html>
<html>

<head>
  <script src="a.js"></script>
  <script src="b.js"></script>
</head>

<body>
  <script>
    const creator = 'Joyee';
    sayHi(creator);  // Welcome!  Joyee
  </script>
</body>

</html>
```

可以看到，在上面这个 case 中，`b.js` 中的方法覆盖了 `a.js` 的同名方法，因为默认情况下，`script` 标签会按照顺序加载并执行

这种覆盖的特性在大型工程中会导致一系列的头疼问题，其次，你也没办法保证任意的一个三方脚本不会在无意中篡改了你的逻辑

### 大型工程管理

在大型工程中，为了提升代码的可维护性，一般会将代码拆分成多个独立的部分，如果没有模块化的支持，管理 script 标签的难度会直接上升到地狱级别，因为每次改动代码，要考虑的不仅仅包括是否要新增/删除 script 标签，还必须考虑到 script 标签的顺序是否需要调整

<br />

为了解决上述两个主要痛点，前端模块化方案应运而生

## 闭包 + IIFE

让我们直接看例子：

```js
var greeting = (function () {
    var module = {};

    var helloInLang = {
        en: 'Hello world!',
        es: '¡Hola mundo!',
        ru: 'Привет мир!'
    };

    module.getHello = function (lang) {
        return helloInLang[lang];
    };

    module.writeHello = function (lang) {
        document.write(module.getHello(lang))
    };
    
    return module;
}());
```

可以看到，这么做的好处是，我们可以把一个文件里面所有对外的变量全部封装起来，并且给到一个唯一的名字（唯一暴露一个全局对象）

这种方案虽然还是会污染全局变量，但优势在于可以在原生 js 下直接使用，非常方便

## CommonJS

CommonJs 是 nodejs 提供的原始模块化方案，因为它是基于 nodejs 实现的，所以在其他的 js 运行时中（比如浏览器）需要配合类似 Browserify 或 Webpack 的打包工具才能使用

### 使用方法

我们先用一个例子来看看它要如何使用：

我们要实现一个有加减功能的计算器，其中加跟减的方法会被抽象到一个 math 文件中：

```js
// math.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

// Exporting functions
module.exports = {
  add,
  subtract
};
```

math 文件通过 `module.exports` 将两个方法进行导出，然后我们就可以在 app 文件导入这两个方法了：

```js
// app.js
const math = require('./math');

const sum = math.add(2, 3);
const difference = math.subtract(5, 3);

console.log(`Sum: ${sum}`); // Output: Sum: 5
console.log(`Difference: ${difference}`); // Output: Difference: 2
```

### Commonjs 的特点

- **模块作用域**：Commonjs 提出了 “模块” 的概念，每一个 js 文件都是一个模块，所有的代码都运行在各自的模块作用域中，不会污染全局作用域

- **同步加载**：由于 nodejs 最初被设计为服务端服务，同步加载较为简单，并且对服务端影响不大；但是如果在浏览器使用则需要考虑同步阻塞的问题
- **模块缓存**：Commonjs 允许模块被多次加载，但是只有第一次加载会运行整个模块，加载完成后模块会被缓存起来，后续加载只会读取缓存结果
- **顺序加载**：模块加载顺序按照代码的执行顺序而定

### Commonjs 原理理解

Commonjs 最重要的有两个部分：**Module 对象**、**require 方法**

#### Module

前面我们说到，对 CommonJs 来说，每一个 js 文件都是一个 Module，那么这两者是怎么关联起来的呢？

事实上，当某个 js 文件被 require 的时候，nodejs 会把 js 文件用一个函数包裹起来：

```js
(function (exports, require, module, __filename, __dirname) {
  // The content of example.js goes here
});
```

举个例子，我们有一个 example.js 文件：

```js
// example.js
const name = 'CommonJS Module Example';

function greet() {
  console.log(`Hello from ${name}`);
}

// Export the greet function
module.exports = greet;
```

下面是它被包裹的样子：

```js
(function (exports, require, module, __filename, __dirname) {
  const name = 'CommonJS Module Example';

  function greet() {
    console.log(`Hello from ${name}`);
  }

  // Export the greet function
  module.exports = greet;
});
```

### require

上面我们描述了 js 文件是如何跟 module 关联起来的，接下来我们看看 nodejs 是如何导入 module 中导出的方法

让我们继续上面的例子，我们在 main.js 中使用 require 方法导入了 example.js 的 greet 方法：

```js
// main.js

// Require the example.js module
const greet = require('./example');

// Call the greet function from the module
greet(); // Output: Hello from CommonJS Module Example

```

`require` 方法究竟做了什么呢？让我们一步一步分析：

1. 解析参数：如果 nodejs 判断是文件路径的话，会直接定位到对应的文件；如果是包名，则会去 `node_modules` 中寻找
2. 缓存：定位到对应模块后，首先判断模块是否已被加载，如果有则直接返回之前的缓存；如果没有则创建该模块的实例后缓存实例
3. 加载模块：如果当前模块是第一次被加载，则会编译并运行整个模块（上述包裹的逻辑也是这个时候被执行的）
4. 返回 `module.exports`：加载完后，会返回模块实例中的 export 对象（在我们的例子中，这个对象是 `greet` 方法）
5. 调用 `greet`：最后，因为 `require` 返回了 `example.js` 导出的 `greet` 方法引用，所以 `main.js` 可以直接调用该方法

更加详细的逻辑可以在 nodejs 的 `lib/internal/modules/cjs` 文件中学习

## AMD (Asynchronous Module Definition)

AMD 是一个很常拿来跟 CommonJS 做对比的模块化方案，它与 CommonJS 最大的区别在于，它采取异步的方式加载所需模块

### 使用方法

老规矩，让我们从一个例子来认识 AMD 的使用方法：

```js
// 定义模块
define('myModule', ['dependency1', 'dependency2'], function(dep1, dep2) {
    // Define the module using dep1 and dep2
    var myModule = {
        doSomething: function() {
            console.log(dep1.someMethod());
            console.log(dep2.anotherMethod());
        }
    };

    return myModule;
});
```

```js
// 使用模块
require(['myModule'], function(myModule) {
    myModule.doSomething();
});
```

AMD 通过两个方法实现了模块化方案，我们先来看看 `define` 方法，`define` 方法接收三个参数：

- 第一个参数定义了模块的名称
- 第二个参数说明了此模块依赖哪些模块，在这个例子中，AMD 会把两个依赖模块加载完之后才会执行模块后面的方法
- 第三个参数可以定义该模块的方法，并且接受依赖模块作为参数传入

<br />

`require` 方法代表要引入什么模块，它也接受两个参数：

- 第一个参数代表依赖/要导入的模块
- 第二个参数代表当模块导入完了之后要调用的方法

### AMD 的特点

- 异步并行加载：通过异步并行加载能够很大程度避免加载过程中阻塞 DOM 渲染，更适合在浏览器中运行
- 依赖前置：会提前执行依赖的模块
- 前期成本较高：`define` 与 `require` 方法的使用使得代码编写与阅读比较困难；requireJS（实现 AMD 的库）甚至为了兼容非规范的模块而出了一个  **require.config()** 方法来定义其他库的导入导出行为以便与 AMD 模块协同

## UMD (Universal Module Definition)

UMD，这个名字非常霸气，它的目标是兼容上述三种不同的模块化方案

### UMD 原理

UMD 的原理其实很简单，它通过判断当前的 js 文件有哪一个模块的特征，然后对其使用对应模块话方案

判断流程大致如下：

1. 判断是否有 define 方法（如果有则用 AMD 方案）
2. 判断是否有 module 这个对象（如果有则是 CommonJS 方案）
3. 如果都没有，使用闭包方案

简单实现逻辑如下：

```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD. Register as an anonymous module.
        define([], factory);
    } else if (typeof module === 'object' && module.exports) {
        // Node. Does not work with strict CommonJS, but only CommonJS-like environments
        // that support module.exports, like Node.
        module.exports = factory();
    } else {
        // Browser globals (root is window)
        root.MyModule = factory();
    }
}(typeof self !== 'undefined' ? self : this, function () {
    // Define your module here
    var MyModule = {
        sayHello: function() {
            return 'Hello, world!';
        }
    };

    return MyModule;
}));
```

## ES6

在 ES6 规范形成之前，JS 的模块化方案一直是由社区来推动的，随着 ES6 规范的发布，意味着 JS 也有标准的模块化方案了

ES6 的模块化方案的设计思想是尽可能的静态化，通过在编译时加载对应模块，是它相较于社区方案的最大优势

如果想要在旧的工程项目中使用此新特性，可以使用 babel 等编译工具将其编译成 ES5 以前的代码

### 使用方法

#### 导出

ES6 的导出使用 `export` 关键字

可以导出对象、也可以导出简单类型：

```js
// math.js
export const pi = 3.14159;

export function add(x, y) {
  return x + y;
}

export function subtract(x, y) {
  return x - y;
}
```

如果只想要导出一个变量或者方法，可以使用 `export default`

```js
// greeting.js
export default function greet(name) {
  return `Hello, ${name}!`;
}
```

两者的区别：

|                  | 同一文件中的导出数量 | 导入的时候重命名的方法 |
| ---------------- | -------------------- | ---------------------- |
| `export`         | 没有限制             | 要使用 as 关键字       |
| `export default` | 只能有一个           | 可以随意命名           |

#### 导入

ES6 的导入使用 `import` 关键字

- 导入通过 `export` 导出的变量：

```js
// 可以用一个大括号选择要导入的变量；也可以使用 as 来重命名
import { pi, add as plus, subtract } from './math.js';

// 使用 * 指代全部；使用 as 重命名
import * as math from './math.js';
```

- 导入通过 `export default` 导出的变量：

```js
// 其中的 greet 可以换成任意的名字
import greet from './greeting.js';
```

#### 动态导入

JS 的模块化方案也包含了动态导入的解决方案，只需要把 import 关键字当作函数来调用，他就会返回一个 promise

动态导入的标准语法如下：

```js
import(moduleSpecifier)
  .then(module => {
    // Use the module
  })
  .catch(err => {
    // Handle error
  });

```

当然，它也可以配合 async/await 使用：

```js
async function loadModule(moduleSpecifier) {
  const module = await import(moduleSpecifier);
  // Use the module
}
```

## 总结

|          | 闭包           | CommonJS       | AMD                | UMD        | ES6                  |
| -------- | -------------- | -------------- | ------------------ | ---------- | -------------------- |
| 导入方式 | 全局获取       | require        | require            | require    | import               |
| 导出方式 | 挂载到全局     | module.exports | define             | exports    | export               |
| 加载方式 | 运行时加载     | 运行时同步加载 | 运行时并行异步加载 | 运行时加载 | 编译时加载、异步加载 |
| 实现模块 | 原生 JS 支持   | NodeJS         | RequireJS          | SeaJS      | 原生 JS 支持         |
| 适用场景 | 任何场景均支持 | 服务器         | 浏览器             | 浏览器     | 服务器/浏览器        |



## Reference

[The Evolution of JavaScript Modularity](https://github.com/myshov/history-of-javascript/tree/master/4_evolution_of_js_modularity#conclusion)