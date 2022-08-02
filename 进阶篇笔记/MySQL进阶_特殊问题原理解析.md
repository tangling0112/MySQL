## 1 用户登陆的底层流程

## 2 用户创建,信息修改,删除等操作的底层流程

## 3 为什么访问控制是先查`user`表,再查`db`表,再查`table_priv`表,再查`columns_priv`表?

## 4 当我们给用户赋予权限时的`MySQL`内部的流程会是如何?

## 5 `MySQL`常用系统变量汇总

## 6 红黑树原理剖析

## 7 从聚簇索引的原理的角度来解读`AUTO_INCREMENT`自增列的特性的设计初衷

## 8 从`MyISAM`与`Innodb`使用的不同索引机制上来解释为什么`MyISAM`需要索引与数据分离,而`Innodb`则将索引与数据都保存在`.ibd`文件中

## 9 `Innodb`引擎通过什么文件来保存数据表的非聚簇索引文件?

## 10 为什么`Innodb`引擎通过查询聚簇索引文件可以让我们直接获取到目标记录数据而不是其物理地址,难道只要我要查询这个表就要一次性把这个表的`.ibd`文件全部加载到内存中?

## 11 B+树的存储能力如何？为何说一般查找行记录，最多只需1~3次磁盘IO  

## 12 为什么说B+树比B树更适合实际应用中操作系统的文件索引和数据库索引？

## 13 Hash 索引与 B+ 树索引的区别  

## 14 Hash 索引与 B+ 树索引是在建索引的时候手动指定的吗？

## 15 B树的根节点与叶子节点的内部结构

## 16 B+树的根节点与叶子节点的内部结构

## 17 给出``100``条带自增列的用户记录如何建立起一棵`B`树,如何建立起一颗`B+`树

## 18 操作系统的页式空间管理

## 19 `Innodb`如何知道一个页中那些位置是存储的什么内容呢?

- 任何一个页的前`38Bytes`的空间都是**页头部基本信息空间**
- `Page Hearder`页面头部空间在任何一个页中相对于页面开始的偏移量都相同,且长度固定为`56Bytes`
- 通过`Page Hearder`的`PAGE_HEAP_TOP`可以获取到`Free Space`空闲空间相对于我们的页头部的偏移量
  - **注意**:`Free Space`的空间一定是`PAGE_HEAP_TOP`到页的尾部

## 20 为什么被删除的记录还会被存储在我们的页中?

> 你以为它删除了，可它还在真实的磁盘上。这些被删除的记录之所以不立即从磁盘上移除，是因为移除它们之后其他的记录在磁盘上需要`重新排列，导致性能消耗`。所以只是打一个删除标记而已，所有被删除掉的记录都会组成一个所谓的`垃圾链表`，在这个链表中的记录占用的空间称之为`可重用空间`，之后如果有新记录插入到表中的话，可能把这些被删除的记录占用的存储空间覆盖掉。

## 21 `Innodb`下`B+`树的组成与构建全过程

## 22 为什么多列索引下查询必须遵守最左前缀规则?

## 23 多列索引的实现原理(`Innodb`下)

## 24 全文索引的实现原理(`Innodb`下)



## 25 主键索引与唯一性索引的实现原理与区别

## 26 索引之所以可以加速我们的查询过程的原因剖析

## 27 `MySQL`针对外键约束自动建立的索引的原理与作用

## 28 普通索引的创建原理以及其与特殊索引的不同点

## 29 `ASC|DESC`对于索引创建的影响

## 30 为什么`AUTO_INCREMENT`的字段的唯一性索引无法被删除

## 31 多列索引创建后删除被索引的列中的某一个的效果

## 32 什么是通过反向扫描`ASC`索引来一定程度上实现`DESC`索引?

## 33 为什么`ORDER BY`排序的性能会受到`ASC|DESC`类型的索引影响?

## 34 `GROUP BY`子句与`ORDER BY`子句同时使用时,单列索引与多列索引的区别

> **考虑这样一个查询问题,我们有如下三种索引创建方式**
>
> ```mysql
> SELECT FROM
> GROUP BY 字段1
> ORDER BY 字段2
> ```
>
> - `字段1,字段2`分别**创建单列索引**(**``ORDER BY``子句未使用到索引,`GROUP BY`使用了``字段1的索引``**)
> - 按照`字段1,字段2`的顺序**创建双列索引**(**`ORDER BY与GROUP BY`都使用到了该联合索引,查询速度获得了提升**)
> - `字段1,字段2`分别**创建单列索引**,然后按照`字段1,字段2`的顺序**创建双列索引**(**`ORDER BY与GROUP BY`都使用到了索引,且使用的是联合索引而非单列索引**)
> - 按照`字段2,字段1`的顺序**创建双列索引**(**没有使用到该双列索引,该索引不满足最左前缀规则**)
> - `字段1,字段2`分别**创建单列索引**,**然后还按照**`字段2,字段1`的顺序**创建双列索引**(只使用了``字段1``的单列索引)

### 原因分析

#### 情况1

- 由于`SELECT`语句中**`GROUP BY`子句的执行顺序在`ORDER BY`之前**,因此**在`GROUP BY`子句执行时,就已经通过`字段1的索引`查出了我们的所有的记录**,并且由于`字段1的ASC型`索引的使用,我们的**记录查出来之后是按照``字段1``升序排列的**,`ORDER BY`子句**不再能去查询我们的数据文件**,**只能利用`GROUP BY`获取到的记录**.**不能查询文件**也就**无法谈及使用索引**

#### 情况2

- 我们的**`GROUP BY`子句通过使用该联合索引从我们的数据文件中查出了我们的记录**,并且我们的**记录会是按照这样的规则排序**的,由此**我们查询出来的记录在``字段2``上会是相对有序**的因此`ORDER BY`子句**也可以节省一定时间**
  - 首先所有记录都会按照``字段1``升序排序
  - 然后对于``字段1``取值相同的记录,就会按照`字段2`升序排序

#### 情况3

- 由于**联合索引**对于整个查询**具有最佳效率提升**,因此我们的`MySQL`**解析器安排我们的查询使用联合索引而不是单列索引**

#### 情况4

- 由于**`GROUP BY`子句会先执行**,而``情况3``下设定的索引对于`GROUP BY`子句而言**不满足最左前缀规则**,**因此不会使用`情况3`下设定的索引**

#### 情况5

- 联合索引**不满足最左前缀规则**,因此**只能使用单列索引**

## 35 由`SELECT`语句各个子句的执行顺序对`SELECT`语句使用索引的情况进行理论分析

## 36 `MySQL`对于索引使用的潜在规则(==**十分重要**==)

- **一个`SELECT`查询语句若只涉及到一次对`.ibd`内数据表的查询,那么该语句就最多只会使用一个索引**
- **`SELECT`查询语句若只涉及一次数据表的查询,那么其在涉及到对索引的使用时会根据解析器对于整个查询语句的分析选取一个满足索引使用规则的对于整个查询语句有最大效率提升的索引**

## ==**37 引入索引后`MySQL`的`SELECT`语句的关键字的先后排列顺序以及索引使用方式的具体原理剖析**==

## 38 `CHAR,VARCHAR,TEXT`等字符串类型的字段是如何建立索引的,字符串怎么排序呢?

## 39 字段区分度计算及其原理

### **基本公式**

$$
\frac{COUNT(DISTINCT \quad LEFT(列名,索引长度))}{COUNT(*)}
$$

### **原理**

## 40 基于字段区分度量化地分析取字符串类型的字段的前多少个字符用于建立索引最为合适(P132 12:44)

## 41 什么是页面分裂与记录移位

## 42 全局系统变量与会话系统变量的使用优先级

> 若一个系统变量即有全局的又有会话的,那么对于当前会话,其会优先使用会话系统变量

## 43 在不使用索引时,我们的`MySQL`是如何获取数据记录的?

## 44 `MySQL`是如何在不执行`SQL`语句的情况下估算该语句的开销的?

## 45 `MySQL`查询语句对于索引使用的理论化分析(以`B+`索引为例)

### 一个查询语句可以使用多少个索引?

### 单表单列索引查询语句是如何使用索引的?

> **情况1**
>
> - ``CREATE INDEX Students(stu_name)``
> - ``SELECT * FROM Students WHERE stu_name = "LiHua"``
>
> **情况2**
>
> - ``CREATE INDEX Students(stu_name)``
> - ``SELECT * FROM Students WHERE stu_name <=> "LiHua"``

### 单表多列索引查询语句是如何使用索引的

> **情况1**
>
> - `CREATE INDEX Students(stu_name,age,score)`
>
> - `SELECT * FROM Students WHERE stu_name = "LiHua" AND age = 18 AND score >= 98`
>
> **情况2**
>
> - `CREATE INDEX Students(stu_name,score,age)`
>
> - `SELECT * FROM Students WHERE stu_name = "LiHua" AND age = 18 AND score >= 98`

### 规则的理论化总结

> `MySQL`下一个单表无子查询的`SELECT`查询语句都只会有如下两个情况之一
>
> - `情况1`:**先查一个非聚簇索引文件,然后回表查询聚簇索引文件获取到所需数据记录**
> - `情况2`:**直接查聚簇索引文件就获取到所需数据记录**
>
> **注意**:当涉及到多表查询或者子查询时情况就更加复杂一点.但是道理也是异曲同工的

### 用理论化规则解释现象

- `SELECT * FROM Students GROUP BY school_id ORDER BY student_id`

> **前提**:数据表具备一个以`school_id,student_id`为顺序建立的联合索引

- `SELECT * FROM Students WHERE age=20 AND name = "汤凌" AND school_id = 56123 `

> **前提**:数据表具备一个以`age,name,school_id`为顺序的联合索引

- `SELECT * FROM Students WHERE age=20 AND name = "汤凌" AND school_id = 56123 `

> **前提**:数据表具备一个以`name,age,school_id`为顺序的联合索引

- `SELECT * FROM Students WHERE age>20 AND name = "汤凌" AND school_id = 56123 `

> **前提**
>
> - 数据表具备一个以`age,name,school_id`为顺序的联合索引
> - 开启了索引下推`ICP`

- `SELECT * FROM Students WHERE age=20 AND name = "汤凌" AND school_id = 56123 `

> **前提**
>
> - 数据表具备一个以`name,age,school_id`为顺序的联合索引
> - 数据表具备一个以`age,name,school_id`为顺序的联合索引

- `SELETC * FROM Students s JOIN Teachers t ON s.teacher_id = t.teacher_id WHERE s.student_name = "LiHua"`

> **前提**:
>
> - `Teacher`数据表具备一个以`teacher_id`建立的单列索引

- `SELETC t.teacher_id,t.teacher_name FROM Students s JOIN Teachers t ON s.teacher_id = t.teacher_id WHERE s.student_name = "LiHua"`

> **前提**:
>
> - `Teacher`数据表具备一个以`teacher_id,teacher_name`建立的联合索引

- `SELECT * FROM Students WHERE age = 18 GROUP BY teacher_id ORDER BY student_id`

> **前提**:
>
> - 数据表具备一个以`age,teacher_id,student_id`的联合索引

- `SELECT * FROM Students WHERE age > 18 GROUP BY teacher_id ORDER BY student_id`

> **前提**:
>
> - 数据表具备一个以`age,teacher_id,student_id`的联合索引
> - **没有开启**索引条件下推`ICP`

- `SELECT * FROM Students WHERE age > 18 GROUP BY teacher_id ORDER BY student_id`

> **前提**:
>
> - 数据表具备一个以`age,teacher_id,student_id`的联合索引
> - **开启了**索引条件下推`ICP`

- `SELECT * FROM Students WHERE age = 18 ORDER BY student_id`

> **前提**:
>
> - 数据表具备一个以`age,student_id`的联合索引

- `SELECT * FROM Students WHERE age > 18 ORDER BY student_id`

> **前提**:
>
> - 数据表具备一个以`age,student_id`的联合索引

## 46 联合索引与非联合索引在索引文件的内容组成上的区别

## 47 索引建立的经验总结

## 48 为什么涉及`>,<,>=,<=`等范围条件的字段还是可以使用索引?

## 49 为什么`IS NOT NULL`不能使用索引?

## 50 `SELECT * FROM Students WHERE age > 18 ORDER BY student_id`的`ORDER BY`子句能不能使用到索引?

## 51 `GROUP BY`是如何实现的对书局记录进行分组?

## 52 为什么我们的数据记录即便被删除了也不一定被真正删除,而是仅仅只是将该记录的记录头部中的`delete_mask`参数设置为`1`?

