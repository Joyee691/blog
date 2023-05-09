# 关系数据库标准语言 SQL

## SQL

SQL(Structured Query Language) 是一种结构化查询语言，是关系数据库中的标准语言

SQL 的特点：

- 集数据定义语言 DDL、数据操纵语言 DML、数据控制语言 DCL 功能于一体
- 空容易独立完成数据库声明周期中的全部活动（定义关系模式、插入数据、建立数据库、查询、更新、数据库重构与维护、数据库安全性与完整性控制）
- 可以随时修改数据库模式，不影响数据运行
- 数据操作符统一
- 高度非过程化，只需要提出“做什么”就好
- 面向集合的操作方式，操作对象与查找结果可以是元组的集合
- 以同一种语法结构提供多种使用方式（SQL 可以独立使用，也能嵌入 C++ 等语言使用）
- 语法简洁，只有 9 个动词：

| 功能 | 动词 |
| --- | --- |
| 数据查询 | SELECT |
| 数据定义 | CREATE, DROP, ALTER |
| 数据操纵 | INSERT, UPDATE, DELETE |
| 数据控制 | GRANT, REVOKE |

## SQL 的基本概念

SQL 支持关系数据库中的三级模式结构：

- 外模式在 SQL 中被称为**视图**，是多个数据库表根据需求整合成的一张虚拟表
- 模式在 SQL 中被称为**基本表**，是数据库维护的数据表，一个关系对应着一个基本表
- 内模式在 SQL 中被称为**存储文件**，是数据在硬盘中的实际存储

## SQL 的数据定义

SQL 中支持数据定义的功能有：模式定义、表定义、视图定义、索引定义

| 操作对象 | 创建 | 删除 | 修改 |
| --- | --- | --- | --- |
| 模式 | CREATE SCHEMA | DROP SCHEMA |  |
| 表 | CREATE TABLE | DROP TABLE | ALTER TABLE |
| 视图 | CREATE VIEW | DROP VIEW |  |
| 索引 | CREATE INDEX | DROP INDEX | ALTER INDEX |

### 操作模式

如果把数据库比喻成一栋楼，那么模式就是楼里面的房间，模式保存着两个实体之间的关系

一、定义模式

```sql
CREATE SCHEMA <ModelName> AUTHORIZATION <UserName>[<Define table here>]
```

例子：定义一个学生-课程模式 “S-T”，并把 WANG 设置为管理员

```sql
CREATE SCHEMA "S-T" AUTHORIZATION WANG
```

说明：

- 如果 ModelName 没有指定的话，则默认与 UserName 同名
- 创建模式必须有 DataBase Administrator, DBA 权限，或者 DBA 授予的 CREATE SCHEMA 权限

<br />

在定义模式后面可以使用 [] 创建基于该模式的表

例子：为用户 WANG 创建一个模式 TEST，并在其中定义表 TAB1

```sql
CREATE SCHEMA "TEST" AUTHORIZATION WANG
CREATE TABLE TAB1 ( 
	COL1 SMALLINT,
	COL2 INT,
	...)

```

<br />

二、删除模式

```sql
DROP SCHEMA <ModelName> <CASCADE | RESTRICT>
```

说明：

- **CASCADE | RESTRICT 必须二选一**：CASCADE 表示级联（删除模式的同时一并把该模式所有的数据库对象删除）；REASTICT 表示限制（如果删除的模式定义了下属的数据库对象，则拒绝执行删除）

### 操作表

#### 定义基本表

```sql
CREATE TABLE <TableName> (
	<ColName> <DataType> [<Column-level integrity constraints>]
	[, <ColName> <DataType> [<Column-level integrity constraints>]]...
	[, <Table-level integrity constraints>]
)
```

说明：

- <xxx> 表示变量，实际写的时候不需要写 <>， 比如 table1
-  []中的部分表示可选项
- 完整性约束如果涉及到多个属性，则应该添加到表级的完整性约束；否则可以自由选择列或表级

例子：

```sql
CREATE TABLE Student (
	Sno CHAR(9) PRIMARY KEY,
	Sname CHAR(20) UNIQUE,
	Scno SMALLINT,
	Ssex CHAR(2),
	Sage SMALLINT,
	Sdept CHAR(20),
	FOREIGN KEY (Scno) REFERENCES Course(Cno),
	// 如果有多个主码：
	// PRIMARY KEY (Sno, Sname)
)
```

说明：

- PRIMARY KEY 表示主码
- UNIQUE 表示唯一值
- FOREIGN KEY 表示外键，REFERENCES 表示参照的外部表

#### 数据类型表

| 数据类型 | 含义 |
| --- | --- |
| CHAR(n) | 长度为 n 的字符串 |
| VARCHAR(n) | 最大长度为 n 的字符串 |
| INT | 长整数（也可以写成 INTEGER） |
| SMALLINT |  短整数 |
| NUMERIC(p, d) | 定点数，由 p 位数字组成（不包括符号，小数点），小数后面有 d 位数字 |
| REAL | 取决于机器精度的浮点数 |
| Double Precision | 取决于机器精度的双精度浮点数 |
| FLOAT(n) | 浮点数，精度至少 n 位数字 |
| DATE | 日期，包含年月日，格式为 YYYY-MM-DD |
| TIME | 时间，包含一日的时分秒，格式为 HH:MM:SS |

#### 建立模式与表之间的联系

每一个基本表都应该属于某一个模式，一个模式可以包含多个基本表

创建基本表时，如果没有指定模式，系统会根据**搜索路径**来确定对象所属的模式

```sql
// 显示当前搜索路径
SHOW search_path

// 输出结果
$<userName>, PUBLIC

// 手动设置搜索路径
SET search_path TO <"pathName">, PUBLIC
```

<br />

也可以在创建表时给出模式名

```sql
CREATE TABLE <"modelName">.<tableName>(...)
```

也可以在创建模式时一起创建表（见操作模式中，“定义模式” 部分）

#### 修改基本表

```sql
ALTER TABLE <tableName>
// add new column
[ADD[COLUMN] <newCalName> <dataType> [<Column-level integrity constraints>]]
[ADD <Table-level integrity constraints>]
// delete column
[DROP [COLUMN ] <ColumnName> [CASCADE|RESTRICT]]
[DROP CONSTRAINT <Column-level integrity constraints> [CASCADE|RESTRICT]]
// update column
[ALTER COLUMN <ColumnName> <DataType>]
```

#### 删除基本表

```sql
DROP TABLE <TableName> [RESTRICT|CASCADE]
```

### 操作索引

- 索引：类似于书的目录，用于加快查询速度
- 创建索引的权限：数据库管理员或表的拥有者
- 默认索引：DSMS 一般会自动建立 PRIMARY KEY 与 UNIQUE 的索引
- 维护索引：DBMS 自动完成
- 使用索引：DBMS 会自动选择是否使用索引，以及使用哪些索引
- 索引原理：索引一般使用 B+ 树（动态平衡）或 HASH（查找速度快）索引
- 索引类别：唯一索引（索引的值是唯一的）、非唯一索引（索引的值不是唯一的）、聚簇索引（索引顺序与表中记录的物理顺序一直的索引组织）
- 聚簇索引：在最经常查询的列中建立聚簇索引可以提高查询效率；一个表最多只建立一个、经常更新的列不宜建立（因为会频繁变更物理地址）

```sql
// CLUSTER 表示聚簇索引
CREATE [UNIQUE] [CLUSTER] INDEX <IndexName> ON <TableName> (
	<ColName> [<ASC | DESC>][, <ColName>[<ASC | DESC>]]...
)
```

例子：给 Student 表按照学号升序建立唯一索引

```sql
CREATE UNIQUE INDEX Stusno ON Student(Sno ASC)
```

## 数据字典

数据字典是关系数据库管理系统内部的一组系统表

数据字典记录数据库所有定义信息，包括：模式定义、视图定义、完整性约束定义、各类用户对数据库的操作权限、统计信息等

RDBMS 执行 SQL 数据定义时，实际上就是更新数据字典

## 数据查询（重点）

```sql
SELECT [ALL|DISTINCT] <TargrtExp> [, <TargetExp>]...
FROM <TableName | ViewName> [, <TableName | ViewName>]...
	[WHERE <ConditionExp>]
	[GROUP BY <ColName> [HAVING <ConditionExp>]
	[ORDER BY <ColNAme> [ASC|DESC]
```

### 单表查询

#### 查询某几列

例子：查询全体学生的学号与姓名

```sql
SELECT Sno, Sname FROM Student
```

#### 查询全部列

例子：查询所有学生的信息

```sql
// * 代表全部
SELECT * FROM Student
```

#### 查询经过计算的值

SELECT 中的 TargetExp 可以是：

- 算术表达式
- 字符串常量
- 函数
- 列别名

算术表达式例子：查询所有学生的出生年份

```sql
SELECT 2023-Sage FROM Student
```

函数例子：查询全体学生科系，使用小写

```sql
SELECT LOWER(Sdept) FROM Student
```

列别名例子：查询所有学生的姓名与出生年份，出生年份使用 Birthday 为列名展示

```sql
SELECT 2023-Sage Birthday, Sname FROM Student
```

### 筛选表中元组（行）

#### 消除重复的行

SQL 使用 DISTINCT 消除重复的行，若缺省则默认为 ALL

例子：查询学生选修了的课程号

```sql
SELECT DISTINCT Sno FROM SC
```

#### 查询满足条件的行

查询满足条件的行可以通过 WHERE 语句实现，常用的查询条件如下

| 查询条件 | 谓词 |
| --- | --- |
| 比较 | =, >, <, ≥, ≤, ≠, <>, !>, !<, NOT + 上述比较运算符 |
| 确定范围 | (NOT) BETWEEN … AND … |
| 确定集合 | (NOT) IN |
| 字符匹配 | (NOT) LIKE |
| 空值 | IS (NOT) NULL |
| 多重条件（逻辑运算） | AND, OR, NOT |

**使用逻辑运算的时候，可以联结多个查询结果，但是 AND 的优先级高于 OR（可以使用括号改变优先级）**

例子：查询计算机科学系全体学生的名字

```sql
SELECT Sname FROM Student WHERE Sdept = 'CS'
```

<br />

例子：查询年龄在 20～30 之间的学生的姓名

```sql
SELECT Sname FROM Student WHERE Sage BETWEEN 20 AND 30
```

<br />

例子：查询不是信息系、数学系、计算机系学生的姓名

```sql
SELECT Sname FROM Student WHERE Sdept NOT IN ('IS', 'MA', 'CS')
```

<br />

例子：查询所有姓王的学生的详细情况

```sql
// % 表示任意长度字符串
SELECT * FROM Student WHERE Sno LIKE '王%'

// _ 代表任意一个字符，如果需要查询两个字的名字
SELECT * FROM Student WHERE Sno LIKE '王_'

// 如果需要添加转义符号，比如想要使用 _
// 表示要查找叫 王_ 开头的学生姓名，ESCAPE 后的符号表示换码字符，跟在这个字符后面的字符表示原来的意思
SELECT * FROM Student WHERE Sno LIKE '王\_%' ESCAPE '\'
```

<br />

例子：查询计算机系年龄在 20 以下的学生的姓名

```sql
SELECT Sname FROM Student WHERE Sdept = 'CS' AND Sage < 20
```

#### 使用 ORDER BY 排序

```sql
SELECT xxx FROM xxx WHERE xxx ORDER BY xxx ASC|DESC
```

ASC 代表升序；DESC 代表降序；默认按照升序排序

当排序列还有空值时，**空值默认为最大值**

例子：查询选修了 3 号课程的学生学号，查询结果按照分数降序排列

```sql
SELECT Sno FROM SC WHERE Cno='3' ORDER BY Grade DESC
```

<br />

例子：查询全体学生情况，按照所在系号升序，同一系中学生按照年龄降序

```sql
// 不同的条件按照优先级从左到右，以逗号隔开，ASC 可以省略
SELECT * FROM Student ORDER BY Sdept, Sage DESC
```

#### 聚集函数

聚集函数可以选择去重（DISTINCT）或不去重（ALL）

| COUNT([DISTINCT | ALL] * ) | 统计元组个数 |
| --- | --- |
| COUNT([DISTINCT | ALL] <ColName> ) | 统计一列中值的个数 |
| SUM(DISTINCT | ALL] <ColName>) | 计算一列值的总和（必须是数值型） |
| AVG([DISTINCT | ALL] <ColName>) | 计算一列值的平均（必须是数值型） |
| MAX([DISTINCT | ALL] <ColName>) | 求一列值的最大值 |
| MIN([DISTINCT | ALL] <ColName>) | 求一列值的最小值 |

例子：查询学生总人数

```sql
SELECT COUNT(*) FROM Student
```

<br />

例子：求各个课程号以及相应的选课人数

```sql
// 先从 SC 表中根据 Cno 分组，然后计算每一组中的选课数量（Sno）
SELECT Cno, COUNT(Sno) FROM SC GROUP BY Cno
```

<br />

**使用 HAVING 短语指定筛选条件**

例子：查询选修了三门以上课程的学生学号

```sql
// 使用 HAVING 可以对分组之后的数据筛选
SELECT Sno FROM SC GROUP BY Sno HAVING COUNT(*) > 3
```

<br />

**HAVING 与 WHERE 的区别**：

- WHERE 作用于表或视图；HAVING 作用于组
- WHERE 子句中不能使用聚集函数

### 连接查询（多表查询）

多表查询的前提是需要先使用**等值连接**或**自然连接**将两张以上的表连接成一张大表（注意这张大表不会真实存在于物理地址中）

连接查询一般使用 WHERE 子句来连接两个表的条件，一般格式为：

```sql
// Compare Operator: = < > <= >= !=
[<Table1>.]<Col1> <Compare Operator> [<Table2>. ]<Col2>

// or...
[<Table1>.]<Col1> BETWEEN [<Table2>. ]<Col2> AND [<Table3>. ]<Col3>
```

注意事项：

- 当连接运算符为 “=” 时，表示等值连接；否则称为非等值连接
- 连接谓词中的列名称为连接字段，各个连接字段必须是可以相互比较的，名字不必相同

#### 等值连接

例子：查询每个学生及其选修课程的情况

```sql
SELECT Student.*, SC.* FROM Student, SC WHERE Student.Sno=SC.Sno
```

#### 自然连接

例子：使用自然连接查询每个学生及其选修课程的情况

```sql
// 自然连接需要写出所有要显示的列
// 表名可以省略
SELECT Student.Sno, Sname, Ssex, Sage, Sdept, Cno, Grade
FROM Student, SC 
WHERE Student.Sno=SC.Sno
```

#### 外连接

外连接指的是以特定表为连接主体，将主体表中不满足连接条件的元组一并输出

普通连接则是只输出满足条件的元组

外连接分为左外连接与右外连接：

```sql
// 左外连接用法
LEFT OUT JOIN <TableName> ON <Condition>

// 右外连接用法
RIGHT OUT JOIN <TableName> ON <Condition>
```

例子：使用左外连接查询每个学生与其选修课程的情况

```sql
SELECT Student.Sno, Sname, Ssex, Sage, Sdept, Cno, Grade
FROM Student
LEFT OUT JOIN SC ON Student.Sno=SC.Sno
```

#### 三个表的连接

例子：查询每个学生的学号、姓名、选修的课程名与成绩

```sql
SELECT Student.Sno, Sname, Cname, Grade
FROM Student, SC, Course
WHERE Student.Sno = SC.Sno AND Sc.Cno = Cource.Cno
```

### 嵌套查询

我们把一个 SELECT-FROM-WHERE 语句称为一个查询块

嵌套查询表示将一个查询块嵌套在另一个查询块的 WHERE 子句或 HAVING 短语的条件中查询

说明：

- 外层查询被称为父查询；内层查询被称为子查询
- 子查询中不能使用 ORDER BY 子句
- 有些嵌套查询可以用连接运算替代

#### 带有 IN 谓词的子查询

例子：查询与 “Liu” 在同一个系学习的学生

```sql
// 老方法需要分成两步查询：
// 1. 查询所在系
SELECT Sdept FROM Student WHERE Sname="Liu"
// 2. 查询所在系的所有学生
SELECT * FROM Student WHERE Sdept='xx'

// 使用子查询可以用一条语句搞定
SELECT * FROM Student WHERE Sdept IN (SELECT Sdept FROM Student WHERE Sname="Liu")
```

#### 带有比较运算符的子查询

如果能确切知道哪层查询返回**单值**时，可以使用比较运算符

例子：找出每个学生超过他选修课程平均成绩的课程号

```sql
SELECT Sno, Cno
FROM SC x
WHERE GRADE >= 
	(SELECT AVG(Grade)
	FROM SC y
	WHERE x.Sno=y.Sno)
```

#### 带有 ANY（SOME），ALL 谓词的子查询

- ANY：表示任意一个值
- ALL：表示所有值

需要配合只用比较运算符：

| 符号 | 含义 |
| --- | --- |
| >ANY | 大于子查询结果中的某个值 |
| >ALL | 大于子查询结果中的所有值 |
| <ANY | 小于子查询结果中的某个值 |
| <ALL | 小于子查询结果中的所有值 |
| ≥ANY | 大于等于子查询结果中的某个值 |
| ≥ALL | 大于等于子查询结果中的所有值 |
| ≤ANY | 小于等于子查询结果中的某个值 |
| ≤All | 小于等于子查询结果中的所有值 |
| =ANY | 等于子查询结果中的某个值 |
| =ALL | 等于子查询结果中的所有值 |
| ≠ANY | <>ANY | 不等于子查询结果中的某个值 |
| ≠ALL | <>ANY | 不等于子查询结果中的所有值 |

例子：查询非计算机科学系中比计算机科学任意一个学生年龄小的学生姓名与年龄

```sql
SELECT Sname, Sage
FROM Student
WHERE Sage < ANY
	(SELECT Sage FROM Student
	WHERE Sdept='CS')
AND Sdept <> 'CS'
```

<br />

说明：使用聚集函数实现子查询会比用 ANY， ALL 效率更高，其转换关系如下：

|  | = | <>  \|  ≠ | < | ≤ | > | ≥ |
| --- | --- | --- | --- | --- | --- | --- |
| ANY | IN | - | <MAX | ≤MAX | >MIN | ≥MIN |
| ALL | - | NOT IN | <MIN | ≤MIN | >MAX | ≥MAX |

#### 带有 EXIST 谓词的子查询

EXIST 谓词代表存在量词，带有 EXIST 谓词的子查询只会返回 true 或 false

例子：查询所有选修了 1 号课程的学生姓名

```sql
SELECT Sname 
FROM Student
WHERE EXISTS
	(SELECT * FROM SC WHERE Sno=Student.Sno AND Cno='1')
```

#### NOT EXIST 谓词

NOT EXIST 谓词用来判断查询结果不为空的情况，如果：

- 内层查询结果非空，则外层 WHERE 子句返回 false
- 内层查询结果为空，则外层 WHERE 子句返回 true

例子：查询没有选修 1 号课程的学生姓名

```sql
SELECT Sname 
FROM Student
WHERE NOT EXISTS
	(SELECT * FROM SC WHERE Sno=Student.Sno AND Cno='1')
```

<br />

说明：带有 EXIST 或 NOT EXIST 谓词的子查询不能被其他形式子查询等价替换；但是所有带 IN谓词、比较运算符、ANY、ALL 谓词的子查询都可以用 EXIST 谓词等价替换

### 集合查询

常见的集合操作的种类：并、交、差

参加集合操作的查询结果列数必须相同，对应的数据类型也必须相同

#### 并操作 UNION

例子：查询计算机科学系的学生级年龄不大于 19 岁的学生

```sql
// 使用并操作
SELECT * FROM Student WHERE Sdept='CS'
UNION SELECT * FROM Student WHERE Sage<=19

// 使用原来的方法
SELECT DISTINCT * FROM Student WHERE Sdept='CS' OR Sage<=19
```

#### 交操作 INTERSECT

例子：查询计算机科学系的学生与年龄不大于 19 岁的学生的交集

```sql
// INTERSECT
SELECT * FROM Student WHERE Sdept='CS'
INTERSECT SELECT * FROM Student WHERE Sage<=19

// 原来的方法
SELECT * FROM Student WHERE Sdept='CS' AND Sage<=19
```

#### 差操作 EXCEPT

例子：查询计算机科学系的学生与年龄不大于 19 岁的学生的差

```sql
// EXCEPT
SELECT * FROM Student WHERE Sdept='CS'
EXCEPT SELECT * FROM Student WHERE Sage<=19

// 原来的方法
SELECT * FROM Student WHERE Sdept='CS' AND Sage>19
```

### 基于派生表的查询

FROM 子句中不仅能够查询真实的表，也能查询临时的派生表

<br />

例子：找出每个学生超过他自己选修课程平均成绩的课程号

```sql
SELECT Sno, Cno
FROM SC, 
// 这一行生成了一个派生表
	(SELECT Sno, AVG(Grade) FROM SC GROUP BY Sno) AS Avg_sc(avg_sno, avg_grade)
WHERE SC.Sno=Avg_sc.avg_sno AND SC.Grade>=Avg_sc.avg_grade
```

## 数据更新

数据更新包括：插入、修改、删除

### 插入数据

#### 插入元组

格式：

```sql
INSERT INTO<TableName> [(<Property1> [, <Property2>]...) 
	VALUES (<Const1> [, <Const2>]...)
```

说明：

- INTO 子句中属性列的顺序可以跟表中的顺序不一致，没有指定属性列的则默认插入全部
- VALUES 子句中提供的值必须与 INTO 子句匹配，值与属性列的值的类型要一致
- RDBMS 会自动在新插入记录中没有值的列默认赋空值

例子：将一个新的学生元组插入到 Student 表中

```sql
INSERT INTO Student (Sno, Sname, Ssex, Sdept, Sage) 
	VALUES ('123456', 'chen', 'male', 'IS', 18)
```

#### 插入子查询结果

格式：

```sql
INSERT INTO <TableName> [(<Property1> [, <Property2>>]...) <SubSearch>
```

说明：

- SELECT 子句目标列必须与 INTO 子句匹配，值的个数与类型都必须一致

例子：对每一个系，求学生的平均年龄，并把结果存入数据库

```sql
// create table
CREATE TABLE Dept_age (
	Sdept CHAR(15)
	Avg age SMALLINT
)

// insert data
INSERT INTO Dept-age (Sdept, Avg_age)
	SELECT Sdept, AVG(Sage) FROM Student GROUP BY Sdept
```

### 修改数据

格式：

```sql
// 修改指定表中满足 where 子句条件的元组
UPDATE <TabelName> SET <ColName>=<Exp> [, <ColName>=<Exp>]...[WHERE <Condition>]
```

说明：

- SET 子句用于指定修改方式、修改的列、修改后的取值
- WHERE 子句用于指定要想修改的元组，缺省表示修改所有元组
- 在执行修改语句时会检查修改操作是否会破坏表上已经定义的完整性规则

<br />

#### 修改某一个元组的值

例子：将学生 123456 的年龄改为 22 岁

```sql
UPDATE Student SET Sage=22 WHERE Sno='123456'
```

#### 修改多个元组的值

例子：将所有学生年龄增加一岁

```sql
UPDATE Student SET Sage=Sage+1
```

#### 带子查询的修改

例子：将计算机系的所有学生成绩置0

```sql
UPDATE SC SET Grade=0 WHERE Sno IN (SELECT Sno FROM Student WHERE Sdept='CS')
```

### 删除数据

格式：

```sql
// 删除置顶表中满足 where 子句条件的元组
DELETE FROM <TableName> [WHERE <Condition>]
```

说明：

- WHERE 子句用于指定要删除的元组，缺省表示删除表中所有元组，表的定义仍在

#### 删除某一个元组的值

例子：删除学号为 123456 的学生的记录

```sql
DELETE FROM Student WHERE Sno='123456'
```

#### 删除多个元组的值

例子：删除所有学生的选课记录

```sql
DELETE FROM SC
```

#### 带子查询的删除语句

例子：删除计算机系所有学生的选课记录

```sql
DELETE FROM SC WHERE Sno IN (SELECT Sno FROM Student WHERE Sdept='CS')
```

### 空值的处理

空值的存在代表取值有不确定性，所以需要特殊处理

SQL 语言中的元组取空值一般有几种情况：

- 属性有值，但是目前不知道具体值
- 属性不应该有值
- 由于某种原因不便于填写

#### 插入空值

```sql
// NULL 代表空值，也可以不写默认为空值
INSERT INTO SC (Snom Cno, Grade) VALUES ('12345', '1', NULL)
```

#### 空值的判断

空值的判读啊可以用 IS NULL 或 IS NOT NULL 来判断

例子：从 Student 表中找出漏填数据的学生信息

```sql
SELECT * FROM Student 
	WHERE Sname IS NULL OR SSex IS NULL OR Sage IS NULL OR Sdept IS NULL
```

#### 空值的约束条件

1. 属性定义中有 NOT NULL 约束的条件不能取空值
2. 加了 UNIQUE 限制的属性不能取空值
3. 码属性不能取空值

#### 空值的运算

- 算术运算：空值与另一个值（包括空值）的算术运算结果为空值
- 比较运损：空值与另一个值（包括空值）的比较运算结果为 UNKNOWN
- 逻辑运算：如下表，T 代表 true；F 代表 false；U 代表 unknown

| XY | X AND Y | X OR Y | NOT X |
| --- | --- | --- | --- |
| TT | T | T | F |
| TU | U | T | F |
| TF | F | T | F |
| UT | U | T | U |
| UU | U | U | U |
| UF | F | U | U |
| FT | F | T | T |
| FU | F | U | T |
| FF | F | F | T |

## 视图

### 视图的特点

- 视图是虚表，是从一个或多个基本表（或视图）导出的
- 导出的表只存放视图的定义，不存放视图对应的数据
- 基本表表中的数据发生变化，视图中查询出的数据也随之改变

### 视图的操作

视图的操作包含了查询、删除、受限更新、定义给予该视图的新视图

### 建立视图

格式：

```sql
CREATE VIEW <ViewName> [(<ColName> [, <ColName>]...)] 
	AS <SubQuery> [WITH CHECK OPTION]
```

说明：

- ColName 为组成视图的属性列名，可以全部缺省（表示全部来源于组成视图的表）或全部指定
- SubQuery 为子查询，不允许还有 ORDER BY 子句和 DISTANCT 短语
- DBMS 只会保存视图的定义，并不执行其中的 SELECT 语句
- 查询视图的时候是按照视图的定义从基本表中查询数据

#### 基于一个基表的视图

例子：建议信息系学生的视图

```sql
CREATE VIEW IS_Student AS
	SELECT Sno, Sname, Sage FROM Student WHERE Sdept='IS'
```

#### 基于多个基表的视图

例子：建立信息系选修了一号课程的学生视图

```sql
CREATE VIEW IS_S1(Sno, Sname, Grade) AS
	SELECT Student.Sno, Sname, Grage FROM Student, SC WHERE Sdept='IS' AND Student.Sno=SC.Sno AND SC.Cno='1'
```

#### 基于视图的视图

例子：建立信息系选修了一号课程且成绩在 90 分以上的学生视图

```sql
CREATE VIEW IS_S2 AS
	SELECT Sno, Sname, Grade FROM IS_S1 WHERE Grade >= 90
```

#### 给视图分组

例子：将学生的学号以及平均成绩定义为一个视图

```sql
CREATE VIEW S_G (Sno, Gavg) AS SELECT Sno, AVG(Grade) FROM SC Group BY Sno
```

#### 不指定属性的列

例子：将 Student 表中所有女生记录定义为一个视图

```sql
// 使用 * 表示将查询的结果以此放到上述视图列出的属性中
CREATE VIEW F_Student(F_Sno, name, sex, age, dept) AS
	SELECT * FROM Student WHERE Ssex='female'
```

注意：这种方法会导致 Student 表与 F_Student 视图印象关系被破坏，导致视图不能正常工作

### 删除视图

格式：

```sql
DROP VIEW <ViewName> [CASCADE]
```

说明：

- 执行语句会从数据字典中删除指定**视图的定义**
- 如果从视图上还导出了其他视图，必须使用 CASCADE 所有视图一起删除
- 删除基本表的时候，由该基本表所导出的所有视图都必须显式删除

#### 删除普通视图

```sql
DROP VIEW IS_Student
```

#### 删除有导出其他视图的视图

```sql
// 如果不使用 CASCADE 会报错，因为 IS_S2 是根据这个视图建立的
DROP VIEW IS_S1 CASCADE
```

### 查询视图

视图在定义后，用户可以像车讯基本表一样对视图进行查询

RDBMS 中实现视图查询的方法被称之为**视图消解法**：

1. 进行有效性检车
2. 转换成等价的对基本表的查询
3. 执行修正后的查询

例子：在信息系学生视图 IS_Student 中找出奶年龄小于 20 的学生

```sql
// 视图查询
SELECT Sno, Sage FROM IS_Student WHERE Sage < 20

// 基本表查询
SELECT Sno, Sage FROM Student WHERE Sage < 20 AND Sdept='IS'
```

<br />

视图消解法也有其局限性

例子：在 S_G 视图中查询平均成绩在 90 分以上的学生学号和平均成绩

```sql
// 视图查询
SELECT * FROM S_G WHERE Gavg >= 90

// 【错误转换】视图消解法之后对基本表的查询
SELECT Sno, AVG(Grade) FROM SC WHERE AVG(Grade) >= 90 GROUP BY Sno

// 【正确转换】
SELECT Sno, AVG(Grade) FROM SC GROUP BY Sno HAVING AVG(Grade) >= 90
```

### 更新视图

更新视图指的是通过视图来插入、删除、修改数据，因为视图不会存储数据，因此对视图的更新会通过视图消解来转换成对实际表中的更新操作

为了防止在视图更新时出错，在**定义视图时一定要加上 WITH CHECK OPTION 子句**

例子：将信息系学生视图 IS_Student 中学号 200215122 的学生姓名改为 liu

```sql
// 视图更新语句
UPDATE IS_Student SET Sname='liu' WHERE Sno='200215122'

// 转换后的语句
UPDATE Student SET Sname='liu' WHERE Sno='200215122' AND Sdept='IS'
```

#### 更新视图的限制

有一些视图时无法更新的，因为这些视图的更新不能唯一的有意义的转换成对应基本表的更新

例子：将视图 S_G 中学号为 123456 的学生平均成绩改为 90 分

```sql
// 1. 建立视图
CREATE VIEW S_G(Sno, Gavg) AS SELECT Sno, AVG(Grade) FROM SC GROUP BY Sno

// 2. 更新视图
// 这种更新是没有意义的，因为平均成绩是经过计算出来的
UPDATE S_G SET Gavg=90 WHERE Sno='123456'
```

#### 视图的作用

- 视图能简化用户的操作
- 视图使用户能以多种角度看待同一数据
- 视图对重构数据库提供了一定程度的逻辑独立性
- 视图能狗对机密数据提供安全保护
- 适当利用视图可以更清晰表达查询