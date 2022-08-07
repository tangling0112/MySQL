#  第一部分

## `MySQL`安装好后在`Windows`下的存在情况与特点

- `Data`存储文件夹

> 其中存储了``MySQL``中我们创建的的数据库，表，配置文件等。当我们删除`MySQL`后,这个文件并不会被删除.

- `Application`存储文件夹

> 其中存储着我们的`MySQL`的应用

- 自启动服务

> 存在于我们的任务管理器的服务选项中.

- `Path`环境变量

> 存在于`系统-高级设置-环境变量-系统变量-Path`中,值得注意的是,我们的`MySQL`安装器不会自动配置环境变量,而是我们需要自己将`MySQL`的`bin`目录添加为系统变量

## 多版本`MySQL`共存

> 多个不同版本的`MySQL`是可以同时存在于我们的系统中的,当然我们应该注意不同版本的`MySQL`应该使用不同的端口号(这里我们的`MySQL 8.0 = 3306,MySQL 5.7 = 3307`)

### 不同版本`MySQL`登录

- 首先我们在命令行中登录`MySQL`是需要使用到`MySQL`的登陆代码的,因此我们在环境变量中是`MySQL 8.0`的`bin`目录在前还是`MySQL 5.7`的`bin`目录在前是有不同的.如果是前者,那么使用不带端口号的登陆就会登入`MySQL 8.0`,相反后者则会进入`MySQL 5.7`.
- 若我们的`MySQL 8.0`与`MySQL 5.7`的端口号分别为`3306/3307`,那么就可以通过指定端口号来进行不同版本`MySQL`服务的登陆
  - `mysql -u root -p -P 3306`:非明文密码登陆`MySQL 8.0`
  - `mysql -u root -p -P 3307`:非明文密码登陆`MySQL 8.0`
  - `mysql -u root -p123456 -P 3306`:明文密码登陆`MySQL 8.0`
  - `mysql -u root -p123456 -P 3307`:明文密码登陆`MySQL 8.0`

> **问题**:不同版本`MySQL`的密码不同的情况下,使用`8.0`版本的登陆代码,输入`5.7`版本的登陆密码与用户名,是否可以正确登陆`5.7`的`MySQL`服务?

## `MySQL`服务的启动与停止

> 服务的作用是用于连接我们的`MySQL`客户端,如果我们将服务关闭,那么我们便无法在命令行中连接到我们的`MySQL`
>
> 前提:以管理员权限打开命令行

- `net stop mysql2022_6_29`
- `net start mysql2022_6_29`

## `MySQL`的登陆与退出

### 登陆

> 命令:`mysql -u <用户名> -p<密码> -h <需要连接的MySQL服务器的IP> -P <MySQL服务进程的端口号>`
>
> 注意:`-p`之后如果要输入密码则不应该接空格

- `-h`:默认情况下为`localhost`,即我们当前主机.当我们需要远程连接其他主机上的`MySQL`时就需要指定这一属性为目标主机的`IP`
- `-P`:默认情况下为``3306``计算机的每一个进程都具备一个唯一的端口号,我们需要通过指定端口号来连接到目标主机指定的`MySQL`服务进程上.比如目标主机上有多个不同版本的`MySQL`,分别占据不同的端口,那么我们就可以通过这个属性来指定连接到哪个版本的`MySQL`服务上
- 实例
  - `mysql -u root -p<密码> -P 3306`:连接到`MySQL 8.0`
  - `mysql -u root -p<密码> -P 3307`:连接到`MySQL 5.7`

### 退出

> `quit/exit`

## `MySQL`数据库中文存储问题的解决(`MySQL 5.7`)

> 问题原因:`MySQL`的数据表的默认字符集为`latin`字符集,而这个字符集是无法表示中文字符的,因此当我们在创建数据表时没有显式地指明其要使用的字符集,那么我们的数据表就会默认使用`Latin`字符集

### 当前默认字符集查询

- `show variables like 'character_%'`
- `show variables like 'collation_%'`

### 默认字符集修改

- 我们需要在`MySQL`的`Data File`的`mysql.ini`文件中找到`default-character-set=`选项,将其修改成我们需要的默认字符集即可(如`utf8,utf16,gbk,gbk2312`等)
- <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-1?token=AOAPFCNEXDN57SY5YO7SE4LC57WDE" alt="image-20220629230227930" style="zoom:80%;" />

### 显式地指明数据库/数据表使用的字符集

#### 创建

- `create databese <数据库名> charset '<Latin1|utf8|gbk2312...>'`
- `create table <表名>(...) charset '<Latin1|utf8|gbk2312...>'`

#### 修改

- `alter databese <数据库名> charset '<Latin1|utf8|gbk2312...>'`
- `alter table <表名> charset '<Latin1|utf8|gbk2312...>' `

> **注意**:
>
> - 我们修改了默认字符集后,`MySQL`并不会自动改变我们在修改之前创建的数据表使用的默认字符集,因此我们应该在创建数据表之前养成判断其需要使用的字符集的习惯.
> - 在修改好默认字符集后,务必要通过`net stop mysql2022_6_29和net start mysql2022_6_29`来重启服务
>
> - 在`MySQL 8.0`中默认使用的为`utf8`

## `MySQL`图形化管理工具

> 图形化管理工具能够让我们通过图形界面来操作我们的`MySQL`服务

### `MySQL Workbench`

### `Navicat Preminum`

> [点击下载](https://www.52pojie.cn/thread-1117892-1-1.html)

### `SQLyog`

> [点击下载](https://www.52pojie.cn/thread-1066141-1-1.html)

### `dbeaver`

## `MySQL`默认的自带数据库介绍

### `information_schema`

> 保存数据库服务系统的基本信息(包括但不限于)
>
> - 数据库名称
> - 表名称
> - 字段名称
> - 存储权限

### `mysql`

> 保存数据库运行时所需的系统信息
>
> - 数据库文件夹
> - 当前系统字符集

### `performance_schema`

> 存储``MySQL``服务的各个系统性能指标

### `sys`

> 存储`MySQL`服务的各个系统性能指标

# 第二部分

## `SQL`基本介绍

> 美国国家标准局(`ANSI`)制定了统一的`SQL`语言标准,但是不同的`DBMS`大多都具备不同的`SQL`,并没有严格地遵守`SQL`语言的标准.

### `SQL`语言的分类

- `DDL`:**数据定义语言**
  - `CREATE`:创建
  - ``ALTER``:修改
  - ``DROP``:丢弃
  - ``RENAME``:重命名
  - ``TRUNCATE``:清空
- `DML`:**数据操作语言**
  - `INSERT`:插入
  - `DELETE`:删除
  - `UPDATE`:更新
  - `SELECT`:查找
- `DCL`:**数据控制语言**
  - `COMMIT`:提交
  - `ROLLBACK`:回滚
  - `SAVEPOINT`:保存点
  - `GRANT`:授权
  - `REVOKE`:收回权限

### `SQL`语言的基本规则规范

- `SQL`语言借助分号来表示一个语句的结束,因此我们可以将一个语句任意拆分为多行,只要保证在结束的时候添加分号,就能保证`SQL`解析器能够正确解析我们的`SQL`语句

  ```SQL
  SELECT * FROM student;
  等价于
  SELECT *
  FROM student;
  ```

- `SQL`语言在`Windows`环境下是**大小写不敏感**的,但是在`Linux`环境下是**大小写敏感**的

- 推荐将`数据库名`,`表名`,`表别名`,`字段名`,`字段别名`都使用**小写**,`关键字`,`函数名`,`绑定变量`都使用**大写**

- `SQL`语言的单行注释使用`#或-- `,多行注释使用`/**/`

- 我们在创建``数据库``,``数据表``,``字段``等自命名成员时,应该保证我们的命名不与数据库系统的关键字相互冲突.**当已经相互冲突时我们可以通过使用着重号`<字符串>`的方式来告诉系统这不是一个关键字**

- `SQL`语言的不区分大小写指的是我们的`SQL`**指令不区分**大小写,而**数据表中的记录的属性的值是区分大小写的**.

## `MySQL`基础命令

- `show databases`:呈现所有现存的数据库
- `create databese <数据库名>`:创建数据库
- `delete database <数据库名>`:删除数据库
- `use <数据库名>`:使用指定数据库
- `show tables`:呈现当前数据库内的所有数据表
- `SOURCE <SQL文件>`:执行我们指定的`SQL`脚本
- `DESCRIBE <Table_Name> 或 DESC <Table_Name>` :显示指定表中的字段的**详细信息**(如`数据类型`,`是否可为空`,`是否主键`等)

## `MySQL`中的单引号与双引号

### 单引号

- 字符串
- 日期

### 双引号

- 别名

## ==`MySQL`的显式转换与隐式转换==

> **隐式转换**
>
> - **整数与整数**做``加减乘以及比较``,**不会**发生类型转换
> - **整数与整数**``相除以及比较``**会**被转换为浮点数相除或比较
> - **整数与浮点数**做``加减乘除以及比较``**会**被转换为浮点数的加减乘除以及比较
> - **整数与字符,字符串**的``加减乘以及比较``**会**被转换为整数与整数的加减乘以及比较
> - **整数与字符,字符串**``相除以及比较``**会**被转换为浮点数相除以及比较
> - **浮点数与浮点数**的``加减乘除以及比较``**不会**发生类型转换
> - **浮点数与字符,字符串**的``加减乘除以及比较``**会**被转换为浮点数与浮点数的加减乘除以及比较
> - **字符,字符串与字符,字符串**做``加减乘除以及比较``**不会**发生类型转换
> - 任何数据类型与``NULL``做``加减乘除以及比较(除安全等于以外)``的结果都必定为`NULL`

- `SELECT 1 + '1'`:结果为2
- `SELECT 1 + 'a'`:结果为1
- `SELECT 1 + 'ab'`:结果为1
- `SELECT 1 + NULL`:结果为``NULL``

## `MySQL`查询时给数据表起别名

- `SELECT * FROM Student as S`
- `SELECT * FROM Student S`

##  ==`MySQL`的查询初级==

### 基础查询

- `SELECT * FROM <Table_Name>`
- ``SELECT 字段1,字段2… FROM <Table_Name>``

****

### 常量查询

- `SELECT 12 FROM <Table_Name>`
- `SELECT '汤凌' FROM <Table_Name>`
- `SELECT '汤凌',字段1 FROM <Table_Name>`

****

### 别名查询(`AS`)

> - **别名查询**时会按照字段原名查询我们指定的字段,但是在呈现结果时,设置了别名的字段就会以别名呈现出来,而非原名
>
> - **双引号**:我们可以使用`""`将别名包裹起来,其作用为使得`"别 名"`可以实现,即使得别名中可以带有空格.当然如果我们的别名中不带空格,那么可以不用添加`""`

- `SELECT 字段1 as <别名> 字段2,字段3 as <别名> FROM <Table_Name>`
- `SELECT 字段1 <别名> 字段2,字段3 <别名> FROM <Table_Name>`
- `SELECT 字段1 "<别名>" 字段2,字段3 <别名> FROM <Table_Name>`

****

### 去重查询(``DISTINCT``)

> 去重查询会将我们查询得到的记录中,对于我们指定的字段相同的记录,我们的查询语句最终只会呈现其中一个

- `SELECT DISTINCT 字段1 FROM <Table_Name>`:根据**字段1**进行去重
- `SELECT DISTINCT 字段1,字段2 FROM <Table_Name>`:根据**字段1**与**字段2**进行去重
- `SELECT 字段1,DISTINCT 字段2 FROM <Table_Name>`:根据**字段2**进行去重,这样的去重查询**一般都会报错**

****

### `MySQL`运算符

> **注意**
>
> - 用于`运算符`的字段并不一定是我们查询的字段之一,也可以是数据表中其他任意未被查询的字段

####  算术运算符

| **+** | 加         |
| ----- | ---------- |
| **-** | **减**     |
| ***** | **乘**     |
| **/** | **除**     |
| **%** | **取余数** |

#### 数学比较运算符

> `MySQL`中非零表示真,0表示假

| **>**      | **大于**                                                     |
| ---------- | ------------------------------------------------------------ |
| **<**      | **小于**                                                     |
| =****      | **等于,注意`1=NULL`的结果为``NULL``,`NULL=NULL`的结果为``NULL``** |
| **<=>**    | **安全等于,其与普通等于的区别在于,``2<=>NULL``的结果为0,`NULL<=>NULL`结果为1** |
| **!=或<>** | **不等于**                                                   |
| **<=**     | **小于等于**                                                 |
| **>=**     | **大于等于**                                                 |

#### 特殊比较运算符

| 运算符                   | 名称       | 作用                                 | 使用实例                                                  |
| ------------------------ | ---------- | ------------------------------------ | --------------------------------------------------------- |
| **IS NULL**              | 为空       | 判断值,字符串,表达式的结果是否为NULL | `SELECT * FROM students WHERE age IS NULL`                |
| **IS NOT NULL**          | 不为空     | 判断值,字符串,表达式结果是否不为NULL | `SELECT * FROM students WHERE age IS NOT NULL`            |
| **ISNULL**               | 为空       | 判断值,字符串,表达式结果是否为NULL   | `SELECT * FROM students WHERE ISNULL(age)`                |
| **LEAST**                | 最小值     | 返回多个值中的最小值                 | ``SELECT LEAST(字段1,字段2) FROM students``               |
| **GREATEST**             | 最大值     | 返回多个值中的最大值                 | ``SELECT GREATEST(字段1,字段2) FROM students``            |
| **BETWEEN  A AND B**     | 两值之间   | 判断一个值是否在[A,B]区间内          | `SELECT * FROM students WHERE age BETWEEN 25 AND 30`      |
| **NOT BETWEEN  A AND B** | 非两值之间 | 判断一个值是否不在[A,B]区间内        | `SELECT * FROM students WHERE age NOT BETWEEN 25 AND 30`  |
| **IN**                   | 属于       | 判断一个值是否属于给出的值中的一个   | `SELECT * FROM students WHERE age IN(20,25,23,36)`        |
| **NOT IN**               | 不属于     | 判断一个值是否不属于给出的值中的一个 | `SELECT * FROM students WHERE age NOT IN(20,25,23,36)`    |
| **LIKE**                 | 模糊匹配   | 判断一个值是否符合模糊匹配规则       | `SELECT * FROM students WHERE age LIKE('<模糊匹配式>')`   |
| **REGEXP**               | 正则匹配   | 判断一个值是否符合正则匹配规则       | `SELECT * FROM students WHERE age REGEXP('<正则匹配式>')` |
| **RLIKE**                | 正则匹配   |                                      | `SELECT * FROM students WHERE age RLIKE('<正则匹配式>')`  |

#### 逻辑运算符

> 逻辑运算符优先级排序为`NOT>AND>OR=XOR`

| **运算符**     | **作用**     |
| -------------- | ------------ |
| **NOT 或 !**   | **逻辑非**   |
| **AND 或 &&**  | **逻辑与**   |
| **OR 或 \|\|** | **逻辑或**   |
| **XOR**        | **逻辑异或** |

#### 位运算符

> 位运算是基于我们**给定的值的二进制数**来进行运算

| **运算符** | **作用**     | **实例**           |
| ---------- | ------------ | ------------------ |
| **&**      | **按位与**   | **`SELECT A & B`** |
| **\|**     | **按位或**   | **`SELECT A | B`** |
| **^**      | **按位异或** | **`SELECT A ^ B`** |
| **~**      | **按位取反** | **`SELECT ~A`**    |
| **>>**     | **按位右移** | **`SELECT A>>2`**  |
| **<<**     | **按位左移** | **`SELECT A<<2`**  |

> **注意**
>
> - **:red_circle: 空值`NULL`在`SQL`中并不等于0,当`NULL`参与运算时无论参与的是``加减乘除``中的任何一个运算,都会导致最后的结果为`NULL`,``NULL``参与比较运算的结果也为`NULL`**

****

### **条件查询(`WHERE`)**

> **条件查询通过借助比较运算符，逻辑运算符以及`WHERE`关键字的组合来实现数据记录的过滤**
>
> **注意**
>
> - 用于`WHERE`的字段并不一定是我们查询的字段之一,也可以是数据表中其他任意未被查询的字段

- **`SELECT 字段1,字段2 FROM <Table_Name> WHERE 字段1 = '汤凌'`**
- **`SELECT 字段1,字段2 FROM <Table_Name> WHERE 字段2 > 50 `**

****

### **模糊查询(`WHERE+LIKE`)**

#### **模糊查询运算符**

> **注意:模糊查询以包含模糊查询表达式的左边的单引号为开始标志,右边的单引号为结束标志**

| **符号** | **作用**                                            |
| -------- | --------------------------------------------------- |
| **%**    | **匹配不确定个数的任意字符**                        |
| **_**    | **匹配一个任意字符**                                |
| **\或$** | **转义字符,用于将用于匹配的专用字符转换为普通字符** |

****

### **排序查询(`ORDER BY`)**

> **注意**
>
> - 我们的字段的别名**只可以**在`ORDER BY`子句中使用,而`WHERE`等其他任何关键字的子句中是**无法使用**字段别名的.
> - 用于`ORDER BY`的字段并不一定是我们查询的字段之一,也可以是数据表中其他任意未被查询的字段
> - 当语句中使用到`WHERE`时`ORDER BY`必须使用在`WHERE`关键字之后
> - 无论使用升序还是降序排列,`NULL`永远在前
>
> **原因**:其原因涉及到排序与条件过滤对于数据表中记录的操作.

#### `MySQL`默认排序规则

- 当我们在查询时未指定查询结果的排序规则时,`MySQL`使用其默认的排序规则,**即按照我们在向数据表中插入数据时的各个记录插入的先后顺序来进行排序.先插入的记录排在前,后插入的记录排在后**

#### `MySQL`排序查询实例

##### 基础排序

- `SELECT 字段1,字段2,字段3 FROM <Table_Name> ORDER BY 字段3 `
  - 默认为``ASC``
- `SELECT 字段1,字段2,字段3 FROM <Table_Name> ORDER BY 字段3 DESC `
  - `DESC`:降序
- `SELECT 字段1,字段2,字段3 FROM <Table_Name> ORDER BY 字段3 ASC `
  - `ASC`:升序

##### 别名排序

- `SELECT 字段1,字段2,字段3 别名 FROM <Table_Name> ORDER BY 别名 `
- `SELECT 字段1,字段2,字段3*100 别名 FROM <Table_Name> ORDER BY 别名 `

##### 二级排序

> - 当某些记录的字段3取值相等时,那么这些记录就会按照字段2进行`ASC`排序
>
> - 参考二级排序,我们还可以设计**多级排序**

- `SELECT 字段1,字段2,字段3 FROM <Table_Name> ORDER BY 字段3 DESC,字段2 ASC `
- `SELECT 字段1,字段2,字段3 FROM <Table_Name> ORDER BY 字段3,字段2 ASC`

##### 表达式排序

- `SELECT 字段1,字段2,字段3 FROM <Table_Name> ORDER BY 字段3 * 3 `
- `SELECT 字段1,字段2,字段3 FROM <Table_Name> ORDER BY ANS(字段3) `
- `SELECT 字段1,字段2,字段3 FROM <Table_Name> ORDER BY 字段3 - 1，字段4 + 5 `

****

### **分页查询(`LIMIT`)**

> **注意**
>
> - 我们应该保证在查询语句中`LIMIT`关键字对应的子句位于`WHERE`,`ORDER BY`对应的子句之后(**实际上我们更进一步地,`LIMIT`关键字必须处于任意查询语句的最后**)
> - `LIMIT a,b`中``a``为起始索引,`b`为偏移量
> - **分页区间计算公式`区间=[a,a+b)`**
> - 在`MySQL`中第一条记录的索引为`0`
>
> **原因**:

#### 作用与应用背景

- **作用**
  - 分页即将我们的查询结果分成多个页,每个页由多个记录组成.我们每一次只能查看到某一页中的记录.而剩余的记录则需要涉及到对页的切换.

- **应用背景**
  - 当我们的查询结果由巨量的记录组成时,我们可以通过分页使得每一次只获取查询结果的指定条记录

#### 分页的实现

> **分页区间计算公式`区间=[a,a+b)`**

- `SELECT * FROM <Table_Name> LIMIT(0,20)`:返回$[0,20)$区间内的记录
- `SELECT * FROM <Table_Name> LIMIT(20,40)`:返回$[20,60)$区间内的记录
- `SELECT * FROM <Table_Name> LIMIT(0,10)`:返回$[0,10)$区间内的记录
- `SELECT * FROM <Table_Name> LIMIT(10,20)`:返回$[10,30)$区间内的记录

#### `MySQL 8.0`分页新特性(`LIMIT <a> OFFSET <b>`)

> **分页区间计算公式`区间=[b,b+a)`**
>
> - `b`为起始索引,`a`为偏移量.与纯`LIMIT`正好相反

- `SELECT * FROM <Table_Name> LIMIT 40 OFFSET 20`:返回$[20,60)$区间内的记录
- `SELECT * FROM <Table_Name> LIMIT 2 OFFSET 20`:返回$[20,22)$区间内的记录
- `SELECT * FROM <Table_Name> LIMIT 88 OFFSET 2`:返回$[2,90)$区间内的记录

#### 拓展(不同的`DBMS`如何实现分页查询)

- 在`MySQL`,`PostgreSQL`,`MariaDB`,`SQLite`中使用`LIMIT`关键字实现分页

- 在`SQL Server`与`Access`中使用`TOP`关键字

  ```SQL
  SELECT TOP 5 字段1,字段2 FROM <Table_Name> ORDER BY 字段3 DESC;
  ```

- 在`DB2`中使用`FETCH FIRST 5 ROWS ONLY`关键字

  ```SQL
  SELECT 字段1,字段2 FROM <Table_Name> ORDER BY 字段3 DESC FETCH FIRST 5 ROWS ONLY;
  ```

- 在`Oracle`中不具备分页关键字,我们只能通过`Oracle`给每个数据表自动维护的`rownum`字段来进行分页

  ```sql
  SELECT Rownum,字段1,字段2 FROM <Table_Name> WHERE Rownum<5 ORDER BY 字段3 DESC;
  ```

****

### **多表查询**(内连接,外连接,等值连接,非等值连接,自连接,非自连接)

> 多表查询即使用一个`SQL`语句查询多个不同的数据表中的数据,并在结果呈现时,将多个表的查询结果有机结合起来进行呈现.
>
> ==**当涉及到三个及以上的表的查询时最好不要使用`JOIN`**==
>
> **注意**
>
> - 不同与给**字段**起别名,在`SQL`语句中给**表**起了别名,那么其原名在改`SQL`语句中便不再可用
> - **当涉及到$$n$$个表的多表查询时,那么我们至少需要有$n-1$个连接条件**

#### 多表查询实例

- `SELECT 字段1,字段2 FROM <Table1_Name>,<Table2_Name>`:在字段1属于表1,字段2属于表2时	会返回字段1与字段2的笛卡尔积
- ``SELECT 字段1,字段2 FROM <Table1_Name> CROSS JOIN<Table2_Name>``:等价于上面的查询语句
- `SELECT 字段1,字段2 FROM <Table1_Name>,<Table2_Name> WHERE <Table1_Name>.字段3= <Table2_Name>.字段4 `
- `SELECT 字段1,字段2 FROM <Table1_Name>,<Table2_Name> WHERE <Table1_Name>.'字段3'= <Table2_Name>.'字段4' `:与上一个查询等价
  - 这里的`WHERE`是作为多表查询的表间连接条件出现的
- ``SELECT 字段1,字段2,<Table1_Name>.字段5 FROM <Table1_Name>,<Table2_Name> WHERE <Table1_Name>.'字段3'= <Table2_Name>.'字段4' ``
  - 当字段5在表1与表2中都出现时,我们就需要在查询时指明我们要查询的是哪个表中的字段5
  - **实际上我们推荐在使用到多表查询时,对于每一个待查询的字段都指定好其表的归属**
- ``SELECT <Table1_Name>.字段1,<Table2_Name>.字段2,<Table1_Name>.字段5 FROM <Table1_Name>,<Table2_Name> WHERE <Table1_Name>.'字段3'= <Table2_Name>.'字段4' ``
- ``SELECT T1.字段1,T2.字段2,T1.字段5 FROM <Table1_Name> T1,<Table2_Name> as T2 WHERE T1.'字段3'= T2.'字段4' ``:与上一个查询相同,只不过通过对表起别名的方法增加了`SQL`语句的可读性
- ``SELECT T1.字段1,T2.字段2,T1.字段5 FROM <Table1_Name> T1,<Table2_Name> as T2 WHERE <Table1_Name>.'字段3'= <Table2_Name>.'字段4' ``:与上一个查询相同,但这个语句是错误的,因为一旦在`SQL`语句中给表起了别名,那么其原名便不再可用

#### 多表查询的分类

- **等值连接与非等值连接**

  - **等值连接**
    - `SELECT s1.age,s2.address,s2.height FROM students1 s1,students2 s2 WHERE(s1.StudentID=s2.StudentID)  `
  - **非等值连接** 
    - `SELECT s1.name,s1.age,s1.height,s2.salary,s2.mother FROM students1 s1,students2 s2 WHERE(s1.age BETWEEN s2.least_age AND s2.greatest_age`

- **自连接与非自连接**

  > 即我们的表与表间联系的**自我引用**

  - **自连接**
    - `SELECT emp.employeeid,emp.last_name,mgr.employeeid,mgr.last_name FROM employees as emp,employees as mgr WHERE(emp.manager_id=mgr.employeeid)`
  - **非自连接**
    - `SELECT emp.employeeid,emp.last_name,mgr.employeeid,mgr.last_name FROM employees as emp,managers as mgr WHERE(emp.manager_id=mgr.employeeid)`

- **内连接与外连接**

  > **注意**
  >
  > - `MySQL`不支持`SQL92`标准,部分支持`SQL99`标准
  > - `Oracle`同时支持`SQL92`标准与`SQL99`标准
  > - `SQL92`与`SQL99`的内外连接我们可以按照相同的方式去理解

  - **内连接**

    - 一般用于合并具有**相同字段的多个表时使用**,其会将该相同字段上取值相同的各个表的记录拼接起来,然后根据我们的需要进行查询结果的返回.**那些该字段取值不相等的记录则会被不会被拼接,也就不会呈现在结果中**

    - `SQL92`

      - **单级内连接**
        - `SELECT s1.age,s1.name,s2.age,s2.name FROM students1 s1,students2 s2 WHERE(s1.height = s2.height)`
      - **多级内连接**
        - `SELECT s1.age,s1.name,s2.age,s2.name,s3.age,s3.name FROM students1 s1,students2 s2,students s3 WHERE(s1.height = s2.height AND s2.weight=s3.weight)`

    - `SQL99`

      - **单级`INNER JOIN`的使用**

        - `SELECT s1.age,s1.name,s2.age,s2.name FROM students1 s1 JOIN students2 s2 ON(s1.height = s2.height)`

          > **原理**就相当于先把`students2`中符合`ON`关键字指示的相等条件的记录合并到到`students1`的对应记录上,然后再对这个合并后的虚拟数据表进行查询

      - **多级`INNER JOIN`的使用**

        - `SELECT s1.age,s1.name,s2.age,s2.name,s3.age,s3.name FROM students1 s1 JOIN students2 s2 ON(s1.height = s2.height) JOIN students3 s3 ON(s2.weight=s3.weight)`

          > **原理**就相当于在得到第一个`JOIN`的虚拟表后,再将第三个表`JOIN`到我们的虚拟表中,形成一个新的虚拟表后再对这个新的虚拟表进行查询操作

  - **外连接**

    - **左外连接**$(A ∩ B)∪(A-B)$

      - 在**内连接**的基础上,还会返回**左表**中没有被呈现的记录的对应的被查询字段

      - **`SQL92`**

        - `SELECT s1.age,s1.name,s2.age,s2.name FROM students1 s1,students2 s2 WHERE(s1.height = s2.height(+))`

      - **`SQL99`**:**LEFT OUTER JOIN** 

        - `SELECT s1.age,s1.name,s2.age,s2.name FROM students1 s1 LEFT OUTER JOIN students2 s2 ON(s1.height = s2.height)`

          > **原理**就相当于首先把`students1`与`students2`中符合`ON`关键字指示的相等条件的记录拼接起来
          >
          > **然后**再在拼接好后的虚拟数据表中**自动添加**上**左表**中没有被添加到该虚拟数据表中的记录添加进来(**当然这会出现此时插入的记录的字段数与前面的记录字段数不匹配的情况,系统会自动补齐字段数,并将用于补齐该记录字段数的字段设置为`NULL`**)
          >
          > **然后**再对这个合并后的虚拟数据表进行查询

    - **右外连接**($(A ∩ B)∪(B-A)$)

      - 在**内连接**的基础上,还会返回**右表**中没有被呈现的记录的对应的被查询字段

      - **`SQL92`**

        - `SELECT s1.age,s1.name,s2.age,s2.name FROM students1 s1,students2 s2 WHERE(s1.height(+) = s2.height)`

      - **`SQL99 `RIGHT OUTER JOIN**

        - `SELECT s1.age,s1.name,s2.age,s2.name FROM students1 s1 RIGHT OUTER JOIN students2 s2 ON(s1.height = s2.height)`

          > **原理**就相当于首先把`students1`与`students2`中符合`ON`关键字指示的相等条件的记录拼接起来
          >
          > **然后**再在拼接好后的虚拟数据表中**自动添加**上**右表**中没有被添加到该虚拟数据表中的记录添加进来(**当然这会出现此时插入的记录的字段数与前面的记录字段数不匹配的情况,系统会自动补齐字段数,并将用于补齐该记录字段数的字段设置为`NULL`**)
          >
          > **然后**再对这个合并后的虚拟数据表进行查询

    - **满外连接(全外连接)**($A∪B$)

      - 在**内连接**的基础上,还会返回**左表与右表**中没有被呈现的记录的对应的被查询字段
      - **`SQL92`**
      - **`SQL99`** **FULL OUTER JOIN**
        - `SELECT s1.age,s1.name,s2.age,s2.name FROM students1 s1 FULL OUTER JOIN students2 s2 ON(s1.height = s2.height)`
      - **`MySQL`**(**不支持满外连接**)

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-2?token=AOAPFCLPFHMMEQTVORFRY33C57WFW" alt="image-20220630211243062" style="zoom:80%;" />

****

### `MySQL`使用`UNION`与`UNION ALL`实现满外连接

> **注意**:这两个关键字都是用于对两个`SELECT`查询的结果进行拼接,但是他们的拼接效果不同.这里我们假设第一个`SELECT`查询有$$10$$条查询结果,第二条查询有$$20$$条查询结果,并且两个查询的查询结果中有$$5$$条记录是重复的
>
> - `UNION`:拼接后会返回一个$(10-5)+5+(20-5)=25$条记录的结果
>
> - `UNION ALL`:拼接后会返回一个$[(10-5)+5]+[5+(20-5)]=30$条记录的结果下`下`

![image-20220630211718491](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-3?token=AOAPFCJD7K3UVBCA33SCTV3C57WHE)

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-4?token=AOAPFCJTDEGAUUG4CG5AKL3C57WIE" alt="image-20220630212408645" style="zoom:67%;" />

****

### **`SQL99`新特性(自然连接`NATURAL JOIN`与`USING`连接)**

#### 自然连接`NATURAL JOIN`

> `NATURAL JOIN`会自动寻找两个待连接的表中**字段名相同的所有字段**，并且按照这些字段对我们的两个表进行等值连接

- `SELECT s1.age,s1.name,s2.age,s2.name FROM students1 s1 NATURAL JOIN students2 s2`

#### `USING`连接

> 我们可以通过`USING`**指定一个或多个两个表中都有的字段**，那么我们的两个表就会根据指定的这一个或多个字段对两个表进行等值连接

- `SELECT s1.age,s1.name,s2.age,s2 .name FROM students1 s1  JOIN students2 s2 USING(height)`

****

### ==`MySQL`的多表外连接的字段补齐问题==

> - 我们容易知道多表之间的外连接，由于会将左表或右表或左右表中没有符合连接条件的记录也会最终拼接到我们的待呈现虚拟表上，因此这就会带来两个个问题
>   - **问题1**：拼接的不符合条件的记录的字段数与符合连接条件的合并记录的字段数是不同的，那么如何让字段数不相同的记录共存在同一个虚拟表中呢？
>   - **问题2**：对于左外连接，是不是左表中所有不符合连接条件的记录都会被拼接到虚拟表中符合连接条件的合并记录之后呢？那右外连接和全外连接又是什么样的顺序呢？

****

### ==`MySQL`的多表连接的`WHERE`子句条件过滤问题==

> 我们可以对这个过滤做出如下简易化理解，但不代表这是`MySQL`的内部实现方式

- **首先**`MySQL`会对我们要连接的两个表进行笛卡尔积拼接操作(如左表$$10$$条记录,右表$$10$$条记录,那么拼接好后的虚拟表中则有$100$条合并记录)
- **然后**`MySQL`就会对我们上一步得到的共$100$条记录的虚拟数据表进行`WHERE`过滤操作
- **然后**通过上一步,$100$条合并记录中那些不符合连接条件的记录就会被我们的`MySQL`从虚拟数据表中剔除.
- **然后**`MySQL`就会对最终的虚拟数据表进行`SELECT`查询操作,并将得到的查询结果返回给用户

****

### ==`SQL`查询的简易化理解==

> 我们可以将`SQL`查询的过程理解为一个对数据表中每一个记录进行遍历筛选的过程.因此我们可以将`SQL`查询中的字段名理解为一个用于遍历的变量,其会在遍历数据表的每一条记录时跟随记录的变化而变化.**例如当遍历到第`t`个记录时,那么该`SQL`语句中的那些`WHERE`子句中的字段都可以认为是改记录中该字段所对应的值.**
>
> - 如第`t`条记录为
>
> | 姓名 | 年龄 | 身高 |
> | ---- | ---- | ---- |
> | 汤凌 | 20   | 181  |
>
> - 查询语句为`SELECT 姓名 FROM students WHERE 身高=181`
> - 那么查询到该条数据时查询语句实际为`SELECT 姓名 FROM students WHERE 181=181`

#### 单表查询

- 遍历到第`t`个记录,发现改记录符合`WHERE`表示的过滤条件.将该记录加入到待呈现记录集合
- 遍历完整张数据表,得到完整的待呈现数据表
- 将待呈现数据表按照`ORDER BY`子句进行排序
- 按照`LIMIT`子句提取待呈现数据表中的指定个记录组成预备数据表
- 将我们`SELECT`中指定的要呈现的字段从排序好,并提取好的预备数据表提取出来
- 按顺序将我们上一步提取得到的数据呈现出来

#### 多表查询

> 多表查询涉及到多个表的记录的遍历问题.实际上我们可以理解为一个二重`for`循环,第一个循环的循环元素为第一个表的记录,第二个循环的循环元素为第二个表的记录

```python
虚拟数据表=[]
for 记录1 in Table1:
	for 记录2 in Table2:
		合并记录 = [记录1,记录2]
        if 合并记录 符合 WHERE过滤条件:
            虚拟数据表.append(合并记录)

SELECT 字段1,字段2... FROM 虚拟数据表
		
```



****

## 单行函数与聚合函数的区别

> - **单行函数**:即一次接收数据表的**一个**记录,字段,常量等,并返回一个结果
> - **聚合函数**:即一次可以接收数据表的**多个**记录,字段,常量等,返回一个结果

## **单行函数**

不同`DBMS`的函数的差异

### 数值函数

### 字符串函数

### 日期与时间函数

### 流程控制函数

### 加密与解密函数

### `MySQL`信息函数

### 其他函数

****

## **聚合函数**

****

## ==子查询(查询进阶)==

> 为什么要引入子查询?子查询可以实现的功能不应该通过多句查询也能够实现吗?
>
> - `FROM`子句中可以写子查询
> - `JOIN|LEFT JOIN|FULL JOIN|RIGHT JOIN`子句中可以写子查询
> - `WHERE`子句中可以写子查询
> - `HAVING`子句中可以写子查询
> - `ORDER BY`子句中可以写子查询

### 子查询应遵守的规则与规范

- 不相关子查询会在主查询开始前完成,并只会完成一次
- 相关子查询会随着主查询的进行而进行
- 子查询的结果可以被主查询多次使用
- 子查询必须由括号包括
- 子查询应放在`WHERE`子句或`HAVING`子句的比较符号右侧以提高可读性

### 单行子查询

> 单行子查询的查询结果**能且只能为一条记录**

#### 单行子查询比较操作符

| 操作符  | 含义     |
| :-----: | -------- |
|  **>**  | 大于     |
|  **<**  | 小于     |
| **>=**  | 大于等于 |
| **<=**  | 小于等于 |
|  **=**  | 等于     |
| **<=>** | 安全等于 |
| **<>**  | 不等于   |

#### 实例

- ```SQL
  #查询Abel所在的部门所处的城市
  SELECT city
  FROM locations
  WHERE(location_id = (
  				     SELECT location_id
  					 FROM departments
  					 WHERE(department_id = (																				    SELECT department_id 
  	     									FROM employees 
  		 									WHERE last_name='Abel'
  										   )
  						  )
  				    )
  	 );
  ```

- ```SQL
  #查询employees表中所有薪资大于Abel的员工的last_name和salary
  SELECT last_name,salary
  FROM employees
  WHERE salary>(
      		  SELECT salary 
      		  FROM employees 
      		  WHERE last_name='Abel'
  			 );
  ```

- ```SQL
  #查询employees表中所有薪资小于Abel的员工的last_name和salary
  SELECT last_name,salary
  FROM employees 
  WHERE salary<(
      		  SELECT salary 
      		  FROM employees 
      		  WHERE last_name='Abel'
  			 );
  ```

### 多行子查询

> 多行子查询的查询结果**为多条记录**

#### 多行子查询比较操作符

| 操作符 | 含义                                                         |
| :----: | ------------------------------------------------------------ |
|   IN   | 等于==**列表**==中的某一个则返回值为真                       |
| NOT IN | 不等于==**列表**==中的任何一个则返回值为真                   |
|  ANY   | 与``单行操作符``配合使用,如与`>`搭配表示只要**大于==子查询==返回的字段中任何一个值**返回值就为真 |
|  ALL   | 与``单行操作符``配合使用,如与`>`搭配表示只有**大于==子查询==返回的字段的每一个值**返回值才为真 |
|  SOME  | 等价于ANY                                                    |

#### 实例

- ```SQL
  #查询employees表中所有工资同时大于Abel,Mary,Jack的员工的last_name和salary
  
  SELECT last_name,salary 
  FROM employees 
  WHERE(salary > (
      			SELECT MAX(salary)
                  FROM employees
                  WHERE last_name IN ('Abel','Mary','Jack')
                 )
        );
  ```

  

### 相关子查询

> 子查询利用到了主查询的字段信息等,导致了**随着主查询的进行,子查询的语句会发生改变,**因此子查询会随着主查询的进行而多次进行.而不是只在主查询开始前执行一次

#### 实例

##### 条件相关子查询

- ```SQL
  #查询employees表中所有salary大于其所在部门的平均薪资的员工的salary和last_name
  SELECT salary,last_name
  FROM employees as e1 
  WHERE(salary > (
        		  	SELECT AVG(salary) 
  			  	FROM employees as e2 
  			  	WHERE(e2.department_id <=> e1.department_id))
  	 );
  ```

##### 虚拟表子查询(==**即将子查询结果作为一个虚拟表用于`FROM`子句**==)

- ```SQL
  #查询employees表中所有salary大于其所在部门的平均薪资的员工的salary和last_name
  
  #SQL92
  SELECT e1.salary,e1.last_name 
  FROM employees as e1,(
      				  SELECT department_id,AVG(salary) as AVG_salary
                        FROM employees 
                        GROUP BY department_id
  					 ) as e2
  					 
  WHERE((e1.department_id<=>e2.department_id)
  AND (e1.salary>e2.AVG_salary));
  
  #SQL99
  SELECT e1.salary,e1.last_name 
  FROM employees as e1 JOIN (
      					   SELECT department_id,AVG(salary) as AVG_salary
                        	   FROM employees 
                        	   GROUP BY department_id
  						  ) as e2
  ON e1.department_id<=>e2.department_id
  AND e1.salary>e2.AVG_salary;
  ```
  
- ```SQL
  #查询employees表中所有员工的employee_id和salary,并按照他们所属的department_name进行排序
  
  #SQL92  有一点小问题
  SELECT e1.employee_id,e1.salary 
  FROM employees as e1,(SELECT department_name,department_id
           			  FROM employees as e2,departments as d1
           			  WHERE e2.department_id <=> d1.department_id ) as s1
  WHERE e1.department_id <=> s1.department_id
  ORDER BY s1.department_name;
  #改进并正确解决了问题
  SELECT e1.employee_id,e1.salary 
  FROM employees as e1,(SELECT department_name,department_id
           			  FROM employees as e2,departments as d1
           			  WHERE e2.department_id <=> d1.department_id ) as s1
  WHERE e1.department_id <=> s1.department_id(+)
  ORDER BY s1.department_name;
  
  #SQL99  有一点小问题
  SELECT DISTINCT employee_id,salary 
  FROM employees as e1 JOIN (
      					   SELECT d1.department_name,d1.department_id
           			  	   FROM employees as e2,departments as d1
           			  	   WHERE e2.department_id <=> d1.department_id
  						  ) as s1
  ON e1.department_id <=> s1.department_id
  ORDER BY s1.department_name;
  #改进并正确解决了问题
  SELECT DISTINCT employee_id,salary 
  FROM employees as e1 LEFT JOIN (
      					   SELECT d1.department_name,d1.department_id
           			  	   FROM employees as e2,departments as d1
           			  	   WHERE e2.department_id <=> d1.department_id
  						  ) as s1
  ON e1.department_id <=> s1.department_id
  ORDER BY s1.department_name;
  
  #方式3  正确解决了问题   
  SELECT e1.employee_id,e1.salary
  FROM employees as e1
  ORDER BY (SELECT department_name
            FROM departments as d1
            WHERE d1.department_id=e1.department_id);
  ```

### 不相关子查询

> 不相关子查询就是子查询中并**不会利用到主查询中会随着查询过程的进行而发生变化的量**,如``字段``等.因此**不相关子查询会且只会在主查询开始之前执行一次**

### `EXISTS`与`NOT EXISTS`

> - `EXISTS`后应跟随一条子查询,如果该子查询的查询结果为空,那么**返回假**,如果子查询的查询	结果有一条或以上的记录那么**返回真**
> - `NOT EXISTS`:与`EXISTS`正好相反

### 基本使用示范

```mysql
EXISTS(子查询语句);
NOT EXISTS(子查询语句)
```

### `EXISTS`优化多表连接

```SQL
#查询departments表中,不存在于employees表中的部门的department_id和department_name

#方式1 错误结果
SELECT d1.department_id,d1.department_name
FROM departments as d1
WHERE d1.department_id NOT IN (
							   SELECT DISTINCT department_id
    						   FROM employees
							  );
#方式1  正确结果							  
SELECT d1.department_id,d1.department_name
FROM departments as d1
WHERE d1.department_id NOT IN (
							   SELECT DISTINCT department_id
    						   FROM employees
							   WHERE department_id  IS NOT NULL
							  );
#方式2
SELECT d1.department_id,d1.department_name
FROM departments as d1
WHERE NOT EXISTS(SELECT *
             FROM employees as e1
             WHERE e1.department_id <=> d1.department_id);

#方式3
SELECT d1.department_id,d1.department_name
FROM departments as d1 LEFT JOIN employees as e1
ON d1.department_id<=>e1.department_id
WHERE e1.department_id <=> NULL;
```

****

# 第三部分

## `MySQL`数据库的创建

### 命名规则

![image-20220701211637422](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-5?token=AOAPFCK4NJ7CLNUQCM63LZ3C57WJ4)

## ==`MySQL`数据类型==

### 数据类型总结

| 数据类型         | 对应关键字                                                   |
| ---------------- | ------------------------------------------------------------ |
| 整数类型         | TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT                    |
| 浮点数类型       | FLOAT，DOUBLE                                                |
| 定点数类型       | DECIAL                                                       |
| 位类型           | BIT                                                          |
| 时间日期类型     | YEAR，TIME，DATA，DATATIME，TIMSTEAMP                        |
| 文本字符串类型   | CHAR，VARCHAR，TINYTEXT，TEXT，MEDIUMTEXT，LONGTEXT          |
| 枚举类型         | ENUM                                                         |
| 集合类型         | SET                                                          |
| 二进制字符串类型 | BINARY，VARBINARY，TINYBLOB，BLOB，MEDIUMBLOB，LONGBLOB      |
| JSON类型         | JSON对象，JSON数组                                           |
| 空间数据类型     | 单值：GEOMETRY，POINT，LINESTRING，POLYGON；<br />集合：MULTIPOINT，MULTILINESTING，MULTIPOLYGON，GEOMETRYCOLLECTION |

### 特殊类型

| **关键字**                 | **含义**                                             |
| -------------------------- | ---------------------------------------------------- |
| **NULL**                   | **字段可以为空**                                     |
| **NOT NULL**               | **字段不可未空**                                     |
| **DEFAULT**                | 用于**指定字段默认值**                               |
| **PRIMARY KEY**            | **字段为主键**,主键是唯一,不可重复的                 |
| **AUTO_INCREMENT**         | **自动递增**,**适用于整数类型**.如用于学号的自动生成 |
| **UNSIGNED**               | **字段为无符号类型**                                 |
| **CHARACTER SET 字符集名** | 用于**指定字段使用的字符集**                         |

### 整数类型

> **注意事项**
>
> - 如果使用了`ZEROFILL`那么**系统会自动同时**使用`UNSIGNED`
> - 宽度指定在整数类型上是不推荐使用的

![image-20220702182014108](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-6?token=AOAPFCMUAI27DR3UIUKA3RTC57WKK)

- 宽度指定`数据类型(<宽度>)`

  ```SQL
  INT(5)#表示最小宽度为5,要配合ZEROFILL使用否则无效果(小于5就0填充,大于5则无影响)
  ```

- 无符号指定`UNSIGNED`

  ```SQL
  INT UNSIGNED#指定为无符号数据类型
  ```

- 零填充`ZEROFILL`

  ```SQL
  INT ZEROFILL
  ```

- 综合示例

  ```SQL
  INT(5) ZEROFILL
  ```

### 浮点数类型

> **在数据库中,我们不推荐使用浮点数类型**
>
> - **浮点数类型在进行不同`DBMS`间数据迁移时会带来很大麻烦**
> - **浮点数类型由于其取值离散的特性会发生如`1.0+1.0=1.99999`类似的问题.而`MySQL`不像一些程序语言对于这一问题会进行处理,因此就会导致发生极其大的错误`1+1 != 2`**
> - **由上面我们知道,我们如果用`=`来直接比较两个两个浮点数很可能发生问题**

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-7?token=AOAPFCPVGV7GDUWKDQTU52TC57WK6" alt="image-20220702182842998" style="zoom:90%;" />

- **精度与标度**

  > **精度与长度是截然不同的概念**,长度是指的最低数字位数,而精度是指最大数字位数

  ```SQL
  FLOAT(5)#表示该字段可以最多有5位数字(包括整数位与小数位)
  FLOAT(5,2)#表示该字段最多有5位,且小数部分必须位2位
  ```

- **无符号浮点数**

  > 无符号浮点数并不会使得取值返回变大,反而会缩小

  ```SQL
  FLOAT(5,2) UNSIGNED;
  ```

#### 浮点数对于取值的控制规则

- 不管是否显式设置了精度(M,D)，这里MySQL的处理方案如下：
  -  如果存储时，整数部分超出了范围，MySQL就会报错，不允许存这样的值
  - 如果存储时，小数点部分若超出范围，而整数部分没有超出范围,就分以下情况：
    - 若四舍五入后，**整数部分没有超出范围(即四舍五入到小数部分为指定位后)**，则**只警告**，但**能成功操作**并**四舍五入删除多余的小数位后保存**。例如在FLOAT(5,2)列内插入999.009，近似结果是999.01。
    -  若四舍五入后，**整数部分超出范围**，则**MySQL报错**，并**拒绝处理**。如FLOAT(5,2)列内插入 999.995和-999.995都会报错。

### 定点数类型

> - 定点数类型只有`DECIMAL`关键字
>
> - `DECIMAL(M,D)`关键字有如下可选属性
>   - `M`:作用同浮点数的该属性
>   - `D`:作用同浮点数的该属性
> - ![image-20220702210256169](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-8?token=AOAPFCK3SE2OXJGFJYYT4ATC57WLY)

#### 注意事项

- 我们从上面可以知道,`DECIMAL`的空间占用随着`M`的变化而变化.其取值范围也随着`M`的变化而变化.但是我们可以很容易有这样的结论**在占用相同字节的情况下浮点数相比于定点数有更大的取值范围**
- ==**定点数在``MySQL``内部是以``字符串``的形式进行存储，这就决定了它一定是精准的。**==
- 当``DECIMAL``类型不指定精度和标度时，其默认为``DECIMAL(10,0)``。当数据的精度**超出了定点数类型的 精度范围**时，则``MySQL``同样会进行四舍五入处理。

###  位类型

| 数据类型   | 位数 | 位数取值范围 | 占用空间                      |
| ---------- | ---- | ------------ | ----------------------------- |
| ``BIT(M)`` | `M`  | $[1,64]$     | `M`比特,$$\frac{M+7}{8}$$字节 |

### 日期时间类型

![image-20220702224514372](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-9?token=AOAPFCJCFOMPYRZWIDEMBZ3C57WMO)

> **注意事项**
>
> - 2位`YEAR`类型**不推荐使用**,应**尽量使用**4位`YEAR`类型(**更明确**)
> - `DATA`类型推荐使用标准的`YYYY-MM-DD`
> - `TIME`类型推荐使用标准的`D HH:MM:SS`或`HH:MM:SS`格式
>   - `D HH:MM`或`HH:MM`
>   - `D HH`或`HH`
>   - `D SS`或`SS`
>   - `HHMMSS`或`'HHMMSS'`
> - `DATATIME`类型推荐使用标准的`YYYY-MM-DD HH:MM:SS`格式
>   - `YYYYMMDDHHMMSS`
> - `TIMSTEAMP`类型推荐使用`YYYY-MM-DD HH:MM:SS`格式

#### `TIMESTAMP`与`DATATIME`的区别

> 这个类型要单独说,因为`TIMESTAMP`涉及到一些`DBMS`的时间区自动转换

- `TIMESTAMP`存储空间比`DATATIME`小，表示的日期时间范围也比较小
- 底层存储方式不同，``TIMESTAMP``底层存储的是毫秒值，距离``1970-1-1 00:00:00``毫秒的毫秒值。
- **两个日期比较大小或日期计算时，TIMESTAMP更方便、更快。**
- **``TIMESTAMP``和时区有关。``TIMESTAMP``会根据用户的时区不同，显示不同的结果。而``DATETIME``则只能 反映出插入时当地的时区，其他时区的人查看数据必然会有误差的。**

### 文本字符串类型

> **注意事项**
>
> - **由于实际存储的长度不确定**,`MySQL`不允许``TEXT`` 类型以及其衍生类型的**字段做主键**
> - `TEXT`文本类型，可以存比较大的文本段，搜索速度稍慢，因此**如果不是特别大的内容，建议使用``CHAR`， `VARCHAR`来代替**。还有**`TEXT`类型不用加默认值**，加了也没用。而且`TEXT`和`BLOB`类型的**数据删除后容易导致 “空洞”**，使得文件碎片比较多，所以**频繁使用的表不建议包含`TEXT`类型字段**，建议**单独分出去，单独用一个表,要使用时采用多表查询的方式**。
> - `ENUM`的效果类似于**自定义选项内容以及个数**的**单选题**
> - `SET`的效果类似于**自定义选项内容以及个数**的**多选题**

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-10?token=AOAPFCNVOG6J35LVXGFN7BDC57WNC" alt="image-20220702213409420" style="zoom:89%;" />

#### `CHAR`与`VARCHAR`类型的使用与比较

##### `CHAR`

> - `CAHR`类型可以通过`CHAR(M)`来指定其能够容纳的字符的最大值,如果不指定则默认`M=1`
> - `CHAR`类型的数据存储中的**溢出**与**过短**问题
>   - 当我们的字符串中包含的字符数量大于该`CHAR`类型字段的指定最大长度时,**操作会失败**
>   - 当我们的字符串中包含的字符数量少于`CHAR`类型字段的指定最大长度时,`DBMS`会**自动给我们的字符串的右侧添加空格符号**(==**当然我们查询该字段时,`DBMS`又会把添加的空格去掉**==)
> - 从上面我们可以知道如果我们指定了``M=5``,那么就有无论我们存储的字符串的长度是`0(空串),1,2,3,4,5`中的任何一个`CAHR`占用的存储空间都为`5字节`

##### `VARCHAR`

> - `VARCHAR`类型是必须以`VARCHAR(M)`的方式出现的,不指定长度是不被`DBMS`允许的
> - `VARCHAR(M)`表示其最大占据`M+1字节`的存储空间,最小占据``1字节``空间.最大允许存储`M`个字符
> - `VARCHAR`的数据溢出与过短问题(以``VARCHAR(15)``为例)
>   - 当字符串长度超过``15``时,我们的操作会引发系统错误
>   - **当字符串长度小于等于``15``时(假设为``10``),`VARCHAR`类型会正确地不加任何操作地保存该字符串,并且此时占据的存储空间为`10+1=11`字节**
> - 我们注意到`VARCHAR`存储空间占用需要在字符数的基础上`+1`,其原因如下
>   - 首先,由于该类型为一个可变长度类型,因此同一个表中每一个记录下的该类型的字段都可能有不同长度
>   - **然后,我们的`VARCHAR`类型就必须具备一个能够存储其对应记录的该字段的长度为多少,因此我们就为``VARCHAR``类型多分配了一个字节用于保存他存储的字符串的字符数**

##### `CHAR`与`VARCHAR`类型的比较(前提为我们谈论的都是不发生溢出的情况)

- `CHAR`类型无论保存的字符串为多长,他都占据**一个固定的字节数**.
- `VARCHAR`类型的**存储空间占用会随着其保存的字符串的长度的变化而随时变化**,如对于一个长度为`5`的字符串,`VARCHAR`类型会占据`6`个字节,如对于一个长度为`15`的字符串,`VARCHAR`类型会占据`16`个字节

#### `TEXT`类型与其衍生类型

> **我们从主观上可以把`TEXT`类型与其衍生类型看作一个升级版的`VARCHAR`类型(底层上是不可以这样理解的,两者对于性能的影响不同)**,其升级在于
>
> - **能够存储长度更长的字符串**
> - **不需要且也无法指定其数据长度(`TEXT(20)`是不被支持的)**
>
> **注意**:由于`TEXT`类型及其衍生类型能够保存的字符串长度越来越长,因此只用一个字节已经无法保存其所存储的字符串的长度,这也就导致了不同的`TEXT`类型有的`+2字节`有的`+3字节`甚至有的`+4字节`

![image-20220703000157545](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-11?token=AOAPFCO4DFMQJVJFSHBAUD3C57WNY)

#### `ENUM`枚举类型

> **`ENUM`枚举类型在使用时可以设置枚举列表**,当不把该字段设置为`NOT NULL`时我们可以按照如下规则向该字段存入数据
>
> - **==可以且仅可以==存入枚举列表中的某一个元素或`NULL`**
> - **`NULL`可以被存入**
> - **如果存入的数据既不属于枚举列表,有不是`NULL`时,操作会失败.**
> - **如果以`'字符串1,字符串2,字符串3'`的方式存入数据,就算三个字符串都是枚举列表中的元素,存储也会失败**
> - **只有`'字符串'`或`NULL`两种方式被支持**

![image-20220703000755965](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-12?token=AOAPFCOGRMXI5A3JTEEEK2LC57WOI)

##### 使用示例

```SQL
CREATE TABLE test_enum(
season ENUM('春','夏','秋','冬','unknow')
);

INSERT INTO test_enum
VALUES('春'),('秋');
# 忽略大小写
INSERT INTO test_enum
VALUES('UNKNOW');
# 允许按照角标的方式获取指定索引位置的枚举值
INSERT INTO test_enum
VALUES('1'),(3);
# Data truncated for column 'season' at row 1
INSERT INTO test_enum
VALUES('ab');
# 当ENUM类型的字段没有声明为NOT NULL时，插入NULL也是有效的
INSERT INTO test_enum
VALUES(NULL);
```

#### `SET`集合类型

> `SET`集合类型从作用上类似于加强版`ENUM`类型
>
> **`SET`集合类型在使用时可以设置集合列表**,当不把该字段设置为`NOT NULL`时我们可以按照如下规则向该字段存入数据
>
> - **==可以且仅可以==存入枚举列表中的一个或多个元素或`NULL`**
> - **`NULL`可以被存入**
> - **如果存入的数据既不属于集合列表,又不是`NULL`时,操作会失败.**
> - **如果以`'字符串1,字符串2,字符串3'`的方式存入数据,只要三个字符串都是集合列表中的元素,存储会顺利地正确地进行**
> - **有`'字符串1'`或`字符串1,字符串2,字符串3`或`NULL`三种存入方式被支持**

![image-20220703000812060](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-13?token=AOAPFCMASJEHEJIW3BFWA33C57WO4)

##### 使用示例

- ```SQL
  CREATE TABLE test_set(
  s SET ('A', 'B', 'C')
  );
  
  INSERT INTO test_set (s) VALUES ('A'), ('A,B');
  #插入重复的SET类型成员时，MySQL会自动删除重复的成员
  INSERT INTO test_set (s) VALUES ('A,B,C,A');
  #向SET类型的字段插入SET成员中不存在的值时，MySQL会抛出错误。
  INSERT INTO test_set (s) VALUES ('A,B,C,D');
  SELECT *
  FROM test_set;
  ```

- ```SQL
  CREATE TABLE temp_mul(
  gender ENUM('男','女'),
  hobby SET('吃饭','睡觉','打豆豆','写代码')
  );
  
  INSERT INTO temp_mul VALUES('男','睡觉,打豆豆'); #成功
  # Data truncated for column 'gender' at row 1
  INSERT INTO temp_mul VALUES('男,女','睡觉,写代码'); #失败
  # Data truncated for column 'gender' at row 1
  INSERT INTO temp_mul VALUES('妖','睡觉,写代码');#失败
  INSERT INTO temp_mul VALUES('男','睡觉,写代码,吃饭'); #成功
  ```

### 二进制字符串类型

> ==**二进制字符串类型会将我们的存入数据自动转换为二进制后进行保存,当然当我们读取它时,系统又会自动将其还原**==
>
> **注意事项**：
>
> - `VARBINARY`使用时必须指明长度,否则报错
> - `BLOB`类型与其衍生类型一般用于保存`图片`,`音频`,`视频`等数据.
> - **通常我们并不应该使用`BLOB`去直接保存`图片`等文件.而是应该将这类大存储空间占用数据存储在磁盘上,而我们的数据库只保存从磁盘上访问该数据的路径方法**

![image-20220703122526722](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-14?token=AOAPFCI64EYKU2A3DRKP2SLC57WPW)

![image-20220703123013458](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-15?token=AOAPFCMEDSBFNVMR7YGFH33C57WQG)

##### 使用实例

- ```SQL
  CREATE TABLE test_binary1(
  f1 BINARY,
  f2 BINARY(3),
  # f3 VARBINARY,
  f4 VARBINARY(10)
  );
  
  INSERT INTO test_binary1(f1,f2)
  VALUES('a','a');
  INSERT INTO test_binary1(f1,f2)
  VALUES('尚','尚');#失败
  
  INSERT INTO test_binary1(f2,f4)
  VALUES('ab','ab');
  
  SELECT LENGTH(f2),LENGTH(f4)
  FROM test_binary1;
  ```

- ```SQL
  CREATE TABLE test_blob1(
  id INT,
  img MEDIUMBLOB
  );
  
  ```

### `JSON`类型

- 基于`->`或`->>`来对`JSON`数据进行检索

### `AUTO_INCREMENT`自增长类型

> **前提**
>
> - **每一个数据表只能最多设置一个字段为自增长类型**
> - 只有被设置为一个`KEY`(`PRIMARY KEY`,`FOREIGN KEY`等)的字段才可以设置为自增长类型
>
> **注意事项**
>
> - **当我们最新的记录的该自增类型字段已经自增到了`10`,此时我们插入一个数据,这个数据指明了该自增字段为`20`,那么我们的数据库就会将其值原封不动地按照`20`保存,并且后续再插入不指定该自增字段的记录时,会从`20`开始自增,而不是`10`**
> - **当存入的记录的该自增类型字段的值为`0`或`NULL`时,等价于没有指定该自增字段的值时.**
> - **当我们最新记录的该字段为`10`,然后我们添加了一个字段值为`-10`的记录,然后再添加一个不指定该字段的记录,那么这个记录的该字段的值为`11`**

- **使用自增类型**

  ```SQL
  #错误,会发生输入字段数与表记录字段数不匹配的问题
  INSERT INTO students
   VALUES('汤凌');
  #正确
  
  DROP TABLE IF EXISTS students;
   
  CREATE TABLE students(
  id INT PRIMARY KEY AUTO_INCREMENT,
  last_name VARCHAR(25)
  );
  
  
  INSERT INTO students(last_name)
  VALUES('汤凌');
  
  INSERT INTO students(last_name)
  VALUES('马婷婷');
  
  INSERT INTO students
  VALUES(20,'马婷蓉');
  
  INSERT INTO students(last_name)
  VALUES('汤凌凌');
   
  SELECT * FROM students;
  ```

- **数据表创建好后再添加自增类型**

  ```SQL
  ALTER TABLE employees
  MODIFY id INT PRIMARY KEY AUTO_INCREMENT;
  ```

- **数据表创建好后取消指定字段的自增**

  ```SQL
  ALTER TABLE employees
  MODIFY id INT PRIMARY KEY;
  ```

#### `MySQL 8.0`的自增列新特性

> `MySQL 5.7`中自增列计数器只维护在内存,磁盘上并未维护,因此当我们重启**`MySQL`服务**时
>
> - 计数器会在停止时被清空
> - 然后启动时根据表上的信息重建.
>   - 我们计数器到达了``10``(最新的记录的自增列值为``10``),表中有``10``条记录
>   - 我们删掉``5``条(此时最后一条记录的自增列值为``6``),然后重启`MySQL`服务
>   - 此时我们的计数器的值会变成`6`而不是``10`
>
> `MySQL 8.0`中自增列计数器同时在内存,磁盘上都有维护.因此当我们重启**`MySQL`服务**时
>
> - 计数器会在停止时,先将其内容覆盖到磁盘上,然后清空
> - 当服务启动时,`MySQL`就会到磁盘上读取该计数器到内存中.
> - 因此重启服务在`MySQL 8.0`下对计数器内容无影响

## ==**`MySQL`创建与管理库与表**==

### **创建管理数据库**

> - **`CREATE DATABASE`**
> - **`USE`**
> - **`CHARACTER SET`**
> - **`IF NOT EXISTS`**
> - **`IF EXISTS`**
> - **`SHOW`**
> - **`SHOW CREATE`**
> - **`ALTER DATABASE`**
> - **`DROP DATABASE`**

### **数据库创建**

- **创建数据库**

  ```SQL
  CREATE DATABASE <数据库名>;
  ```

- **创建数据库,并指定字符集**

  ```SQL
  CREATE DATABASE <数据库名> CHARACTER SET <utf8|gbk2312|latin...>;
  ```

- **判断数据库是否存在,不存在则创建**

  ```SQL
  CREATE DATABASE IF NOT EXISTS <数据库名>;
  ```

- **判断数据库是否存在,不存在则创建并指定字符集**

  ```SQL
  CREATE DATABASE IF NOT EXISTS <数据库名> CHARACTER SET <utf8|utf8mb4|utf8mb3|		gbk2312|latin>;
  ```

### **数据库管理**

- **显示所有数据库**

  ```SQL
  SHOW DATABASES;
  ```

- **使用指定数据库**

  ```SQL
  USE <数据库名>;
  ```

- **显示当前数据库中所有的表**

  ```SQL
  SHOW TABLES;
  ```

- **查看当前使用的数据库**

  ```SQL
  SELECT DATABASE();
  ```

- **显示指定数据库下的全部的表**

  ```SQL
  SHOW TABLES FROM <数据库名>;
  ```

- **更改数据库默认字符集**

  ```SQL
  ALTER DATABASE <数据库名> CHARACTER SET <utf8|utf8mb4|utf8mb3|gbk2312|latin>;
  ```

- **显示数据库的结构**

  ```SQL
  SHOW CREATE DATABASE <数据库名>;
  ```

- **删除指定数据库**

  ```SQL
  #如果要删除的数据库不存在,会报错
  DROP DATABASE <数据库名>;
  #如果要删除的数据库存在就会删除,如果不存在就不会有任何行动
  DROP DATABASE IF EXISTS <数据库名>;
  ```

### **创建管理表**

> - **`CREATE TABLE`**
> - **`CREATE TABLE AS`**
> - **`DESC`**
> - **`SHOW CREATE`**
> - **`ALTER TABLE ADD`**
> - **`ALTER TABLE MODIFY`**
> - **`ALTER TABLE CHANGE`**
> - **`ALTER TABLE DROP COLUMN`**
> - **`ALTER TABLE RENAME TO`**
> - **`RENAME TABLE TO`**
> - **`DROP TABLE`**

#### **表创建**

> - **创建表时如果没有指明应该使用的字符集,那么会使用该表所在的数据库的默认字符集**

- **直接创建表**

  ```SQL
  CREATE TABLE IF NOT EXISTS <表名>(
      							  字段1 数据类型(长度) 默认值 约束,
                                 	  字段2 数据类型(长度) 默认值 约束,
                                 	  字段3 数据类型(长度) 默认值 约束
  								 );
  ```

- **根据查询语句创建表**

  > - **查询语句中如果对字段起了别名,那么创建的表中对应字段的名字会使用我们的别名**
  > - **查询语句可以使用极其丰富的结构,多表查询,表连接,单行函数,聚合函数等等都可以使用.**

  ```SQL
  #一般的方式
  CREATE TABLE <表名>
  AS
  SELECT employee_id,last_name_salary 
  FROM employees;
  
  #表连接
  CREATE TABLE <表名>
  AS
  SELECT *
  FROM employees AS e JOIN departments AS d
  ON e.department_id <=> d.department_id;
  
  #为字段起别名
  CREATE TABLE <表名>
  AS
  SELECT employee_id AS emp,last_name AS Lname,department_name AS Dname
  FROM employees AS e JOIN departments AS d
  ON e.department_id <=> d.department_id;
  
  #查询中使用函数
  CREATE TABLE <表名>
  AS
  SELECT employee_id*50 AS emp,UPPER(last_name) AS Lname,LOWER(department_name) AS Dname
  FROM employees AS e JOIN departments AS d
  ON e.department_id <=> d.department_id;
  
  #复制表结构,同时还复制表数据
  CREATE TABLE <表名>
  AS
  SELECT *
  FROM employees;
  
  #只复制表结构,不复制表数据
  CREATE TABLE <表名>
  AS
  SELECT * 
  FROM employees
  WHERE 0;	
  
  CREATE TABLE <表名>
  AS
  SELECT * 
  FROM employees
  WHERE NULL;	
  ```

#### **表管理**

- **查看表结构**

  ```SQL
  #方式1
  DESC <表名>;
  #方式2
  SHOW CREATE TABLE <表名>
  ```

##### **修改表**

- **添加字段**

  ```SQL
  #添加并作为最后一个字段
  ALTER TABLE <表名>
  ADD <字段名> 数据类型 默认值 约束;
  
  #添加并作为第一个字段
  ALTER TABLE <表名>
  ADD <字段名> 数据类型 默认值 约束 FIRST;
  
  #添加到指定字段之后
  ALTER TABLE <表名>
  ADD <字段名> 数据类型 默认值 约束 AFTER <字段名>;
  ```

- **修改指定字段的数据类型,长度,默认值等**

  ```SQL
  ALTER TABLE <表名>
  MODIFY <字段名> 数据类型(长度) DEFAULT 默认值;
  
  ALTER TABLE <表名>
  MODIFY <字段名> DEFAULT 默认值;
  
  ALTER TABLE <表名>
  MODIFY <字段名> 数据类型(长度);
  ```

- **重命名指定字段**

  ```SQL  
  ALTER TABLE <表名>
  CHANGE	<字段名> <新字段名> 数据类型(长度) DEFAULT 默认值;
  
  ALTER TABLE <表名>
  CHANGE	<字段名> <新字段名>;
  
  ALTER TABLE <表名>
  CHANGE	<字段名> <新字段名> 数据类型(长度);
  
  ALTER TABLE <表名>
  CHANGE	<字段名> <新字段名> DEFAULT 默认值;
  ```

- **删除指定字段**

  ```SQL
  ALTER TABLE <表名>
  DROP COLUMN <字段名>
  
  ALTER TABLE <表名>
  DROP COLUMN IF EXISTS <字段名>
  ```

##### **重命名表**

- **方式1**

  ```SQL
  RENAME TABLE <表名>
  TO <新表名>
  ```

- **方式2**

  ```SQL
  ALTER TABLE <表名>
  RENAME TO <新表名>
  ```

##### **删除表**

> **同时删除表结构与表数据**

- ```SQL
  DROP TABLE IF EXISTS <表名>
  ```

##### **清空表**

> **只删除表数据,表结构不会删除**

- ```SQL
  TRUNCATE TABLE <表名>
  ```

##### **按条件清空表**

- ```SQL
  DELETE *
  FROM employees;
  
  DELETE *
  FROM employees
  WHERE salary < 4500;
  ```

## ==`MySQL`表约束==

> **约束的作用**:

### 四大完整性

- **`实体完整性`:即保证同一个表中不能存在两条无法区分的记录(如保证两条记录的主键不同)**
  - `PRIMARY KEY`
- **`域完整性`:即对字段的取值范围进行约束,如约束年龄字段的取值区间为`1-150`**
- **`引用完整性`:如`employees`表中的员工所属的部门必须要在`departments`表中存在**
  - `REFERENCE`
- **`用户自定义完整性`:如用户名不能重复,密码不能为空等**
  - `UNIQUE`
  - `NOT NULL`

### 表约束的基本操作

- **查看指定表的约束**

  ```SQL
  SELECT *
  FROM infomation_schema.table_constraints
  WHERE table_name = '<表名>'
  ```

### `NOT NULL`非空约束

> **设置了非空约束的字段是不能为空的,如果存入的该字段数据为空就会报错,并且操作会失败**
>
> **注意事项**
>
> - 当我们给数据表的字段添加非空约束时,若**存在一个或以上的记录的该字段为空**,那么**此次操作会失败**
> - `NOT NULL`非空约束**只有列级写法没有表级写法** 
>
> **问题**
>
> - 约束添加操作是否具有原子性?

- **创建表时添加非空约束**

  ```SQL
  CREATE TABLE employees(
  id INT NOT NULL,
  department_id INT
  );
  DESC employees;
  ```

- **创建完毕后添加非空约束**

  ```SQL
  CREATE TABLE employees(
  id INT NOT NULL,
  department_id INT
  );
  DESC employees;
  ```

- **删除非空约束**

  ```SQL
  ALTER TABLE employees
  MODIFY id INT;
  ```

### `UNIQUE`唯一性约束

> **设置了唯一性约束的字段的所有记录下的该字段取值都不能相同**
>
> 注意事项
>
> - **一个表可以有多个字段设置为`UNIQUE`**
>
> - 唯一性约束有**表级约束**与**列级约束**两种
> - 表级约束可以命名,列级约束名字自动取为其对于的字段名
> - **唯一性约束的字段是允许为`NULL`的(并且==可以有多个记录取为`NULL`==)**
> - **在我们给已经存在的表的某个字段添加唯一性约束时,如果该字段的一些记录下的值已经发生了重复,那么此次唯一性约束添加操作将会失败并报错**
> - **当设置表级约束时括号中包括了多个字段,那么其唯一性判断如下(以`('test','abc)`为例)**
>   - **`('test','abc)`失败**
>   - **`('test','ab)`成功**
>   - **`('tes','abc)`成功**
>   - **`('tes','ab)`成功**

- **创建表时添加唯一性约束**

  ```SQL
  #列级约束:给字段id设置了唯一性约束,并且系统会自动将该唯一性约束的名字设置为id
  CREATE TABLE employees(
  id INT UNIQUE
  );
  DESC employees;
  
  #表级约束-1(只涉及一列唯一性):给字段department_id设置了唯一性约束,并给这个约束取名为<约束名>.
  #当然如果不指定名字就有如下情况
  	#单列则名字自动取为对应字段的名字
  	#多列则名字自动取为小括号内第一个字段的名字
  CREATE TABLE employees(
  id INT UNIQUE,
  department_id INT,
  CONSTRAINT <约束名> UNIQUE(department_id)
  );
  DESC employees;
  #查看employees表的所有约束
  SELECT *
  FROM information_schema.table_constrains
  WHERE table_name = 'employees';
  
  #表级约束-2(设置多个列组合的唯一性)
  CREATE TABLE employees(
  id INT,
  department_id INT,
  CONSTRAINT <约束名> UNIQUE(id,department_id)
  );
  DESC employees;
  
  #查看employees表的所有约束
  SELECT *
  FROM information_schema.table_constrains
  WHERE table_name = 'employees';
  ```

- **创建完毕后添加唯一性约束**

  ```SQL
  ALTER TABLE
  ADD CONSTRAINT <约束名> UNIQUE(字段名1,[字段名2,字段名3])
  
  ALTER TABLE <表名>
  MODIFY id INT UNIQUE; 
  ```

- **删除唯一性约束**

> 注意事项

![image-20220703142507872](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-16?token=AOAPFCPYTLEJJB5I5XH6J3TC57WSA)

![image-20220703142543349](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-17?token=AOAPFCI6XJGOJSLMYZMYCYDC57WS6)

### `PRIMARY KEY`主键约束

> `PRIMARY KEY`主键约束的效果相当于`UNIQUE`+`NOT NULL`
>
> 注意
>
> - **一个表只能有一个字段(或一组字段)设置为主键**
> - **有列级创建方法也有表级创建方法**
> - 主键约束也有**列级创建**方式与**表级创建**方式两种
>   - 即便使用表级创建自定义了主键约束名,也不会生效,其名字还是会被设置为`PRIMARY`
>   - 表级创建可以设置以一组字段为主键
> - **复合主键约束的判定问题**

- **创建时添加主键**

  ```SQL
  CREATE TABLE employees(
  id INT PRIMARY KEY
  );
  
  CREATE TABLE employees(
  id INT,
  PRIMARY KEY(id)
  );
  
  CREATE TABLE employees(
  id INT,
  PRIMARY KEY(id)
  );
  
  CREATE TABLE employees(
  id INT,
  CONSTRAINT <自定义约束名> PRIMARY KEY(id)#不会报错,但自定义约束名不会起作用
  );
  
  CREATE TABLE employees(
  id INT,
  department_id
  PRIMARY KEY(id,department_id)
  );
  ```

- **创建完毕后添加主键**

  ```SQL
  #添加列级主键约束
  ALTER TABLE employees
  MODIFY id INT PRIMARY KEY;
  
  #添加表级单列约束
  ALTER TABLE employees
  ADD PRIMARY KEY(id)
  
  #添加表级复合多列约束
  ALTER TABLE employees
  ADD PRIMARY KEY(id,department_id)
  ```

- **删除主键**(==这样子删除不会删掉其自带的非空约束==)

  ```SQL
  ALTER TABLE employees
  DROP PRIMARY KEY
  ```

### `FOREIGN KEY`外键约束

> **作用**:用于保证指定字段的**引用完整性**
>
> - 若某个字段需要作为其他表的外键,那么该字段必须要是下面两中类型之一,否则表创建会失败
>   - `PRIMARY KEY`
>   - `UNIQUE`
> - **若要删除主表,必须先删除其从表.若先删除主表是无法成功的**
> - **关联的两个字段可以不同名,但是其数据类型必须相同** 
> - **有表级创建与列级创建两种方式**
>   - 表级创建可给外键约束自定义名字,列级约束不可以自定义.
>   - ==当没有自定义约束名时,其约束名不再是类似于`UNIQUE`的自动设置为其所属的字段的名字,而是会以一种特殊的方法来自动起名==
> - **删除外键约束后必须手动删除其外键索引**
> - ==**约束等级中父表指的是被引用的表,而子表指的是引用其他表的表**==

- **创建时设置字段的外键**

  ```SQL
  CREATE TABLE departments(
  dept_id INT PRIMARY KEY,
  department_name VARCHAR(25)
  ); 
  
  #表级创建方法
  CREATE TABLE employees(
  employee_id INT PRIMARY KEY AUTO_INCREMENT,
  last_name VARCHAR(25),
  department_id INT,
  CONSTRAINT foreign_cons FOREIGN KEY(department_id) REFERENCES(departments(dept_id)) 
  );
  #不自定义约束名的表级创建
  CREATE TABLE employees(
  employee_id INT PRIMARY KEY AUTO_INCREMENT,
  last_name VARCHAR(25),
  department_id INT,
  FOREIGN KEY(department_id) REFERENCES(departments(dept_id)) 
  );
  
  
  #列级创建方法
  CREATE TABLE employees(
  employee_id INT PRIMARY KEY AUTO_INCREMENT,
  last_name VARCHAR(25),
  department_id INT FOREIGN KEY REFERENCES departments(dept_id)
  );
  ```

- **创建完成后添加外键约束**

  ```SQL
  #添加列级外键约束
  ALTER TABLE employees
  MODIFY department_id INT FOREIGN KEY REFERENCES departments(dept_id);
  
  #添加表级外键约束
  ALTER TABLE employees
  ADD CONSTRAINT foreign_cons FOREIGN KEY(department_id) REFERENCES(departments(dept_id));
  ```

- **删除外键约束**

  ```SQL
  ALTER TABLE employees
  DROP FOREIGN KEY foreign_cons;
  
  ALTER TABLE employees
  DROP INDEX foreign_cons;
  ```

#### 约束等级(默认为`Restrict`)

> 假设`表A`的`字段A`引用了`表B`的`字段B`,显然`表B`为父表,`表A`为子表

> - `Cascade`:在父表上`update/delete`时,同步`update/delete`子表的匹配记录
>   - 当我们`UPDATE`表B的字段B中取值40的记录为60时,那么表A中所有字段A的值为40的记录的字段A的值都会被修改为60
>   - 当我们`DELETE`表B中的字段B取值为40的记录时,那么表A中所有的字段A的值为40的记录都将被`DELETE`
> - `Set NULL`:在父表上`update/delete`时,同步将子表上匹配的记录的对应字段(即**设置了改外键的字段**)设置为`NULL`,但要注意子表的外键字段不能为`NOT NULL`
>   - 当我们`UPDATE`表B的字段B中取值40的记录为60时,那么表A中所有字段A的值为40的记录的字段A的值都会被修改为`NULL`
>   - 当我们`DELETE`表B中的字段B取值为40的记录时,那么表A中所有的字段A的值为40的记录都将被`NULL`
> - `No action`:如果子表中有匹配的记录,那么不允许父表对应候选键进行`update/delete`操作
> - `Restrict`:同`No action`
> - `Set default`:父表有变更时,子表将外键列设置为一个默认值,但`InnoDB`引擎不能识别

- **设置约束等级**

  ```SQL
  CREATE TABLE departments(
  dept_id INT PRIMARY KEY,
  department_name VARCHAR(25)
  ); 
  
  #设置UPDATE时的的约束等级为Cascade,而DELETE时的约束等级为Set NULL
  CREATE TABLE employees(
  employee_id INT PRIMARY KEY AUTO_INCREMENT,
  last_name VARCHAR(25),
  department_id INT,
  FOREIGN KEY(department_id) 
  REFERENCES(departments(dept_id))
  ON UPDATE CASCADE
  ON DELETE SET NULL
  );
  ```

#### `CHECK`检查约束(`MySQL 5.7`不支持)

> 作用:保证``域完整性``,`用户自定义完整性`
>
> **注意事项**

- **创建时设置检查约束**

  ```SQL
  CREATE TABLE departments(
  dept_id INT PRIMARY KEY,
  department_name VARCHAR(25),
  avg_salary DECIMAL(10,3) CHECK(avg_salary>1500)
  ); 
  CREATE TABLE departments(
  dept_id INT PRIMARY KEY,
  department_name VARCHAR(25),
  avg_salary DECIMAL(10,3) CHECK(avg_salary>1500 AND avg_salary<15000)
  ); 
  
  CREATE TABLE departments(
  dept_id INT PRIMARY KEY,
  department_name VARCHAR(25),
  avg_salary DECIMAL(10,3) CHECK(avg_salary in (10000,5000,4000))
  ); 
  ```

- **创建后添加检查约束**

  ```SQL
  ALTER TABLE departments
  MODIFY avg_salary DECIMAL(10,3) CHECK(avg_salary>1500);
  ```

- **删除检查约束**

  ```SQL
  ALTER TABLE departments
  MODIFY avg_salary DECIMAL(10,3);
  ```

#### `DEFUALT`默认值约束

> 作用:当存入的记录没有给出该字段的值时以默认值存入,而非`NULL`

- **创建时设置默认值约束**

  ```SQL
  CREATE TABLE departments(
  dept_id INT PRIMARY KEY,
  department_name VARCHAR(25),
  avg_salary DECIMAL(10,3) DEFAULT 0
  ); 
  ```

- **创建完成后添加默认值约束**

  ```SQL
  ALTER TABLE departments
  MODIFY avg_salary DECIMAL(10,3) DEFAULT 0;
  ```

- **删除默认值约束**

  ```SQL
  ALTER TABLE departments
  MODIFY avg_salary DECIMAL(10,3);
  ```

## ==**`MySQL`的`COMMIT`与`ROLLBACK`**==

### **`COMMIT`**

> **提交数据。一旦`COMMIT`指令被执行,所有的数据就会被永久地保存到数据库中,此时该数据便不再可回滚**
>
> - **例如我们修改了数据表中的某一个记录的某个字段的值.**
>   - **在修改完成之后,`COMMIT`指令执行之前,磁盘中该记录依然为未修改前的记录,而内存中的该记录则是修改后的.**
>   - **当`COMMIT`指令执行之后,内存中的修改后的记录就会保存到磁盘上,而原本未修改的记录就会被覆盖.到这一步,我们便无法通过任何办法获取到修改前的记录.**

### **`ROLLBACK`**

> **回滚数据。一旦`ROLLBACK`指令被执行,那么数据库就会回到最近的一次`COMMIT`指令之后的状态**
>
> - **我们可以这样理解`ROLLBACK`指令.**
> - **我们的`MySQL`会把磁盘中保存的数据库读取到内存中来供应我们操作,因此我们操作的是内存上的数据库,而不是磁盘上的数据库.**
> - **磁盘上的数据库与内存上的数据库是不同步的.而`COMMIT`指令就是用于将内存中的数据库,覆盖到磁盘上的.**
> - **而我们的`ROLLBACK`则可以理解为,当前我们内存上的数据库发生了错误,我们首先会把内存上的数据库删除掉,然后我们会读取磁盘中的数据库到内存中.然后我们来操作内存上的数据库.**
> - **这也就解释了为什么`ROLLBACK`只能回滚到最近的一次`COMMIT`之后(因为磁盘中保存到数据库一定是最后一次`COMMIT`之后的数据)** 

## **`MySQL`增删改**

### **增**

#### **直接插入数据**

- **方式1**

  ```SQL
  INSERT INTO <表名>
  VALUES(字段1值,字段2值,字段3值...);
  
  INSERT INTO <表名>
  VALUES
  (字段1值,字段2值,字段3值...),
  (字段1值,字段2值,字段3值...),
  (字段1值,字段2值,字段3值...);
  ```

- **方式2**

  ```SQL
  INSERT INTO <表名>(字段2,字段1,字段4,字段3...)
  VALUES
  (字段2值,字段1值,字段4值,字段3值...);
  
  INSERT INTO <表名>(字段2,字段1,字段4,字段3...)
  VALUES
  (字段2值,字段1值,字段4值,字段3值...),
  (字段2值,字段1值,字段4值,字段3值...),
  (字段2值,字段1值,字段4值,字段3值...);
  
  #若表中共有3个字段且字段3可以为空那么下面的添加是合法的
  INSERT INTO <表名>(字段2,字段1)
  VALUES
  (字段2值,字段1值),
  (字段2值,字段1值),
  (字段2值,字段1值);
  ```

#### **将查询结果插入表**

> **注意:查询结果插入表由于被查询的表与被插入的表的字段的数据类型不同可能会引发一些问题,因此我们必须遵守下面的规则,如果不成功则有插入失败或错误的风险**
>
> - **查询表中要插入的字段应该保证其数据类型与被插入表对应字段数据类型相同,并且保证长度小于等于被插入表中对应的字段.**

- ```SQL
  INSERT INTO <表名>
  SELECT *
  FROM eployees;
  
  INSERT INTO <表名>(employee_id,last_name,salary)
  SELECT employee_id,last_name,salary
  FROM eployees;
  ```

### **删**

> **删除记录也可能由于各种原因失败**

- ```SQL
  DELETE FROM <表名>
  WHERE 记录过滤条件
  ```

### **改**

> **注意:修改数据时由于对应的字段的取值的限制可能会发生修改不成功的情况的**
>
> **问题**
>
> - **数据表的修改为`DML`操作,按道理是不具备原子性的,那么对于`SET 字段1=修改后的值,字段2=修改后的值`这样的多字段修改,当``字段1``能够正确修改,而``字段2``不能正确修改时,是否会自动触发`ROLLBACK`指令?**

- ```SQL
  UPDATE <表名>
  SET 字段1=修改后的值;
  
  UPDATE <表名>
  SET 字段1=修改后的值
  WHERE 记录过滤条件;
  
  UPDATE <表名>
  SET 字段1=修改后的值,字段2=修改后的值,字段3=修改后的值
  WHERE 记录过滤条件;
  
  UPDATE <表名>
  SET 字段3=修改后的值,字段1=修改后的值,字段2=修改后的值
  WHERE 记录过滤条件;
  ```

# 第四部分

## 常见的数据库对象

![image-20220703211822443](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-18?token=AOAPFCLSL5ALS7UIOKSM6ELC57WUE)

## 视图

> **作用**:以一种针对性的方式整合我们所有基表上的信息,向不同的用户展示.相当于对基表信息的**引用**
>
> **视图的特性**
>
> - 视图是一个``虚拟表``,其内部并不存储任何数据
> - **我们对视图的字段进行的**==**增删改**==**会同时作用到其涉及的基表**(**仅限直接关联**,**像由基表聚合函数生成的字段(间接关联)就不会作用到基表)**
> - **我们对基表做出的**==**增删改**==**也会作用到视图上**
> - 创建视图时不用对字段进行除命名以外的任何设置(如数据类型,非空约束,默认值约束等)
> - **删除视图本身并不会对基表产生影响**
> - ==**删除视图中的记录有可能对基表产生影响**==
> - **视图可以像数据表一样被查询**
> - **我们可以基于视图创建视图**==**但是如果用于创建的视图被删除了,那么我们创建出来的视图也会同时失效,但系统不会自动删除掉,需要我们手动删除.如果不自己删除则会一直保留下去**==
> - ==**可以根据视图创建数据表**== 
>
> **视图的优点**
>
> - **设置数据表的访问权限 **
>
> **视图的缺点**
>
> - **如果用于创建视图的基表的字段名等发生了修改,那么我们就必须考虑修改该视图**

- **通过在`CREATE VIEW`中嵌入子查询来创建视图**

  ```SQL
  #规范形式
  CREATE [OR REPLACE]
  [ALGORITHM] = {UNDIFINED|MERGE|TEMPTABLE}]
  VIEW 视图名称[(字段列表)]
  AS 子查询语句
  [WITH [CASCADED|LOCAL] CHECK OPTION]
  
  #实例1
  CREATE VIEW <视图名>
  AS 
  (SELECT employee_id,last_name
   FROM employees);
   
   #实例2
  CREATE VIEW <视图名>
  AS 
  (SELECT employee_id e_id,last_name l_name
   FROM employees);
   
   #实例3
  CREATE VIEW <视图名>(ee_id,ll_name)
  AS 
  (SELECT employee_id,last_name
   FROM employees);
   
   #实例4
  CREATE VIEW <视图名>(ee_id,ll_name)
  AS 
  (SELECT employee_id e_id,last_name l_name
   FROM employees);
   
   #实例5
   CREATE VIEW <视图名>
   AS
   (SELECT department_id,AVG(salary) avg_salary
   FROM employees
   WHERE department_id <=> NULL
   GROUP BY department_id);
  ```

- **ALGORITHM**

  - `UNDIFUNED`:
  - `MERGE`:
  - `TEMPTABLE`:

- **WITH**
  - `CASCADED`:
  - `LOCAL`
- **CHECK OPTION**

### 视图管理

- **修改视图**(==视图的修改只能以一种重建的方式进行,不具有数据表的那些修改方法==)

  ```SQL
  #方式1
  ALTER VIEW
  [ALGORITHM] = {UNDIFINED|MERGE|TEMPTABLE}]
  VIEW 已经存在的视图的名字[(字段列表)]
  AS 子查询语句
  [WITH [CASCADED|LOCAL] CHECK OPTION]
  
  #方式2
  CREATE OR REPLACE
  [ALGORITHM] = {UNDIFINED|MERGE|TEMPTABLE}]
  VIEW 已经存在的视图的名字[(字段列表)]
  AS 子查询语句
  [WITH [CASCADED|LOCAL] CHECK OPTION]
  ```

- **删除视图**

  ```SQL
  DROP VIEW <视图名>;
  DROP VIEW <视图1名>,<视图2名>;
  
  DROP VIEW IF EXISTS <视图1名>;
  DROP VIEW IF EXISTS <视图1名>,<视图2名>;
  ```

- **查看视图**

  ```SQL
  SHOW TABLES;
  ```

- **查看视图结构**(==各个字段的情况==)

  ```SQL
  DESC <视图名>;
  ```

- **查看视图的属性信息**(==使用的引擎,字符集,记录数,创建时间等==)

  ```SQL
  SHOW TABLE STATUS LIKE '<视图名>';
  ```

- **查看视图的创建信息**

  ```SQL
  SHOW CREATE VIEW 视图名
  ```

### 视图的增删改

> **注意**:对视图进行增删改操作可能会作用到基表上，也可能不会作用到。有些情况下生成的视图是不被允许进行增删改的

- **修改数据**

  ```SQL
  UPDATE <表名>
  SET salary = 5000
  WHERE department_id = 66;
  ```

- **删除数据**

  ```SQL
  DELETE FROM <视图名>
  WHERE employee_id = 51;
  ```

- **添加数据**

  ```SQL
  INSERT INTO 视图名
  VALUES(<字段1值>,<字段2值>...);
  ```

### **不可更新**

![image-20220703232052382](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-19?token=AOAPFCIZMHZ77NEHTUBXB4DC57WVC)

### 视图的优点

![image-20220703233958096](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-20?token=AOAPFCPQNBFB2BUVEY7OJO3C57WVQ)

### 视图的缺点

![image-20220703234029579](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-21?token=AOAPFCKYLOGJ2R4MQMCJMDTC57WWE)

## 存储过程

> 存储过程实际上就是一组经过``预先编译``的`SQL`语句的封装.其**会存储在我们的主机的`MySQL`服务器上**,当我们**需要用**到该存储过程时,就可以**向`MySQL`服务器发起调用请求**,我们的**`MySQL`服务器就会找到我们指定的存储过程,并执行其封装好的`SQL`语句**
>
> **注意事项**
>
> - ==**存储过程没有返回值**==

### 使用`SET`来设置变量并初始化

- **设置+初始化**

  ```SQL
  SET @<变量名>= <值>;
  ```

### 存储过程的分类

- **无参数无返回**

  ```SQL
  CREATE PROCEDURE 存储过程名()
  [CHARACTERISTICS]
  BEGIN
  	存储过程语句体
  END
  ```

- **`IN`类型,有参数无返回**

  ```SQL
  CREATE PROCEDURE 存储过程名(IN 参数1名 数据类型,参数2名 数据类型...)
  [CHARACTERISTICS]
  BEGIN
  	存储过程语句体
  END
  ```

- **`OUT`类型,无参数有返回**

  ```SQL
  CREATE PROCEDURE 存储过程名(OUT)
  [CHARACTERISTICS]
  BEGIN
  	存储过程语句体
  END
  ```

- **`IN OUT`类型,有参数有返回(参数与返回值不共变量)**

  ```SQL
  CREATE PROCEDURE 存储过程名(IN 参数1名 数据类型,参数2名 数据类型...,OUT 参数3名 数据类型,参数4名 数据类型...)
  [CHARACTERISTICS]
  BEGIN
  	存储过程语句体
  END
  ```

- **`INOUT`类型,有参数有返回(参数与返回值共变量)**

  ```SQL
  CREATE PROCEDURE 存储过程名(INOUT 参数1名 数据类型,参数2名 数据类型...)
  [CHARACTERISTICS]
  BEGIN
  	存储过程语句体
  END
  ```

### `CHARACTERISTICS`选项的组成

```SQL
LANGUAGE SQL
| [NOT] DETERMINISTIC
| {CONTAINS SQL|NO SQL|READS SQL DATA|MODIFIES SQL DATA}
| SQL SECURITY{DEFINER|INVOKER}
| COMMENT '<字符串>'
```

- `LANGUAGE SQL`:说明存储过程的语句体是由`SQL`语句组成,当前系统支持的语言为`SQL`
- `[NOT] DETERMINISTIC`:默认为`NOT DETERMINISTIC`指明存储过程执行的结果是否确定
  - `DETERMINISTIC`:相同的输入一定有相同的输出
  - `NOT DETERMINISTIC`:相同的输入不一定有相同的输出
- `{CONTAINS SQL|NO SQL|READS SQL DATA|MODIFIES SQL DATA}`
  - 默认取为`CONTAINS SQL`
  - `CONTAINS SQL`:当前存储过程语句体中包含`SQL`语句,但并不包含涉及到读写操作的`SQL`语句
  - `NO SQL`:表示存储过程语句体不包含`SQL`语句
  - `READS SQL DATA`:表示存储过程语句体包含涉及到读操作的`SQL`语句
  - `MODIFIES SQL DATA`:表示存储过程语句体包含涉及到写操作的`SQL`语句
- `SQL SECURITY{DEFINER|INVOKER}`
  - `DEFINER`:表示该存储过程只能被其创建者(定义者)使用
  - `INVOKER`:表示该存储过程可以被具有访问权限的用户使用
- `COMMENT '<字符串>'`:注释

### 存储过程语句体的编写

> **前提**
>
> - 由于`MySQL`默认使用`;`作为结束标识符,因此如果我们直接编写存储过程,则会由于存储过程语句体中出现`;`而导致存储过程提前终止运行,因此**我们必须通过`DELIMITER`**来修改`MySQL`系统的默认结束标识符
>
>   ```SQL
>   #将终止标识符改为$
>   DELIMITER $
>                           
>   CREATE PROCEDURE 存储过程名()
>   [CHARACTERISTICS]
>   BEGIN
>   	存储过程语句体
>   END
>                           
>   #将终止标识符改回;
>   DELIMITER ;
>   ```

- **创建存储过程**

  ```SQL
  #准备工作
  CREATE DATABASE TEST2022;
  USE TEST2022;
  
  CREATE TABLE employees
  AS (SELECT * FROM atguigudb.'employees')
  
  CREATE TABLE departments
  AS (SELECT * FROM atguigudb.'departments')
  ```

  - **无参数无返回**

    ```SQL
    #创建存储过程查看employees表中的全部数据
    DELIMITER $
    CREATE PROCEDURE Process()
    BEGIN
    	SELECT * FROM employees;
    END $
    DELIMITER ;
    
    #创建存储过程查看employees表中所有员工的平均工资
    DELIMITER $
    CREATE PROCEDURE avg_Process()
    BEGIN
    	SELECT AVG(salary) avg_salary FROM employees;
    END $
    DELIMITER ;
    ```

  - **`IN`类型**

    ```SQL
    #创建存储过程,根据输入的employee_id查找出对应的员工的last_name与salary
    DELIMITER $
    CREATE PROCEDURE select_Process(IN emp_id INT)
    BEGIN
    	SELECT last_name,salary FROM employees WHERE employee_id<=>emp_id;
    END $
    DELIMITER ;
    ```

  - **`OUT`类型**

    ```SQL
    #创建存储过程查看employees表中所有员工的平均工资,并存储到变量ms中
    DELIMITER $
    CREATE PROCEDURE avg_Process(OUT ms DOUBLE)
    BEGIN
    	SELECT AVG(salary) INTO ms FROM employees;
    END $
    DELIMITER ;
    ```

  - **`IN OUT`类型**

    ```SQL
    #按照输入的emp_id(员工employee_id)查找到对应员工的salary并用emp_salary变量接收
    DELIMITER $
    CREATE PROCEDURE in_out_Process(IN emp_id INT,OUT emp_salary DECIMAL(10,2))
    BEGIN
    	SELECT salary INTO emp_salary FROM employees WHERE employee_id<=>emp_id;
    END $
    DELIMITER ;
    ```

  - **`INOUT`类型**

    ```SQL
    #按照输入的emp_id(员工employee_id)查找到对应员工的部门领导,并重用emp_id变量接收改领导的last_name
    #不同类型不知道可不可以
    DELIMITER $
    CREATE PROCEDURE inout_Process(INOUT emp_id INT)
    BEGIN
    	SELECT last_name INTO emp_id
    	FROM employees
    	WHERE employee_id <=> (SELECT manager_id
                               FROM employees
                               WHERE employee_id <=> emp_id);
    END $
    DELIMITER ;
    
    #按照输入的emp_name(员工last_name)查找到对应员工的部门领导,并重用emp_name变量接收改领导的last_name
    #同类型是可以的
    DELIMITER $
    CREATE PROCEDURE inout_Process(INOUT emp_name VARCHAR(25))
    BEGIN
    	SELECT last_name INTO emp_name
    	FROM employees
    	WHERE employee_id <=> (SELECT manager_id
                               FROM employees
                               WHERE last_name <=> emp_name);
    END $
    DELIMITER ;
    ```

- **调用存储过程**

  - **无参数无返回**

  ```SQL
  CALL Process();	
  CALL avg_Process();
  ```

  - **`IN`类型**

    ```SQL
    #传入常量
    CALL select_Process(113);
    
    #传入变量
    CALL select_Process(@emp_id);
    
    #变量名不一定要为emp_id,可以任意选取,只要变量中存储有对应的数据即可
    CALL select_Process(@empppp_id);
    ```

  - **`OUT`类型**

    ```SQL
    #调用
    CALL avg_Process(@ms);
    
    #查看ms的返回值
    SELECT @ms;
    
    #变量名不一定要为emp_id,可以任意选取,只要变量中存储有对应的数据即可
    CALL avg_Process(@fasfa);
    
    #查看fasfa的返回值
    SELECT @fasfa;
    ```

  - **`IN OUT`类型**

    ```SQL
    #调用
    CALL in_out_Process(@emp_id,@emp_salary);
    #查看emp_salary中保存到结果
    SELECT @emp_salary;
    ```

  - **`INOUT`类型**

    ```SQL
    #调用
    CALL inout_Process(@emp_name)
    #查看emp_name中保存的结果
    SELECT @emp_name
    ```

- **删除存储过程**

  ```SQL
  DROP PROCEDURE 存储过程名;
  ```

## 存储函数

> 存储函数与存储过程的区别
>
> - 存储过程==**可以有返回值(可以是多个也可以是一个(一个具体的值或变量.不能是一个列表,一个列))**==,也**可以没有返回值**
> - 存储函数==**一定有返回值(一定要有`RETURN`子句),并且返回值只能有一个(一个具体的值或变量.不能是一个列表,一个列)**==

### 创建存储函数

> - 在存储过程中`characteristic`不做指定是没有问题的,因为系统会自动添加默认值
> - **在存储函数中`characteristic`是必须由我们自己指定的**
>   - 当然有直接设置系统默认的解决方式,但不推荐使用

- **标准格式**

  ```SQL
  #标准格式
  DELIMITER $
  CREATE FUNCTION 函数名(参数列表)
  RETURN 返回值类型1[,返回值类型2...]
  [characteristic]
  BEGIN
  	RETURN (存储函数语句体)
  END$
  DELIMITER ;
  ```

### 调用存储函数

> - **存储函数可以当作单行函数或者聚合函数使用**
>
> - **存储函其实就是由自定义的单行函数或聚合函数**

```SQL
SELECT 存储函数名(传入参数);
```

### 删除存储函数

```SQL
DROP FUNCTION 存储函数名;
```

## 修改存储过程与存储函数

## 存储函数与存储过程的管理

- **存储过程与存储函数的创建信息查看**

  ```SQL
  SHOW CREATE PROCEDURE 存储过程名;
  SHOW CREATE FUNCTION 存储函数名;
  ```

- **存储过程与存储函数的简要属性信息查看**

  ```SQL
  SHOW PROCEDURE STATUS LIKE '存储过程名';
  SHOW FUNCTION STATUS LIKE '存储函数名';
  ```

- **存储过程与存储函数的详细属性信息查看**

  ```SQL
  SELECT *
  FROM information_schema.Routines
  WHERE ROUTINE_NAME = '存储函数名'
  
  SELECT *
  FROM information_schema.Routines
  WHERE ROUTINE_NAME = '存储过程名'
  
  #以下两个用于存储函数与存储过程重名时使用
  SELECT *
  FROM information_schema.Routines
  WHERE ROUTINE_NAME = '存储函数名' AND ROUTINE_TYPE='FUNCTION'
  
  SELECT *
  FROM information_schema.Routines
  WHERE ROUTINE_NAME = '存储过程名' AND ROUTINE_TYPE='PROCEDURE'
  ```

### 存储过程与存储函数对比

![image-20220704142044462](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-22?token=AOAPFCM3RZ47DWYRH5XWGK3C57WXU)

### 存储过程的优缺点

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-232?token=AOAPFCIHJ7OP7HUCK7CV7RDC57WYI" alt="image-20220704143742175" style="dispaly:flex;zoom: 67%;" /><img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\基础知识汇总_Windows.assets\image-20220704143752942.png" alt="image-20220704143752942" style="zoom:67%;" />

## 变量

> **变量只能接收一个任何数据类型的值,不能接收一列,一个列表的数据**

> **变量一般用于`MySQL`中来对存储过程与存储函数的查询或计算的结果或最终的输出进行存储.**

### 系统变量(`GLOBAL,SESSION`,`@@`)

> **注意**:

> **前提**
>
> - 我们可以**把`MySQL`服务看作一个类**,这个**类中有成员函数与成员变量**.而成员函数就是一个一个的会话,成员变量就是这个类的全局变量(可以被每一个有权限使用的成员函数所共享).
>
> - 我们还应该知道每一个**成员函数中都会有该函数自己的变量**,这些变量被称为该**成员函数的本地变量**.显然这些本地变量只能被他们所属的成员函数自身直接访问.
>
> - 再者我们还应该知道,**成员函数的本地变量是可以和类的成员变量重名的**.

#### 全局系统变量(`GLOBAL`)

> **全局系统变量**就可以**类比为**我们所说的**类的成员变量**
>
> **注意**
>
> - 全局系统变量**可以跨会话**,但**不能跨`MySQL`服务的重启**

#### 查询

- **查看所有系统变量**

  ```SQL
  SHOW GLOBAL VARIABLES;
  ```

- **查看名称符合条件的全局系统变量**

  ```SQL
  SHOW GLOBAL VARIABLES LIKE '<模糊匹配式>';
  ```

- **查看指定全局系统变量**

  ```SQL
  SELECT @@global.全局系统变量名1
  ```

#### 修改

> **注意**
>
> - **第一种方式是永久生效的,而第二种方法只要我们重启`MySQL`服务器就会失效**

- **直接通过修改`my.ini`配置文件的方式来修改.但是每次修改都必须重启`MySQL`服务器**

- **使用`SET`在`MySQL`服务器运行时修改**

  ```SQL
  SET @@global.全局系统变量名 = 变量值;
  SET GLOBAL 全局系统变量名 = 变量值;
  ```

#### 会话系统变量(``SESSION``)

> **会话系统变量会在一个会话被开启时有`MySQL`服务器自动创建,每一个会话都有其自己的会话系统变量,当会话关闭时,其关联的会话系统变量就会被**
>
> **会话系统变量**就可以**类比为**我们所说的**成员函数的本地变量**
>
> ==**注意**==
>
> - 成员函数本地变量可以与类的成员变量同名,**同样的会话系统变量也可以与全局系统变量重名**
> - 当重名时,我们可以在使用时做出声明,**让系统知道我们调用的是全局系统变量还是会话系统变量即可**
> - 会话系统变量显然**无法跨会话**

#### 查询

- **查看所有会话系统变量**

  ```SQL
  SHOW SESSION VARIABLES;
  或
  SHOW VARIABLES;
  ```

- **查看名称满足一定条件的会话系统变量**

  ```SQL
  SHOW SESSION VARIABLES LIKE '<模糊匹配式>';
  或
  SHOW VARIABLES LIKE '<模糊匹配式>';
  ```

- **查看指定会话系统变量**

  ```SQL
  SELECT @@session.<会话系统变量名>;
  ```

#### 修改

> **注意**
>
> - **第一种方式是永久生效的,而第二种方法只要我们切换会话就会失效**

- **直接通过修改`my.ini`配置文件的方式来修改.但是每次修改都必须重启`MySQL`服务器**

- **使用`SET`在`MySQL`服务器运行时修改**

  ```SQL
  SET @@session.全局系统变量名 = 变量值;
  SET SESSION 全局系统变量名 = 变量值;
  ```

### 用户自定义变量(`@`)

#### 会话用户变量

> - **会话用户变量使用时必须带``@``**
> - **会话用户变量可以在`SQL`中直接创建,也可以在存储函数,存储过程的`BEGIN`,``END`之间创建(在此时需要使用`DCLARE`做声明)**
> - **会话用户变量不用事先声明**

> - **作用域与会话系统变量相同.只不过其不是系统自带的,而是由用户自己创建的**.
> - 会话用户变量可以被用户直接收回,或者等到会话结束由系统自动收回

- **查看会话用户变量**

  ```SQL
  SELECT @会话用户变量名
  ```

- **创建会话用户变量**

  ```SQL
  SET @会话用户变量名 = 值;
  SET @会话用户变量名 := 值;
  SELECT @会话用户变量 := 表达式 [FROM 子句];
  SELECT 表达式 INTO @会话用户变量 [FROM子句];
  #1 2
  SET @a = 1;
  SET @b = 3;
  SET @c = @a+@b;
  SET @d := 5;
  SET @e = @d+@c+@a;
  SET @e := @d+@c+@a;
  
  #3 4
  #获取employees表的记录数并保存到@rows会话用户变量中
  SELECT @rows := COUNT(*) FROM employees;
  SELECT COUNT(*) INTO @rows FROM employees;
  
  SELECT salary_Abel := salary FROM employees WHERE last_name = 'Abel';
  SELECT salary INTO salary_Abel FROM employees WHERE last_name = 'Abel';
  
  #无意义,但不报错能正确运行的
  SELECT @rows := 'abc' FROM employees;
  SELECT 'abc' INTO @rows FROM employees;
  ```

- **删除会话用户变量**

- **修改会话用户变量**

#### 局部变量

> 当局部变量未初始化时,其值为`NULL`

> **局部变量的定义必须要在`BEGIN`与`END`之间初始化,且必须紧跟`BEGIN`使用`DECLARE`对其进行声明,中间不能有别的语句,且只要声明之后才可以初始化**
>
> - **起始也就相当于局部变量必须在函数里面先用`DECLARE`声明,再使用`SET`初始化,并不能声明的同时初始化**

> - 局部变量只能在存储过程或存储函数的`BEGIN`与`END`之间使用,并且其作用域也仅在`BEGIN`与`END`之间.
>
> - 我们可以这样理解,将存储过程或存储函数理解成`Python`的一个函数,而局部变量就是这个函数的局部变量.

- **查看局部变量**

  ```SQL
  SELECT 局部变量名;
  ```

- **创建局部变量**

  ```SQL
  #示例
  SET 局部变量名1 = 值;
  SET 局部变量名2 := 值;
  SELECT 表达式 INTO 局部变量名3 [FROM子句]
  
  #完整版示例1
  DELIMITER $
  CREATE PROCEDURE()
  BEGIN
  
  DECLARE 局部变量名1 数据类型 [DEFAULT 值];
  DECLARE 局部变量名3 数据类型 [DEFAULT 值];
  DECLARE 局部变量名2 数据类型 [DEFAULT 值];
  
  SET 局部变量名1 = 值;
  SET 局部变量名2 := 值;
  SELECT 表达式 INTO 局部变量名3 [FROM子句]
  
  END $
  DELIMITER ;
  
  #完整版示例2
  DELIMITER $
  CREATE FUNCTION()
  [RETURN类型声明子句]
  [CHARACTERISTIC]
  BEGIN
  
  DECLARE 局部变量名1 数据类型 [DEFAULT 值];
  DECLARE 局部变量名3 数据类型 [DEFAULT 值];
  DECLARE 局部变量名2 数据类型 [DEFAULT 值];
  
  SET 局部变量名 = 值;
  SET 局部变量名 := 值;
  SELECT 表达式 INTO 局部变量名 [FROM子句]
  [RETURN子句]
  END $
  DELIMITER ;
  ```

## 定义条件与处理程序

> **作用**:程序出错处理

### 应用实例

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-24?token=AOAPFCPKEI3BS6DRE4NIK6TC57WZO" alt="image-20220704190901050" style="zoom:50%;" />

### 错误码与错误条件

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%B1%87%E6%80%BBWindows-25?token=AOAPFCPHLJPVXJRQO2AYKWTC57WZ6" alt="image-20220704191216850" style="zoom:80%;" />

- **错误码`MySQL_error_code`**:数值类型错误代码
- **错误条件`sqlstate_value`**:字符串类型错误码

### 定义条件

> **作用**:给一个``错误代码+错误条件`起一个`错误名字`

- **标准格式**

  ```SQL
  DECLARE 错误名字 CONDITION FOR 错误代码(或错误条件)
  ```

- **使用示例**

  ```SQL
  #使用错误代码
  DECLARE NULL_COL CONDITION FOR 1048;
  
  #使用错误条件
  DECLARE NULL_COL CONDITION FOR '23000';
  ```

### 处理程序

> 处理程序可以**用于`SQL`文件**也可以用于**存储过程**或**存储函数**

> **标准格式**
>
> ```SQL
> DECLARE 处理方式 HANDLER FOR 错误类型 处理语句
> ```

#### 处理方式

- **`CONTINUE`**:遇到错误不终止,运行处理语句后继续执行后续的语句
- **`EXIT`**:遇到错误后,运行处理语句完毕后直接退出,不再执行后续语句
- **`UNDO`**:遇到错误后撤回错误之前做的所有操作,然后运行处理语句,然后直接退出,不再执行后续语句,==**`MySQL`不支持**==
- **对比**
  - **默认情况下,`MySQL`在遇到错误后的处理类似于`EXIT`,只不过不会执行任何处理语句**
  - **`UNDO`相对于`EXIT`而言多出来撤回之前的所有操作的处理.(例如当我们同时更新两个字段时,如果第一个字段更新成功,第二个字段更新失败.那么`EXIT`下第一个字段会变成修改后的,而第二个字段不会.而`UNDO`下第一个字段与第二个字段都会是修改前的值)**

#### 错误类型

- **`SQLSTATE '错误条件'`:`sqlstate_value`的字符串类型错误代码**
- **`MySQL_error_code类型的错误代码`:数值类型错误代码**
- **`错误名称`:使用`DECLARE CONDITION FOR`定义的错误名称**
- **`SQLWARNING`:所有以`01`开头的`sqlstate_value`类型错误代码**
- **`NOT FOUND`:所有以`02`开头的`sqlstate_value`类型错误代码**
- **`SQLEXCEPTION`:所有除`01|02`开头的`sqlstate_value`类型错误代码**

#### 处理语句

#### 示例

- **简要**

  ```SQL
  #方法1：捕获sqlstate_value
  DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02' SET @info = 'NO_SUCH_TABLE';
  
  #方法2：捕获mysql_error_value
  DECLARE CONTINUE HANDLER FOR 1146 SET @info = 'NO_SUCH_TABLE';
  
  #方法3：先定义条件，再调用
  DECLARE no_such_table CONDITION FOR 1146;
  DECLARE CONTINUE HANDLER FOR NO_SUCH_TABLE SET @info = 'NO_SUCH_TABLE';
  
  #方法4：使用SQLWARNING
  DECLARE EXIT HANDLER FOR SQLWARNING SET @info = 'ERROR';
  
  #方法5：使用NOT FOUND
  DECLARE EXIT HANDLER FOR NOT FOUND SET @info = 'NO_SUCH_TABLE';
  
  #方法6：使用SQLEXCEPTION
  DECLARE EXIT HANDLER FOR SQLEXCEPTION SET @info = 'ERROR';
  ```

- **具体案例**

  ```SQL
  DELIMITER //
  CREATE PROCEDURE UpdateDataNoCondition()
  BEGIN
  #定义处理程序
  	DECLARE CONTINUE HANDLER FOR 1048 SET @proc_value = -1;
  #该处理程序使得该存储过程遇到1048的错误代码时会将创建@proc_value会话用户变量并赋值为-1.且存储过程会继续执行不会直接中断退出
  
  	SET @x = 1;
  	UPDATE employees SET email = NULL WHERE last_name = 'Abel';
  	SET @x = 2;
  	UPDATE employees SET email = 'aabbel' WHERE last_name = 'Abel';
  	SET @x = 3;
  END //
  DELIMITER ;
  ```

## 流程控制

### 三大流程

- **顺序结构**:从上到小依次执行

- **分支结构**:按条件在多条路径中选一条执行

- **循环结构**:重复多次执行同一路径

### 三大控制语句

#### **条件判断语句(`IF|CASE`)**

##### `IF`语句

```mysql
IF 表达式1 THEN 操作1;
[ELSEIF 表达式2 THEN 操作2;]
[ELSEIF 表达式3 THEN 操作3;]
[ELSE 操作N;]
END IF;

#声明存储过程“update_salary_by_eid2”，定义IN参数emp_id，输入员工编号。判断该员工薪资如果低于9000元并且入职时间超过5年，就涨薪500元；否则就涨薪100元。
DELIMITER //
CREATE PROCEDURE update_salary_by_eid2(IN emp_id INT)
BEGIN
	DECLARE emp_salary DOUBLE;
	DECLARE hire_year DOUBLE;
	SELECT salary INTO emp_salary FROM employees WHERE employee_id = emp_id;
	SELECT DATEDIFF(CURDATE(),hire_date)/365 INTO hire_year
	FROM employees WHERE employee_id = emp_id;
	IF emp_salary < 8000 AND hire_year > 5
	THEN UPDATE employees SET salary = salary + 500 WHERE employee_id = emp_id;
	ELSE UPDATE employees SET salary = salary + 100 WHERE employee_id = emp_id;
	END IF;
END //
DELIMITER ;
```

##### `CASE`语句

```mysql
#情况一：类似于switch
CASE 表达式
WHEN 值1 THEN 结果1或语句1(如果是语句，需要加分号)
WHEN 值2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）

#情况二：类似于多重if
CASE
WHEN 条件1 THEN 结果1或语句1(如果是语句，需要加分号)
WHEN 条件2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）

#例1
CASE val
WHEN 1 THEN SELECT 'val is 1';
WHEN 2 THEN SELECT 'val is 2';
ELSE SELECT 'val is not 1 or 2';
END CASE;

#例2
CASE val
WHEN 1 THEN SELECT 'val is 1';
WHEN 2 THEN SELECT 'val is 2';
ELSE SELECT 'val is not 1 or 2';
END CASE;

#声明存储过程“update_salary_by_eid4”，定义IN参数emp_id，输入员工编号。判断该员工薪资如果低于9000元，就更新薪资为9000元；薪资大于等于9000元且低于10000的，但是奖金比例为NULL的，就更新奖金比例为0.01；其他的涨薪100元
DELIMITER //
CREATE PROCEDURE update_salary_by_eid4(IN emp_id INT)
BEGIN
	DECLARE emp_sal DOUBLE;
	DECLARE bonus DECIMAL(3,2);
	SELECT salary INTO emp_sal FROM employees WHERE employee_id = emp_id;
	SELECT commission_pct INTO bonus FROM employees WHERE employee_id = emp_id;
	CASE
	WHEN emp_sal<9000
		THEN UPDATE employees SET salary=9000 WHERE employee_id = emp_id;
	WHEN emp_sal<10000 AND bonus IS NULL
	THEN UPDATE employees SET commission_pct=0.01 WHERE employee_id = emp_id;
	ELSE
		UPDATE employees SET salary=salary+100 WHERE employee_id = emp_id;
	END CASE;
END //
DELIMITER ;
```

#### **循环语句(`LOOP|WHILE|REPEAT`)**

##### `LOOP`语句

> **``LOOP``循环语句用来重复执行某些语句。``LOOP``内的语句一直重复执行直到循环被退出（使用`LEAVE`子句），**==**跳出循环过程。  **==

```mysql
#标准形式
[loop_label:] LOOP
循环执行的语句
END LOOP [loop_label]

#实例1
DECLARE id INT DEFAULT 0;
add_loop:LOOP
	SET id = id +1;
	IF id >= 10 THEN LEAVE add_loop;
	END IF;
END LOOP add_loop;

#当市场环境变好时，公司为了奖励大家，决定给大家涨工资。声明存储过程“update_salary_loop()”，声明OUT参数num，输出循环次数。存储过程中实现循环给大家涨薪，薪资涨为原来的1.1倍。直到全公司的平均薪资达到12000结束。并统计循环次数。
DELIMITER //
CREATE PROCEDURE update_salary_loop(OUT num INT)
BEGIN
	DECLARE avg_salary DOUBLE;
	DECLARE loop_count INT DEFAULT 0;
	SELECT AVG(salary) INTO avg_salary FROM employees;
	label_loop:LOOP
		IF avg_salary >= 12000 THEN LEAVE label_loop;
		END IF;
		UPDATE employees SET salary = salary * 1.1;
		SET loop_count = loop_count + 1;
		SELECT AVG(salary) INTO avg_salary FROM employees;
	END LOOP label_loop;
	SET num = loop_count;
END //
DELIMITER ;
```

##### `WHILE`语句``(即c语言的while)``

```mysql
#标准形式
[while_label:] WHILE 循环条件 DO
循环体
END WHILE [while_label];

#实例1
DELIMITER //
CREATE PROCEDURE test_while()
BEGIN
	DECLARE i INT DEFAULT 0;
	WHILE i < 10 DO
		SET i = i + 1;
	END WHILE;
	SELECT i;
END //
DELIMITER ;

#市场环境不好时，公司为了渡过难关，决定暂时降低大家的薪资。声明存储过程“update_salary_while()”，声明OUT参数num，输出循环次数。存储过程中实现循环给大家降薪，薪资降为原来的90%。直到全公司的平均薪资达到5000结束。并统计循环次数。
DELIMITER //
CREATE PROCEDURE update_salary_while(OUT num INT)
BEGIN
	DECLARE avg_sal DOUBLE ;
	DECLARE while_count INT DEFAULT 0;
	SELECT AVG(salary) INTO avg_sal FROM employees;
	WHILE avg_sal > 5000 DO
		UPDATE employees SET salary = salary * 0.9;
		SET while_count = while_count + 1;
		SELECT AVG(salary) INTO avg_sal FROM employees;
	END WHILE;
	SET num = while_count;
END //
DELIMITER ;
```

##### `REPEAT`语句也就是`c语言中的do while`

> REPEAT语句**创建一个带条件判断的循环过程**。与**WHILE循环不同的是，``REPEAT``循环首先会执行一次循**
> **环，然后在 UNTIL 中进行表达式的判断**，**如果满足条件就退出，即`` END REPEAT``；如果条件不满足，则会
> 就继续执行循环，直到满足退出条件为止。  **

```mysql
#标准形式
[repeat_label:] REPEAT
循环体的语句
UNTIL 结束循环的条件表达式
END REPEAT [repeat_label]

#实例1
DELIMITER //
CREATE PROCEDURE test_repeat()
BEGIN
	DECLARE i INT DEFAULT 0;
	REPEAT
		SET i = i + 1;
		UNTIL i >= 10
	END REPEAT;
	SELECT i;
END //
DELIMITER ;

#当市场环境变好时，公司为了奖励大家，决定给大家涨工资。声明存储过程“update_salary_repeat()”，声明OUT参数num，输出循环次数。存储过程中实现循环给大家涨薪，薪资涨为原来的1.15倍。直到全公司的平均薪资达到13000结束。并统计循环次数
DELIMITER //
CREATE PROCEDURE update_salary_repeat(OUT num INT)
BEGIN
	DECLARE avg_sal DOUBLE ;
	DECLARE repeat_count INT DEFAULT 0;
	SELECT AVG(salary) INTO avg_sal FROM employees;
	REPEAT
		UPDATE employees SET salary = salary * 1.15;
		SET repeat_count = repeat_count + 1;
		SELECT AVG(salary) INTO avg_sal FROM employees;
		UNTIL avg_sal >= 13000
	END REPEAT;
	SET num = repeat_count;
END //
DELIMITER ;
```

#### **跳转语句(`ITERATE|LEAVE`)**

##### `LEAVE`语句(``即c语言的break`)

> **可以用在循环语句内，或者以 BEGIN 和 END 包裹起来的程序体内，表示跳出循环或者跳出
> 程序体的操作。如果你有面向过程的编程语言的使用经验，你可以把 LEAVE 理解为 break。**  

```mysql
LEAVE 标记名;
```

##### `ITERATE`语句(`即C语言的continue`)

> **只能用在循环语句（LOOP、REPEAT和WHILE语句）内，表示重新开始循环，将执行顺序
> 转到语句段开头处。如果你有面向过程的编程语言的使用经验，你可以把 ITERATE 理解为 continue，意
> 思为“再次循环”。  **

```mysql
ITERATE 标记名;
```

## 游标

> 在 `SQL` 中，游标是一种**临时的数据库对象**，可以**指向存储在数据库表中的数据行指针**。这里游标 充当了
> 指针的作用 ，我们**可以通过操作游标来对数据行进行操作**。  

### 声明游标

- `MySQL|SQL Server|DB2|MariaDB`

  ```mysql
  DECLARE 游标名 CURSOR FOR 查询语句;
  
  #实例1
  DECLARE cur_emp CURSOR FOR
  SELECT employee_id,salary FROM employees;
  
  #实例2
  DECLARE cursor_fruit CURSOR FOR
  SELECT f_name, f_price FROM fruits ;
  ```

- `Oracle|PostgreSQL`

  ```mysql
  DECLARE 游标名 CURSOR IS 查询语句;
  ```

### 打开游标

> **打开游标的时候 SELECT 语句的查询结果集就会送到游标工作区，为后面游标的 逐条读取 结果集中的记录做准备**  

- ```mysql
  OPEN 已经声明的游标的名字;
  ```

### 使用游标操作数据行

> **游标的查询结果集中的字段数，必须跟 INTO 后面的变量数一致，否则，在存储过程执行的时
> 候，``MySQL``会提示错误  **

- **标准格式**

  ```mysql
  FETCH 已经打开的游标的名字 INTO 变量1名[,变量2名...]
  ```

- **使用实例**

  ```mysql
  FETCH cur_emp INTO emp_id, emp_sal ;
  ```

### 关闭游标

> 有``OPEN``就必须有``CLOSE``，也就是**打开和关闭游标**。当我们**使用完游标后必须要关闭掉该游标**。因为**游标会**
> **占用系统资源 ，如果不及时关闭，**==**游标会一直保持到存储过程或存储函数结束**==，影响系统运行的效率。而**关闭游标**
> **的操作，会释放游标占用的系统资源**。

- ```mysql
  CLOSE cursor_name
  ```

## 触发器

> - `MySQL`从`5.0.2`版本开始支持触发器

### 触发器的应用场景

> 触发器类似一个自动执行的存储过程,其会在`INSERT|UPDATE|DELETE`这三个操作之一发生的时候自动执行.因此当我们有两个相互关联的表,如果我们对其中一个表的修改操作必须同时作用到另一个表,那么我们就可以用触发器来监控表的修改操作,那么当我们修改表是,触发器就会自动运行对另一个表同步修改操作
>
> **注意事项**:
>
> - **如果在子表中定义了外键约束，并且外键指定了ON UPDATE/DELETE CASCADE/SET NULL子句，此
>   时修改父表被引用的键值或删除父表被引用的记录行时，也会引起子表的修改和删除操作，此时基于子
>   表的UPDATE和DELETE语句定义的触发器并不会被激活  **

### 触发器中的`NEW`与`OLD`

> 每一个触发器中都自带`NEW`与`OLD`关键字,我们可以在触发器的`BEGIN`与`AND`中间的任何地方使用这两个关键字

#### `NEW`与`OLD`的作用

> `INSERT`:``NEW``用来表示将要（``BEFORE``）或已经（``AFTER``）插入的新的**一条数据记录**
>
> `UPDATE`:``OLD``用来表示将要或已经被修改的原数据，``NEW``用来表示将要或已经修改的**一条数据记录**
>
> `DELETE`:``OLD``用来表示将要或已经被删除的一条原**数据记录**

#### `NEW`与`OLD`的使用

> `NEW.columnName` （``columnName``为相应数据表某一列名）:可以用于获取当前记录的指定字段的取值

### 触发器的创建

```mysql
DELIMITER $
CREATE TRIGGER <触发器名称>
{AFTER|BEFORE} {INSERT|UPDATE|DELETE} ON <表名>
{FOLLOWS|PRECEDES} <一个已经创建的触发器的名称>
FOR EACH ROW
BEGIN
	语句块
END $
DELIMITER ;
```

### 触发器的查看

- **查看触发器创建的信息**

  ```mysql
  SHOW CREATE TRIGGER <触发器名>
  ```

- **查看当前数据库下所有触发器**

  ```mysql
  SHOW TRIGGERS
  ```

- **从系统库`information_schema`中的`TRIGGERS`表中获取指定触发器的详细信息**

  ```mysql
  SELECT * FROM information_schema.TRIGGERS;
  ```

### 触发器的删除

- ```mysql
  DROP TRIGGER IF EXISTS 触发器名称;
  ```

### 案例

```mysql
CREATE TABLE test_trigger (
id INT PRIMARY KEY AUTO_INCREMENT,
t_note VARCHAR(30)
);
CREATE TABLE test_trigger_log (
id INT PRIMARY KEY AUTO_INCREMENT,
t_log VARCHAR(30)
);


DELIMITER //
CREATE TRIGGER before_insert
BEFORE INSERT ON test_trigger
FOR EACH ROW
BEGIN
INSERT INTO test_trigger_log (t_log)
VALUES('before_insert');
END //
DELIMITER ;

INSERT INTO test_trigger (t_note) VALUES ('测试 BEFORE INSERT 触发器');

SELECT * FROM test_trigger_log;
```

### 触发器是否应该使用?

> - 实际上数据库插入,删除,修改一条记录的用时是很短暂的,但是如果使用触发器来完成这些工作的话,会导致严重的低效率问题.可能原来操作一条记录用时``0.1s``而使用触发器则需要``1s``
> - **因此,我们应该尽量不使用触发器,尤其是2对于增删改操作十分频繁的表.我们应该尽量在我们的`JAVA`开发端实现对触发器可以解决的问题的解决,从而避免使用触发器而导致的低效率的问题.**
