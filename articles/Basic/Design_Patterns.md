# Design Patterns

## 策略模式

- 思想：定义算法族，分别封装，让他们可以互相替换，让算法的变化独立于使用其他算法的客户

![9-1](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-1.jpeg)

## 观察者模式

- 思想：在对象间建立一对多的依赖，这样当一个对象发生改变，依赖它的对象就会收到通知并自动更新。

![9-2](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-2.jpeg)

## 装饰者模式

- 思想：动态的将“装饰品”附加在对象上，比传统继承拥有更高的运行间自由度与弹性
- 缺点：可能导致过多子类与代码复杂度，造成理解困难

![9-3](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-3.jpeg)

## 工厂方法

- 思想：定义一个创建对象的接口，但是由子类决定要实例化的类是哪一个。它将类的实例化推迟到了子类

![9-4](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-4.jpeg)

## 抽象工厂

- 思想：提供一个接口，用于创建相关或者依赖对象的**家族**，而不需要明确指定具体类。

![9-5](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-5.jpeg)

## 单例模式

- 思想：确保一个类只有一个实例，并提供全局访问点。
- 涉及多线程时的处理方法
    1. 将getInstance方法变成同步(synchronized)方法
    2. 在静态初始化时就创建单件实例
    3. 使用双重检查锁，只有第一次创建单例的时候同步方法

![9-6](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-6.jpeg)

## 命令模式

- 思想：通过将请求封装成对象，对发出请求对象与执行请求的对象之间解构，可以支持撤销动作
- 用途：队列请求，日志请求

![9-7](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-7.jpeg)

## 适配器模式

- 思想：通过适配器，可以将一个类的接口转换成另一个接口，让原本不兼容的类可以互相合作

![9-8](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-8.jpeg)

## 外观模式

- 思想：提供一个统一的高层次接口来访问子系统中的一群接口，简化子系统的使用成本

![9-9](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-9.jpeg)

## 模版方法

- 思想：在一个方法中定义算法的“骨架”，而将一些算法步骤延迟到子类实现，子类可以在不改变算法结构之下重新定义某些步骤
- 例子：钩子hooks

![9-10](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-10.jpeg)

## 迭代器

- 思想：提供一种方法顺序访问一个聚合对象的各个元素，同时不会曝露其中的内部表示

![9-11](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-11.jpeg)

## 组合

- 思想：用树形结构来表现对象的“整体/部分”层次结构。使得用户能够以一致的方式处理多个对象的组合

![9-12](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-12.jpeg)

![9-13](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-13.jpeg)

## 状态

- 思想：将对象的行为与内部的状态做绑定，在内部状态改变的时候改变对应的行为。对于对象而言好像改变了它的类。
- 与策略模式的不同
    - 策略模式使用行为或算法配置Contex类，可以在运行时修改
    - 状态模式使用对象改变状态，所有的状态改变被事先定义好了，对外不可见。

## 代理

- 思想：为另一个对象提供一个替身代表用来控制客户对这个对象的访问
- 代理种类：
    1. 远程代理：控制客户与远程对象的交互
    2. 虚拟代理：控制访问实例化开销大的对象，能在等待期间提供简单交互
    3. 保护代理：对于不同的调用者控制对象方法的访问
    4. 缓存代理：为开销大的运算结果提供暂时的缓存
    5. 防火墙代理：控制网络资源的访问
    6. 智能引用代理：在主题被引用时，提供额外的动作，如：计算对象被引用次数
    7. 同步代理：为多线程主题提供安全的访问
    8. 外观代理：隐藏复杂集合的复杂度并提供访问控制
    9. 写入时复制代理：延迟对象的复制直到客户真的需要为止

## 桥接 Bridge

- 思想：通过将实现与抽象放在两个不同的类层次从而使得两者能独立改变
- 优点：将实现与抽象解构，双方都可以独立扩展不会影响到对方
- 缺点：增加了复杂度

![9-14](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-14.jpeg)

## 生成器 Builder

- 思想：将复杂对象的创建过程封装起来，允许对象通过多个步骤来创建，也可以改变过程
- 优点：
    1. 隐藏了产品内部表现
    2. 产品实现可以被替换（客户只看得到抽象接口）
    3. 与工厂模式单一步骤相比，更加灵活，也对客户的专业程度有更高要求

![9-15](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/9-15.jpeg)

## 责任链 Chain of Responsibility

- 思想：每个对象依次检查请求，对其进行处理或传给下一个对象
- 优点：
    1. 将请求的发送者与接受者解构
    2. 简化对象，因为对象不需要知道链的结构
    3. 可以动态改变链的成员或顺序（新增或删除责任）
    4. 所有未被处理的请求都会被落到链尾
- 缺点：不易观察运行时特征，增加除错难度
- 类似实现：Error handler

## 蝇量 Flyweight

- 思想：让类的一个实例提供很多“虚拟实例”，从而减少内存开支
- 优点：
    1. 减少运行时实例个数，减少内存
    2. 对虚拟对象状态集中管理
- 缺点：单个逻辑实例无法拥有独特行为

## 中介者 Mediator

- 思想：使用中介者可以集中不同对象间复杂的交互与控制模式
- 优点：
    1. 通过对象彼此解构，能增加对象的复用性
    2. 集中控制逻辑，可以简化系统维护
    3. 减少并简化对象之间所传递的消息

## 备忘录 Memento

- 思想：创建一个专门储存系统关键对象状态的对象
- 目标：
    1. 储存系统关键对象状态
    2. 维持关键对象的封装
- 优点：
    1. 将储存状态的对象放在关键对象外面，帮助维护内聚性
    2. 提供了容易实现的恢复能力
- 缺点：保存与恢复过程可能相当耗时