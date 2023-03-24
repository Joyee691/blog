# 控制反转(IoC)与依赖注入(DI)

## 控制反转 Inversion of Control

### 说明

它是面向对象编程的一种设计原则，用来降低代码之间的耦合度。

基本思想：借助“第三方”实现有依赖关系对象之间的解耦。

![coupling](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/coupling.webp)

![decoupling.webp](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/decoupling.webp)

### 命名由来

---

1. 在左图中，对象A依赖于对象B，所以A在使用到B时必须主动创建(new)一个对象B，这个时候**主动权在A自己手上**
2. 在右图中， 引入了**IOC容器**，IOC容器在A需要使用到B的时候会**主动创建一个对象B之后注入到A中**
3. 综上所述，A在获得依赖对象B的过程由**主动变为了被动**，*控制权反转了*

## 依赖注入 Dependency Injection

### 说明

将实例变量传入到一个对象中去

### 传统依赖

---

```jsx
public class Human {
    ...
    Father father;
    ...
    public Human() {
        father = new Father();
    }
}
```

#### 存在的问题

1. 如果需要改变Father的方式（比如加一个参数），需要修改Human代码
2. Father对象的初始化被写死在Human构造函数中，无法修改
3. 执行new Father()的过程可能非常慢，无法使用预先初始化好的Father对象mock掉这个过程

### 依赖注入

---

```jsx
public class Human {
    ...
    Father father;
    ...
    public Human(Father father) {
        this.father = father;
    }
}
```

#### 特点

1. 将father对象作为一个参数传入Human构造函数，并且在调用Human构造方法之前就已经初始化好了Father对象
2. 将依赖之间解耦
3. 方便做单元测试，尤其是mock测试

## 控制反转与依赖注入的关系

- 控制反转是一种思想
- 依赖注入是一种设计模式