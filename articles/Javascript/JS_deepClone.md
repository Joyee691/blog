# JS 深拷贝

> 本文共 2768 字，建议阅读时间 15  min

## 前言

今天去面试遇到一个题目：实现一个 `deepClone` 函数实现对 JS 对象的深拷贝

我一听：这不简单？直接 `JSON.parse(JSON.stringify(obj))` 秒了

事实证明我还是太简单了，因为接下来面试官提了一个问题：如果对象包含循环引用呢？

我当下悬着的心终于还是死了🥲

所以就有了这篇文章🫠

## 一个普通的深拷贝函数

我们还是从最简单的形式来入手：手写一个深拷贝

如果把 JS 中的对象看成一棵树，对象的每个 key 都是一个父节点，每个 value 都是 key 的子节点，我们可以很自然的想到递归的方式来求解这个问题：

```js
function deepClone(obj) {
  // 对于普通类型/已经穷尽的情况，直接返回自身
  if(obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // 处理数组的情况：递归每一个数组的项
  if(Array.isArray(obj)) {
    const newArr = [];
    for(let i=0;i<obj.length;i++) {
      newArr[i] = deepClone(obj[i]);
    }
    return newArr;
  }
  
  // 处理一般对象的情况：递归每一个对象的项
  const newObj = {};
  for(const key in obj) {
    if(obj.hasOwnProperty(key))
      newObj[key] = deepClone(obj[key]);
  }
  return newObj
}
```

可以看到，其实深拷贝的核心思路就是：

1. 对于普通类型：直接不管，原路返回
2. 对于引用类型：创建一个新的 “壳子”，把里面的东西 “搬过去”

## 兼容循环引用的深拷贝函数

实现了基础的深拷贝之后，我们下一步就是要 “拷贝” 循环引用了

循环引用的本质就是：**对象中的某个 key 的 value，是对其他 key 的 value 的引用**

如果觉得太过抽象可以参考如下例子：

```js
const original = {
  a: 1,
  b: { c: 2 },
  d: {},
  e: {},
};
original.d.referenceToE = original.e;
original.e.referenceToD = original.d;
```

那么我们只需要多给我们的方法一个参数，用来记录已经拷贝过的对象拷贝前后的引用就好，我们选择 Map

在这里我们顺便把之前没考虑的 Map, Set, RegExp, Date 类型也一起加上了：

```js
function deepClone(data, seen = new Map()) {
  if (data === null || typeof data !== 'object') {
    return data;
  }

  // 如果当前的对象是之前复制过的，直接把之前复制完的引用返回就好
  if (seen.has(data)) {
    return seen.get(data);
  }

  // Array
  if (Array.isArray(data)) {
    const newArr = [];
    // 记得要记录一下新旧值的对应关系
    seen.set(data, newArr);
    for (let i = 0; i < data.length; i++) {
      newArr[i] = deepClone(data[i], seen);
    }
    return newArr;
  }

  // Date
  if (data instanceof Date) {
    return new Date(data.getTime());
  }

  // RegExp
  if (data instanceof RegExp) {
    return new RegExp(data)
  }

  // Set
  if (data instanceof Set) {
    const newSet = new Set()
    // 记得要记录一下新旧值的对应关系
    seen.set(data, newSet);
    data.forEach(item => {
      newSet.add(deepClone(item, seen))
    })
    return newSet
  }

  // Map
  if (data instanceof Map) {
    const newMap = new Map()
    // 记得要记录一下新旧值的对应关系
    seen.set(data, newMap);
    data.forEach((value, key) => {
      newMap.set(key, deepClone(value, seen))
    })
    return newMap
  }

  // Object
  const newObj = {};
  // 记得要记录一下新旧值的对应关系
  seen.set(data, newObj);
  for (let key in data) {
    if (data.hasOwnProperty(key)) {
      newObj[key] = deepClone(data[key], seen);
    }
  }
  return newObj;
}
```

## 一个小问题——函数的深拷贝

以为到这里就结束了？不，这才刚刚开始😂

在跟面试官讨论上诉问题的过程中，我无意间想到了一种类型：Function

于是乎，新一轮的问题又出现了，我们怎么（或者说有办法）给 js 的函数深拷贝？

在解答这个问题之前，我们要回顾一下深拷贝的含义：**深拷贝指创建一个其属性与其拷贝的源对象的属性不共享相同的引用（指向相同的底层值）的副本**

说人话：表现必须一样，但是引用不能指向同一块内存

### 思路一：包裹函数

先看看代码：

```js
function originalFn(a, b) {
  return a + b;
}

const newFn = (...args) => originalFn(...args);

const originalRes = originalFn(1, 2);
const newRes = newFn(1, 2);
console.log(originalRes === newRes) // true
console.log(newFn === originalFn) // false
```

emmmm 怎么说呢，表现确实一样，确实也没有指向同一块内存……但是非常 tricky，像在玩文字游戏（好吧😑 pass）

### 思路二：bind

js 的 bind 方法给我们提供了一个解决思路：bind 方法会返回一个新的函数对象，并且将此函数对象的 this 指向 bind 的第一个参数

看看代码：

```js
function originalFn(a, b) {
  return a + b;
}

// 通过 bind 返回了一个新的函数对象
const newFn = originalFn.bind({});

const originalRes = originalFn(1, 2);
const newRes = newFn(1, 2);
console.log(originalRes === newRes) // true
console.log(newFn === originalFn) // false
```

但是这种方法也有其局限性：**对于在函数体中使用了 this 指向的方法，可能会有预期之外的表现**

所以这个不能视为一种通用方法

### 思路三：序列化与反序列化

这个思路会用到 js 中一个非常危险的操作：`eval`，所以仅仅作为一种思路的拓展，我也会在后面详细说明为什么这个操作非常危险

这个思路的核心其实就一句话：**我们把要深拷贝的目标函数先序列化为字符串后，然后再新的对象中动态创建出来**

看看代码：

```js
function originalFn(a, b) {
  return a + b;
}

// 序列化函数之后，用 eval 将其反序列化为新的函数并返回给 newFn
const newFn = eval(`(${originalFn})`)

const originalRes = originalFn(1, 2);
const newRes = newFn(1, 2);
console.log(originalRes === newRes) // true
console.log(newFn === originalFn) // false
```

为什么说这个操作很危险呢？主要体现在：

1. 安全：使用 eval 可以运行任何的 js 逻辑，如果将这个方法提供在公用方法上，很可能被人利用来执行恶意代码
2. 作用域：序列化方法会丢失他们原本的作用域与闭包，这个可能会让代码有预期之外的表现
3. 性能：序列化与反序列化会有额外的成本，使用 eval 也会导致很多性能优化无法使用，进而拖慢整个程序运行性能
4. 维护性：使用 eval 的代码在阅读、debug、维护上的成本都特别高

### 结论

一般不建议对函数使用深拷贝，如果需要，建议在函数是可控的情况下选择对应的方案

## 通用解决方案

接下来，我们看看业界通用的解决方案是如何解决深拷贝的这个问题

### Lodash

loadash 对于深拷贝的核心逻辑在 [cloneDeep](https://github.com/lodash/lodash/blob/main/src/cloneDeep.ts)，cloneDeep 调用了通用的拷贝函数 `baseClone`，我在这节选了一部分，并且加了一些注释：

```js
/**
 * The base implementation of `clone` and `cloneDeep` which tracks
 * traversed objects.
 *
 * @private
 * @param {*} value The value to clone.
 * @param {number} bitmask The bitmask flags.
 *  1 - Deep clone
 *  2 - Flatten inherited properties
 *  4 - Clone symbols
 * @param {Function} [customizer] The function to customize cloning.
 * @param {string} [key] The key of `value`.
 * @param {Object} [object] The parent object of `value`.
 * @param {Object} [stack] Tracks traversed objects and their clone counterparts.
 * @returns {*} Returns the cloned value.
 */
function baseClone(value, bitmask, customizer, key, object, stack) {
  let result
  const isDeep = bitmask & CLONE_DEEP_FLAG
  const isFlat = bitmask & CLONE_FLAT_FLAG
  const isFull = bitmask & CLONE_SYMBOLS_FLAG

  // 是否有客制化逻辑？（cloneDeep 不涉及） 
  if (customizer) {
    result = object ? customizer(value, key, object, stack) : customizer(value)
  }
  if (result !== undefined) {
    return result
  }
  
  // 对于非引用类型，直接返回值
  if (!isObject(value)) {
    return value
  }
  const isArr = Array.isArray(value)
  const tag = getTag(value)
  if (isArr) {
    // 对于数组类型，在深拷贝的情况下会先创建一个空数组，然后不会进入下面的 if 逻辑
    // 而是走到最后的 arrayEach 方法中对每个元素进行深拷贝
    result = initCloneArray(value)
    if (!isDeep) {
      return copyArray(value, result)
    }
  } else {
    const isFunc = typeof value === 'function'

    if (isBuffer(value)) {
      return cloneBuffer(value, isDeep)
    }
    if (tag === objectTag || tag === argsTag || (isFunc && !object)) {
      result = (isFlat || isFunc) ? {} : initCloneObject(value)
      if (!isDeep) {
        return isFlat
          ? copySymbolsIn(value, copyObject(value, keysIn(value), result))
          : copySymbols(value, Object.assign(result, value))
      }
    } else {
      // 对于函数类型，会直接返回它的原函数引用
      if (isFunc || !cloneableTags[tag]) {
        return object ? value : {}
      }
      result = initCloneByTag(value, tag, isDeep)
    }
  }
  // 针对循环引用的处理，这里的 Stack 类似 Map
  stack || (stack = new Stack)
  const stacked = stack.get(value)
  if (stacked) {
    return stacked
  }
  stack.set(value, result)

  if (tag === mapTag) {
    value.forEach((subValue, key) => {
      result.set(key, baseClone(subValue, bitmask, customizer, key, value, stack))
    })
    return result
  }

  if (tag === setTag) {
    value.forEach((subValue) => {
      result.add(baseClone(subValue, bitmask, customizer, subValue, value, stack))
    })
    return result
  }

  if (isTypedArray(value)) {
    return result
  }

  const keysFunc = isFull
    ? (isFlat ? getAllKeysIn : getAllKeys)
    : (isFlat ? keysIn : keys)

  const props = isArr ? undefined : keysFunc(value)
  // 对于一些可以遍历的类型，递归的去深拷贝他们的子节点
  arrayEach(props || value, (subValue, key) => {
    if (props) {
      key = subValue
      subValue = value[key]
    }
    // Recursively populate clone (susceptible to call stack limits).
    assignValue(result, key, baseClone(subValue, bitmask, customizer, key, value, stack))
  })
  return result
}
```

可以看到，其实 lodash 的实现逻辑跟我们 “兼容循环引用的深拷贝函数” 章节中的思路差不多

区别在于，它考虑了更多的类型的情况，并且写成了一个更加通用的拷贝函数

此外，对于函数 lodash 也是直接返回了原函数的引用

### structuredClone 方法

structuredClone 是一个 JS 挂载在全局上的深拷贝方法，不过使用前请记得看看 [caniuse](https://caniuse.com/?search=structuredClone) 确保你的项目环境支持

下面我简单讲一下 node 是如何实现这个方法的

这个方法第一次被 node 官方挂载到全局上是在 node 17.x 版本，并且在 node 21.2 开始有了一个较大的重构，我们分别来讲讲思路

#### node 17.x

[源码](https://github.com/nodejs/node/blob/v17.0.0/lib/internal/structured_clone.js)

按照惯例，我也把他核心部分节选下来：

```js
function structuredClone(value, options = undefined) {
  // TODO: Improve this with a more efficient solution that avoids
  // instantiating a MessageChannel
  channel ??= new MessageChannel();
  channel.port1.unref();
  channel.port2.unref();
  channel.port1.postMessage(value, options?.transfer);
  return receiveMessageOnPort(channel.port2).message;
}
```

可以看到，这一段代码就单纯的在 channel 的两个端口之间传了一个消息，为什么这么做能实现深拷贝呢？

根据 [The structured clone algorithm](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) 这篇文章描述的，JS 引擎在 postMessage 的过程中会调用 v8 提供的一个算法，这个算法包含了一对序列化与反序列化的逻辑，这套序列化逻辑不依赖 JSON 的方案间接实现了深拷贝，并且还支持循环引用

#### node 21.2

[源码](https://github.com/nodejs/node/blob/v21.2.0/src/node_messaging.cc#L1559)

我在把其中比较重要的逻辑摘录下来：

```cpp
// node_messaging.cc

static void StructuredClone(const FunctionCallbackInfo<Value>& args) {
  // ... Some other codes ...

  if (args.Length() == 0) {
    return THROW_ERR_MISSING_ARGS(env, "The value argument must be specified");
  }

  Local<Value> value = args[0];

  // ... Some other codes ...
  
  Local<Value> result;
  // 通过 Serialize 与 Deserialize 方法，最终会调用到 v8 提供的序列化/反序列化方法，达成深拷贝的目的
  if (msg->Serialize(env, context, value, transfer_list, Local<Object>())
          .IsNothing() ||
      !msg->Deserialize(env, context, nullptr).ToLocal(&result)) {
    return;
  }
  args.GetReturnValue().Set(result);
}
```

可以看到，这一版本的实现更加简单直接，把深拷贝与 postMessage 的逻辑解耦开来了

当然，这一套算法也有一些不适用的数据结构，比如我们上文提到的 Function，个人猜测是因为会丢失函数闭包与上下文的关系才做了禁止，不支持的数据结构列表可以见：https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#things_that_dont_work_with_structured_clone

### 总结一下

如果是在面试场景，一般写出兼容循环引用的深拷贝就可以了，但是如果在实际开发过程中，还是建议优先使用原生提供的 structuredClone 方法，它的适用性更强

如果生产环境中还不支持这个方法，则可以使用 lodash 提供的 cloneDeep 方法
