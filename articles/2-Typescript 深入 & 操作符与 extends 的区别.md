# Typescript: 深入 & 操作符与 extends 的区别

> 全文共 389 字，建议阅读时间 5 min

下面是一个类型拓展的两种常见用法：

```typescript
// base type
interface Shape {
  color: string;
}

// extension
interface Square1 extends Shape {
  width: number;
}

// intersection
type Square2 = Shape & {
  width: number;
}
```

[Playground Link](https://www.typescriptlang.org/play?#code/PTAECMEMGcFNQC4E8AOsBQBLAdg2AnAM0gGN4BlAC0jVAG91RQSB7AGxfwC5RoF8cAcwDc6AL7p0IULAAeebNEwtsWXAWJlQ5AI4BXSPlgBGGfNjYAJtG3VaDJgHdMlhJR7Y9AW3AFREqTAcPHw4EgRlVWRaXQMjACZQAF5bGngAMnpGUGdXd1BPHz9xIA)

看起来没有什么不同对吧？那么看看下面的例子：

```typescript
interface Square {
  width: number;
}

interface Rectangle1 extends Square {	// Type 'string' is not assignable to type 'number'
  width: string;
  height: string;
}

type Rectangle2 = Square & {	// OK
  width: string;
  height: string;
}

let rectangle: Rectangle2 = {
  width: '10px',		// Type 'string' is not assignable to type 'never'.
  height: '12px'
}
```

[Playground Link](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgMoEcCucooN4BQyyA7sACZgAWAXMiJgLYBG0A3AQL4EGiSyIUAJQgIwcEAHMANhACMyCAA9IIcgGc0WHPgCQAen3IAKgE8ADigDk6sFFCSryYJpAB7MMjjr1wSSDhmWWQwNxCLawYWaCsiUgpqOlt7KQ5iKgg-KjAkuwcObgIwCOQRMQkZCAAmZABeLWxcZAAyZDwDIwB5AGk4skpaZGT8uIysnKG81K4eWU9ccqlZOjLxJeq6tr6Ewas5AAZzJSsAGl0OkxKbKcdnVw8vHz8AoJRQ8MtkKxAIADcYgB0o0ykmydD2VSOsU4QA)

上面的例子用了两种方法拓展了不同类型的 width，我们可以观察到两点区别：

1. 使用 `extends` 的时候马上就报错了（line 5）
2. 使用 `&` 的时候会将 width 的类型指派为 never，后面我们把 string 类型分配给 never 才报错（line 16）

由此我们可以得到第一个结论：**当遇到对象属性类型冲突的时候，& 会给冲突属性 never 类型，而 extends 会直接报错**

<br />

接下来，我们再看看函数的例子：

```typescript
interface StringToNumber {
  convert: (value: string)=>number;
}

interface NumberToString {
  convert: (value: number)=>string;
}

// Named property 'convert' of types 'StringToNumber' and 'NumberToString' are not identical.
interface BidirectionalConvert1 extends StringToNumber, NumberToString {}

// {convert: (value: string | number) => string | number;}
type BidirectionalConvert2 = StringToNumber & NumberToString
```

[Playground Link](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgMpiqA5gFQPYByArgLYBG0yA3gFDLIJ4gBu0YAXMgBTNwA2RCJwDOGbAEoAvAD4QpClADcNAL40aoSLEQpi5aPnSYQWanQZNWUDt14ChyOfqhTpo41mVqaAeh-ICOBIIABNkAAcoPHC2AE9kAHJGFjYE5DwYZDBYmOFEo2x8PQU0uBAwhOKDPAKTUqgUEDwwZGAQiHBgBH4AOg1waHgkZAAhNuAGhDBgJn4AYUs2AEZkCAAPSHK82txCeWgAGgD9qEMxE2pvP2pkqxsefkERc9MAH0cT8WQZZHdsZHeTgUijU2Rio3Gk2msz4CxS1gATN80C8iidkAAyY7OM4eIA)

自此，我们可以得到第二个结论：**当遇到函数类型冲突的时候，& 会联集函数的参数与返回值，而 extends 会直接报错**

## Reference

[Difference between extending and intersecting interfaces in TypeScript?](https://stackoverflow.com/questions/52681316/difference-between-extending-and-intersecting-interfaces-in-typescript)

[Interfaces vs Intersections](https://www.typescriptlang.org/docs/handbook/2/objects.html#interfaces-vs-intersections)