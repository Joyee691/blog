# 数据库完整性

数据库的完整性时指数据的**正确性**与**相容性**

数据的正确性：表示数据是否符合**现实世界语义**，反应了当前实际状况，比如身份证号是唯一的

数据的相容性：数据库中同一对象在不同关系表中的数据是符合逻辑的，比如学生的院系必须是学校已有的院系

<br />

数据的完整性与安全性的区别：

- 数据完整性防止数据库中存在不符合语义的数据，防范的是不合语义、不正确的数据
- 安全性保护数据库防止恶意的破坏与非法存取，防范的是非法用户与非法操作

<br />

数据库在完整性方面应该具备的功能：

- 提供定义完整性约束条件的机制，包括实体完整性、参照完整性、用户定义的完整性
- 提供完整性检查方法，一般在 INSERT, UPDATE, DELETE 执行后开始检查
- 违约处理：如果 DBMS 发现用户操作违背了完整性约束，应该采取**拒绝其操作**，或者**级联其操作**的方式保证完整性

## 实体完整性

在现实世界中，实体是独一无二的

映射到数据库中我们可以通过**设置主码**保证每一行数据都能相互区分

主码可以是单属性也可以是多属性的，区别在于：

- 单属性主码：可以定义为列级的约束条件，也可以定义为表级的约束条件
- 多属性主码：只能定义为表级的约束条件

### 实体完整性检查与违约处理

RDBMS 会按照实体完整性规则自动进行检查，检查内容包括：

- 检查主码值是否唯一，如果不唯一则拒绝插入或修改
- 检查主码的各个属性是否为空，只要有一个为空就拒绝插入或修改

DBMS 一般会在主码上自动建立一个索引（比如 B+ 树索引）来加快检索速度

## 参照完整性

参照完整性主要关心数据之间的相容性

在数据库中，我们使用外键 foreign key 来定义参照完整性，其中 references 短语表示这些外键参照哪些表的主码

### 参照完整性检查与违约处理

因为参照完整性会将两个表中相应元组联系起来，所以对被参照表与参照表进行的增删改查操作都有可能破坏参照完整性

参照完整性违约处理策略：

- 拒绝执行（默认策略）
- 级联操作
- 设置为空值

## 用户定义的完整性

用户定义的完整性表示针对某一个具体应用的数据必须满足的语义要求

用户定义的完整性由 RDBMS 提供了检验机制，不必由应用程序来承担检查工作

### 属性上的约束条件

创建表时可以给属性添加的约束包括：

- 列值非空（NOT NULL）
- 列值唯一（UNIQUE）
- 检查列值是否满足一个条件表达式（CHECK）

如果在插入或修改属性值时，属性上的约束条件不被满足则会被**拒绝执行**

### 元组上的约束条件

元组（行）上的约束条件可以用 CHECK 短语定义

元组级别的限制可以设置不同属性之间的取值的相互约束条件

例子：

```sql
// 建立一个 student 表，约束当性别为男时，其名字不能以 Ms. 开头
CREATE TABLE Student (
	Sno CHAR(9),
	Sname CHAR(8) NOT NULL,
	Ssex CHAR(2)
	// 元组约束，check 中表示的是能通过的条件
	CHECK (Ssex='女' OR Sname NOT LIKE 'Ms.%')
```

## 完整性约束命名子句

### 建立表中的完整性限制

格式：

```sql
CONSTRAINT <condition name> <condition>
```

condition 代表完整性约束条件，包括：

- not null
- unique
- primary key
- foreign key
- check

例子：

```sql
// jianlistudent 表，要求:
// 1. 学号在 90000~99999 之间
// 2. 姓名不能为空
// 3. 年龄小于 30
// 4. 性别只能是 m 或 f
CREATE TABLE Student (
	Sno NUMERIC(6) CONSTRAINT C1 CHECK (Sno BETWEEN 90000 AND 99999)
	Sname CHAR(20) CONSTRAINT C2 NOT NULL
	Sage NUMERIC(3) CONSTRAINT C3 CHECK (Sage < 30)
	Ssex CHAR(1) CONSTRAINT  C4 CHECK(Ssex IN ('m', 'f'))
	CONSTRAINT  StudentKey PRIMARY KEY(Sno))
) 
```

### 修改表中的完整性限制

格式：

```sql
ALTER TABLE <table name> DROP CONSTRAINT <condition name>
```

## 域中的完整性限制

于是一组具有相同数据类型的值的集合，简单来说就是属性的取值范围

可以使用 **create domain** 来建立一个域以及定义该域应该满足的完整性约束条件，之后可以用这个域来定义属性

例子：

```sql
// 建立一个性别域
CREATE DOMAIN GenderDomain CHAR(1)
CHECK(VALUE IN ('f', 'm'))

// 使用域来定义性别属性
Ssex GenderDomain
```

## 断言

断言也是设置完整性的一种方式

断言的开销一般比较大，可以考虑在应用程序中添加限制

### 设置断言

格式：

```sql
// 每个断言都会被赋予一个名字，断言的条件使用 check 子句来表示
CREATE ASSERTION <assertion name> <check clause>
```

例子：

```sql
// 限制数据库课程最多 60 名学生选修
CREATE ASSERTION dbMax CHECK(60 >= (select count(*) FROM 
	COURCE, SC WHERE SC.Cno = COURCE.Cno and COURCE.Cname='db' )
```

### 删除断言

格式：

```sql
DROP ASSERTION <assertion name>
```

## 触发器

触发器 trigger 是用户定义在关系表上的一类由事件驱动的特殊过程

触发器有如下特点：

- 触发器保存在数据库服务器中
- 任何用户对表的增删改查都会由服务器自动激活触发器
- 触发器可以实施更为复杂的检查与操作，具有更加精细和更加强大的数据控制能力

### 触发器的定义

触发器遵循 “事件-条件-动作” 的规则，也就是说，必须先有一个事件，触发了某种条件，之后才会执行某个操作

格式：

```sql
CREATE TRIGGER <trigger name>
	{BEFORE | AFTER} <trigger event> ON <table name>
	REFERENCE NEW|OLD ROW AS <variable>
	FOR EACH {ROW|STATEMENT}
	[WHEN <condition>]<action>
```

说明：

- 只有表的拥有者才可以在表上建立触发器
- 触发器的名字可以包含模式名，也可以不包含
- 同一模式下触发器名必须是唯一的
- 触发器名与表名必须是在同一模式下
- 触发器只能定义在基本表上，不能定义在视图上
- 触发器的 action 可以是 INSERT, DELETE, UPDATE 或这几个动作的组合
- 可以使用 UPDATE OF<triggered row> 来进一步指明修改哪些列
- BEFORE｜AFTER 指的是触发器触发的时机在事件的先或后
- 触发器类型有行与列两种，使用 FOR EACH ROW 代表行级触发器；使用 FOR EACH STATEMENT 表示语句级触发器
- 行级别的触发器会将所有符合条件的数据都执行一次 action
- 语句级的触发器只会执行一次
- WHEN 语句表示触发器的条件，只有条件为真时才会执行 action；如果省略 WHEN 语句，则只要事件发生就会执行
- 最后的 action 代表触发的动作体，它可以是一个结构化 SQL 语句，也可以是对一创建存储过程的调用
    - 如果是行级别的触发器，用户可以使用 NEW 或者 OLD 引用事件前后的新旧值；如果是语句级别的触发器则不能使用
    - 如果执行失败，激活触发器的事件会终止执行

### 触发器的激活

触发器是有书法时间激活的，并且由数据库服务器自动执行

一个数据表中可能定义多个触发器，多个触发器遵循以下执行顺序：

1. 先执行 BEFORE 触发器
2. 再执行触发器的 SQL 语句
3. 最后执行表上的 AFTER 语句

### 触发器的删除

格式：

```sql
DROP TRIGGER <trigger name> ON <table name>
```

说明：

- 必须是已经创建的触发器才能删除
- 触发器只能由拥有相应权限的用户删除