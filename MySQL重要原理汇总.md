## 1 服务器进程与客户端进程通信的字符集转换

> **前提**：我们可以通过`SHOW VARIABLES LIKE "character_set%"`查看当前`MySQL`服务的通讯字符集

![image-20220711144230495](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220711144230495.png)

- 客户端进程发出`SQL`请求并按照`character_set_client`进行编码
- 将编码后的`SQL`语句通过网络发送到我们的服务器端进程
- 服务器端进程接收到客户端进程传递的消息后将其转换为`character_set_connection`指定的编码
- 服务器端开始对我们的`SQL`语句进行处理
- 服务器端得到处理结果,编码为`character_set_results`的结果
- 服务器端将处理结果发送给客户端进程
- 客户端进程将处理结果转换为`character_set_client`编码格式

## 2 用户创建相关操作

- **创建用户**

  ```mysql
  CREATE USER "用户名"@"主机名" IDENTIFIED BY "<密码>"
  CREATE USER "用户名"@"%" IDENTIFIED BY "<密码>"
  CREATE USER "用户名" IDENTIFIED BY "<密码>"
  ```

- **修改用户相关信息**

  ```mysql
  #修改用户名
  UPDATE USER SET user="新用户名" WHERE user="旧用户名"
  #修改用户HOST
  UPDATE USER SET host="新host" WHERE user="旧用户名" AND host = "旧host"
  #修改用户密码
  	#由于MySQL 8.0之后不再支持PASSWORD函数因此我们要修改用户密码最好采用先直接删除该用户然后再重新创建的方式,当然这种方式会导致我们曾经给该用户赋予的角色,权限等失效
  DROP USER "用户名"@"host"
  CREATE USER "用户名"@"host" IDENTIFIED BY "密码"
  ```


## 3 角色相关

- **角色的创建删除**

  ```mysql
  CREATE ROLE "角色名"@"主机IP";
  
  DROP ROLE "角色名"@"主机IP";
  ```

- **角色的权限授予与收回，用户授予角色收回角色**

  ```mysql
  GRANT 权限 ON 数据库名.数据表名 TO "角色名"@"主机IP";
  REVOKE 权限 ON 数据库名.数据表名 FROM "角色名"@"主机IP";
  
  
  GRANT "角色名"@"主机IP" TO "用户名"@"主机IP";
  REVOKE "角色名"@"主机IP" FROM "用户名"@"主机IP";
  ```

- **各类指标查看**

  ```mysql
  #查看当前用户的角色
  SELECT CURRENT_ROLE();
  #查看指定角色具有的权限
  SHOW GRANTS FOR "角色名"@"主机IP";
  ```

- **设置用户登入时的默认角色**

  > - 我们在授予了用户权限后必须通过`activate_all_roles_on_login`或`SET`语句设置指定用户登入时使用的角色,否则我们指定的用户虽然具备该角色,但是不能使用该角色的权限
  > - 若我们进行上述设置时用户已经为登入状态,那么必须要重新登陆才会生效

  ```mysql
  #通过activate_all_roles_on_login系统变量告诉系统当用户登陆时激活该用户具备的所有角色
  	#临时设置
  	SET @@global.activate_all_roles_on_login = 1;
  	#永久设置
  	[mysqld]
  	activate_all_roles_on_login = 1;
  #通过SET语句设置用户登入时的默认角色
  	#设置默认角色为所有角色
  	SET DEFUALT ROLE ALL TO "用户名"@"主机IP";
  	#设置指定角色为默认角色
  	SET DEFUALT ROLE "角色名"@"主机IP" TO "用户名"@"主机名"
  ```

- **设置强制角色**

  > 强制角色即每一个`MySQL`服务下所有在强制角色被设置后创建的用户都自动具备的默认角色,强制角色无法被`DROP`与`REVOKE`

  - **永久设置**

    ```mysql
    #修改配置文件
    [mysqld]
    mandatory_roles = '角色名'@'主机IP'[,'角色名1'@'主机IP1']
    #设置系统变量持久化
    SET PERSIST mandatory_roles = '角色名'@'主机IP'[,'角色名1'@'主机IP1']
    ```

  - **临时设置**

    ```mysql
    SET GLOBAL mandatory_roles ='角色名'@'主机IP'[,'角色名1'@'主机IP1']
    ```


## 4 `MySQL`配置文件的使用

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\07MySQL进阶.assets\image-20220712104009564.png" alt="image-20220712104009564" style="zoom:50%;" />

- 在**启动服务器能读取的组**中,我们可以设置`全局系统变量与会话系统变量`
- 在**启动客户端能读取的组**中,我们只能设置`会话系统变量`
  - 如果设置全局系统变量不会报错,但不会生效
- **若一个变量既有全局形式又有会话形式,那么在启动客户端能读取的组中设置的会是他的全局形式,在启动服务器能读取的组中我们设置的为其全局形式与会话形式**

### 选项组的优先级关系

> **这里的优先级关系是指,若两个选项组对同一个系统变量进行了设置,并且``mysql``命令会同时访问这两个选项组,那么最终该系统变量会遵循哪一个选项组下的设置,遵循谁,谁的优先级就高.**

- **基本规则**:若在配置文件中,我们的选项组A的位置相比于选项组B更靠近句首,那么选项组A的优先级就会更高,反之若选项组B更靠近句首,那么选项组B优先级更高
- **注意**:我们在调用`MySQL`服务命令时,是可以在命令行通过后缀`--系统变量名1=取值1 [--系统变量名2=取值2]`的方式给系统变量名赋值的,并且此时,我们的系统中该命令行的赋值优先级高于我们的配置文件,不过要注意的是,``MySQL``命令不会去同步修改我们的配置文件,因此如果重启`MySQL`服务,那么在我们不加命令行指定的情况下,其对应的系统变量就会以配置文件中的为准.

## 5 `MySQL`数据缓冲池

> - 我们`MySQL`的数据都是存储在我们的磁盘上的,而磁盘相对于高速的`CPU`而言是一个十分低速的设备,因此我们的操作系统一般都会通过`内存`来建立缓冲区,将磁盘上的数据不断地加载到我们的`内存缓冲区`上,由于在缓冲区充足的情况下我们不必等待`CPU`处理完传输过去的数据,可以一直连续不间断地将要使用到的数据从磁盘上导入到我们的``内存缓冲区``,我们知道`内存`是一个高速传输设备,因此通过建立`内存缓冲区`的方式可以极大地节约我们的`SQL`语句执行用时中用于数据`IO`的时间(当然在这个概念下是没有减少``IO``时间的,只不过是将原本串行的程序流程组织为了并行从而缩短了总用时).
> - 在上面的基础上,`MySQL`还建立了内存上的缓冲池,其中持久化地固定存储有大量的数据,当我们查询涉及到对这些数据的提取时,不再需要从`磁盘`进行读取,而是直接从内存读取,直接省略了`磁盘IO`这一步骤.当然值得注意的是,一般而言内存的存储量相对于磁盘而言是极小的,因此我们的`MySQL`系统不能考虑将所有的数据都导入到我们的内存中,而是要通过`数据的重要性=位置*访问频次`来决定一个数据的导入优先级.优先级高的数据才会被导入内存
> - **预读特性**:

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\07MySQL进阶.assets\image-20220712154223111.png" alt="image-20220712154223111" style="zoom: 50%;" />

#### 缓冲池大小查看与修改

> **注意**:大小的单位为`Bytes`

- **查看缓冲池大小**

  - `MyISAM`

    ```mysql
    SHOW VARIABLES LIKE 'key_buffer_size'
    ```

  - `Innodb`

    ```mysql
    SHOW VARIABLES LIKE 'innodb_buffer_pool_size'
    ```

- **修改**

  - **临时设置**

    ```mysql
    SET GLOBAL innodb_buffer_pool_size = N
    ```

  - **永久设置**

    ```shell
    [mysqld]
    innodb_buffer_pool_size = N
    ```

#### 多数据缓冲池

> **问题**:由于数据缓冲池的特性,我们的数据缓冲池显然是一个临界资源,因此我们必然会要涉及到加锁的问题,如果不加锁,`MySQL`的服务器的面向不同`客户端`的不同线程可能会在同一时间访问我们的数据缓冲池,这一现象带来的问题的严重性显然是不言而喻的,因此我们必须要进行加锁.但是如果加锁的话又会导致当一个线程正在访问我们的数据缓冲池时,其他的线程就无法访问我们的数据缓冲池,并陷入等待状态.而这会大大降低我们的系统的效率.因此`MySQL`引入了多数据缓冲池的概念
>
> **注意**
>
> - **`innodb_buffer_pool_size`存储的是总的数据缓冲池大小,也就是所有数据缓冲池大小的总和**
> - `innodb`默认设置在`innodb_buffer_pool`小于`1Gbits`的情况下不允许多数据缓冲池,即便我们设置了`innodb_buffer_pool_instances`大于``1``,系统也会将其自动改回`1`

- `MySQL`通过将一个大的数据缓冲池按需划分成不同大小的内存块,块与块之间相互独立,互不干涉.因此当多个线程同时需要访问数据缓冲池时,我们的`MySQL`系统就可以根据我们的各个线程的需求分配不同的数据缓冲池.

##### 设置数据缓冲池数量

```shell
[server]
innodb_buffer_pool_instances = N
```

##### 缓冲池数量查询

```mysql
SHOW VARIABLES LIKE 'innodb_buffer_pool_instances'
```

## 6 `SQL`的执行流程

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\07MySQL进阶.assets\image-20220712134625543.png" alt="image-20220712134625543" style="zoom:80%;" />

## 7 `MySQL`由于数据缓冲池的引入带来的数据同步问题

## 8 存储引擎相关

- **查看操作**

```mysql
#查看当前MySQL版本支持的所有引擎
SHOW ENGINES;
#查看当前全局默认存储引擎
SELECT @@global.defualt_storage_engine;
#查看当前会话默认存储引擎
SELECT @@session.defualt_storage_engine;
#查看所有存储引擎相关系统变量
SHOW VARIABLES LIKE "%storage_engine"
```

- **修改操作**

```mysql
#注意我们无法通过显式的方式设置数据库的默认存储引擎
CREATE TABLE 数据表名(字段列表) ENGINE = 引擎名;
ALTER TABLE 数据表名 ENGINE = 引擎名;
```

## 9 `Innodb`与`MyISAM`的对比

## 10 `HASH`索引的问题

- 哈希索引数据并不是按照索引值顺序存储的，所以无法用于排序

- 哈希索引**不支持部分索引列查找**，因为哈希索引始终是使用索引列的全部内容来计算哈希码。 如,以字段`A,B`为基础计算`HASH`值从而建立哈希索引，如果查询这涉及到字段``A``的筛选而不涉及到字段`B`，则无法使用该哈希索引(**无法计算HASH值**)

- 哈希索引**只支持等值查询**，包括`=、IN()、<=>`,**不支持范围查询**,如``where price > 100``

- 哈希冲突(不同索引列会用相同的哈希码)会影响查询速度,此时需遍历索引中的指向记录存储地址的指针,**逐个记录进行比较**。    

- 如果哈希冲突很多，一些索引维护操作的代价会很高。

- 如果从表中删除一行，需要遍历链表中的每一行，找到并删除对应行的引用，冲突越多，代价越大。

## 11 ``记录移位``,``页面分裂``、``页面回收``

### 记录移位

### 页面分裂

### 页面回收

## 12 `MySQL`页的基本组成

- **页基础信息空间**
- **`Free Space`空闲空间**
- **`User Record`用户记录空间**
- **`Infimum`最小记录空间**
- **`Supremum`最大记录空间**
- **页目录空间**
- **`Page Header`页面头部空间**

## 13 页目录空间的更新流程

> **注意**:在`B+`树下所有的用户记录通过`单向链表`连接并且按照`主键`取值排序.而单向链表的问题在于其检索效率低下,若要检索就必须对整个单向链表进行遍历.因此`MySQL`在每一个页中引入了页目录空间,这个空间中保存的是一些特定的用户记录的单向链表节点的访问地址,以及该记录的主键值等一些补充信息.
>
> **查询记录的流程**:
>
> - 首先找到页目录空间
> - 查询页目录空间找到我们要主键值**小于等于**我们查找的记录的主键值,且两主键值差值最小的分组,获取该分组存储的物理地址
> - 按照物理地址找到单向链表上的节点然后以单向链表遍历的方式向后遍历查找我们的用户记录
>
> **优点**:
>
> - **将单向链表抽象地组织成为了一个类似于数组的数据结构**,使得我们**可以运用二分查找等有序数组元素查找算法**(**用空间换时间**)找到我们的记录所在的分组的上一个分组的最后一条记录的物理地址,从而加快记录查找速度
>
> **页目录空间存储一些什么**
>
> - **各个分组的最后一条记录的相对于该数据页的第一个`Bit`的物理偏移量**
> - **各个分组的最后一条记录的主键值**

### 分组规则

- :one:第一组中只能拥有最小记录这一条记录
- :two:最后一组中只能拥有`1~8`条记录(包括最大记录)
- :three:中间的组中必须保证记录数在`4~8`条

### 记录的插入规则

- :one:初始情况下,一个数据页里**只有最小记录和最大记录两条记录**，它们**分属于两个分组**。
- :two:之后**每插入一条记录**，都会**从页目录中**将其插入到==**主键值比本记录的主键值大**==,==**差值最小的槽所在的分组中**==，然后把**该槽对应的记录的记录头信息的``n_owned``参数值加1**，表示本组内又添加了一条记录，直到该组中的记录数等于8个。
  - **插入方式很简单,我们通过槽可以访问到该分组的最后一个记录,通过该记录我们又可以访问它前面的记录,从而按照`n_owned`参数找到合适的位置就可以让前面的记录的指针指向它,它的指针指向前一个记录原本指向的记录即可实现插入**
- :three:**在一个组中的记录数等于8个后再插入一条记录时，会将组中的记录拆分成两个组，一个组中4条记录，另一个5条记录。这个过程会在页目录中新增一个槽来记录这个新增分组中最大的那条记录的偏移量。**
  - ==**注意**==
    - 拆分方式是将该分组的前四个记录作为一个分组,后四个记录以及新插入的一条记录作为一个分组
    - 拆分后需要更新拆分的分组的基本信息(**物理偏移量,主键值等**)

## 14 常见行格式与其特点

- **设置数据表的行格式**

  ```mysql
  CREATE TABLE 表名(字段列表) ROW_FORMAT = 行格式;
  ALTER TABLE 表名 ROW_FORMAT = 行格式
  ```

### 常见行格式

- `COMPACT`
- `Dynamic`
- `Compressed`
- `Redundant`

### 基础的行格式组成

- **记录的变长字段的长度列表**

  - 存储该记录各个变成的字段使用了多长的长度

- **`NULL`值列表**

  - 存储所有取值为`NULL`的字段

- **记录头信息(`5Bytes`)**

  - 存储一些记录的基本辅助信息,如**记录是不是被删除了,记录所在的分组的记录数,记录的类型**

- **实际用户记录数据**

  > **注意**:**在默认情况下一条记录至少有如下三个基本参数**

  - | 列名             | 是否必须 | 占用空间 | 描述                   |
    | ---------------- | -------- | -------- | ---------------------- |
    | `row_id`         | 否       | 6字节    | 行ID，唯一标识一条记录 |
    | `transaction_id` | 是       | 6字节    | 事务ID                 |
    | `roll_pointer`   | 是       | 7字节    | 回滚指针               |

    - `row_id`:若一个表没有手动定义主键，则会自动选取一个``Unique``键的字段作为主键，如果连``Unique``键都没有定义的话，则**会为表默认添加一个名为``row_id``的隐藏列作为主键**。所以``row_id``是在**没有自定义主键以及``Unique``键的情况下才会存在的**。

### 特点

- **``Compressed和Dynamic``**:对于存放在``BLOB``字段中的数据采用了**完全的行溢出的方式**。即数据页中只存放20个字节的指针（溢出页的地址），实际的数据都存放在``Off Page`（溢出页）中
- **`Compact和Redundant`**:在记录的真实数据处存储一部分数据(存放``768``个前缀字节),剩下的数据存储在`溢出页`中

## 15 索引相关

- **索引管理**

  ```mysql
  #查看指定数据表的所有索引
  SHOW INDEX FROM 表名
  #索引隐藏
  	#表创建时设置索引隐藏
  CREATE TABLE 表名(
  字段列表
  [UNIQUE|PRIMARY|SPATIAL|FULLTEXT] [INDEX|KEY] 索引名(索引字段列表) [INVISIBLE|VISIBLE]
  )
  
  	#表创建好后基于ALTER添加
  ALTER TABLE 表名
  ADD [UNIQUE|PRIMARY|SPATIAL|FULLTEXT] [INDEX|KEY] 索引名(索引字段列表) [INVISIBLE|VISIBLE]
  	
  	#表创建好后基于CREATE INDEX添加
  CREATE [UNIQUE|PRIMARY|SPATIAL|FULLTEXT] [INDEX|KEY] 索引名 [INVISIBLE|VISIBLE] ON 表名(索引字段列表)
  
  	#表创建好后修改索引
  ALTER TABLE 表名
  ALTER INDEX 索引名 [INVISIBLE|VISIBLE]
  ```

- **索引创建**

  ```mysql
  #创建表时添加索引
  CREATE TABLE 表名(
  字段列表
  [UNIQUE|PRIMARY|SPATIAL|FULLTEXT] [INDEX|KEY] 索引名(字段1(长度)[,字段2(长度)...]) [ASC|DESC]
  );
  
  #表创建好后添加索引
  	#方式1
  ALTER TABLE 表名
  ADD [UNIQUE|PRIMARY|SPATIAL|FULLTEXT] [INDEX|KEY] 索引名(字段1(长度)[,字段2(长度)...]) [ASC|DESC];
  	#方式2
  CREATE [UNIQUE|PRIMARY|SPATIAL|FULLTEXT] [INDEX|KEY]  索引名 ON 表名(字段1(长度)[,字段2(长度)...]) [ASC|DESC];
  ```

- **索引删除**

  ```mysql
  #ALTER方式
  ALTER TABLE 表名
  DROP [UNIQUE|FULLTEXT|PRIMARY|SPATIAL] [INDEX|KEY] 索引名;
  #DROP INDEX方式
  DROP [UNIQUE|FULLTEXT|PRIMARY|SPATIAL] [INDEX|KEY] 索引名 ON 表名
  ```

### 索引设置的注意事项

- :one:我们**尽量**在**创建索引时保证数据表中的数据已经填充的差不多**了,这样能够**避免我们不断添加记录到数据表中导致的系统额外的用于动态更新索引的开销**
- :two:对于一个我们**已经创建的表**,当我们将要对该表进行大量的==**增删改**==操作时,我们可以**事先将表的大部分索引删除**,**然后**再进行==**增删改**==**,这样可以最大限度地**==**减少用于动态索引更新的系统资源消耗**==**,当表的增删改操作完成后我们再将索引重新创建,这样的**==**静态创建相对于动态更新的资源消耗少**==
- :three:`DROP PRIMARY KEY ON 表名`可以**删掉指定表的主键与主键约束**,但是我们必须清楚`PRIMARY KEY`**默认带有的`NOT NULL`效果是不会被删除**的,如果**要删除**那么就需要**通过`ALTER TABLE MODIFY `的方式来进行**
- :four:添加了**`AUTO_INCREMENT`的字段**的**`UNIQUE`唯一性索引是无法被删除**的
- :five:**对于多列索引,我们可以删除其对应的字段.当我们把涉及到的所有字段全部删除后该多列索引也会被自动删除**
  - 对于一个三列索引,删除其中一列就会自动变成双列索引,再删一个就变成单列索引,再删一个该索引就会被自动去除
- :six:**`DESC`降序索引只在`MySQL 8.0`版本之后生效**,**之前的版本并不会真的按照降序来创建索引**(即就算指定`DESC` `MySQL`也还是会按照`ASC`创建索引)
  - 当然虽然创建时不会按照`ASC`创建,但是其会在查询时通过`反向扫描的方法`来一定程度上实现降序`DESC`索引

## ==**16 常用数据库`STATUS`性能指标**==

> **查询方法**
>
> ```mysql
> SHOW STATUS LIKE "参数名"
> ```

### `MySQL`服务相关

- `Connections`:`MySQL`服务器启动以来被客户端连接的总次数
- `Uptime`:`MySQL`服务器上线的时间
- `Slow_queries`:`MySQL`服务器启动以来慢查询的次数

### 锁相关

- `Innodb_row_lock_current_waits `:当前`MySQL`服务进程``is_waiting=true``的锁的个数
- `Innodb_row_lock_time`:从系统启动到现在锁定总时间长度  
- `Innodb_row_lock_time_avg`:从系统启动到现在,所有进入过等待状态的锁等待时间的平均值
- `Innodb_row_lock_time_max`:从系统启动到现在,所有进入过等待状态的锁中等待时长最长的锁的具体等待时长
- `Innodb_row_lock_waits`:从系统启动到现在进入过等待状态的锁的总个数  

### 数据记录相关

- `Innodb_rows_read`:`MySQL`服务器启动以来所有`SELECT`语句返回的记录总数
- `Innodb_rows_inserted`:`MySQL`服务器启动以来执行`INSERT`指令插入的记录总数
- `Innodb_rows_updated`:`MySQL`服务器启动以来执行`UPDATE`指令更新的记录总数
- `Innodb_rows_deleted`:`MySQL`服务器启动以来执行`DELETE`指令删除的记录总数

### `SQL`语句相关

- `Com_select`:`MySQL`服务器启动以来查询操作的总次数
- `Com_insert`:`MySQL`服务器启动以来插入操作的总次数(一次插入多个记录只算一次插入)
- `Com_update`:`MySQL`服务器启动以来更新操作的总次数
- `Com_delete`:`MySQL`服务器启动以来删除操作的总次数
- **`last_query_cost`:返回最新的一条`SQL`语句的执行成本** 

### 慢查询相关

- `slow_query_log`:是否开启慢查询日志

**注意**:**当`MySQL`服务器重启后这些参数都会被重置**

## ==**17 常用`VARIABLES`系统变量**==

### **重要参数**

- **`lower_case_table_names`:当前`MySQL`服务的大小写规则**
- **密码相关**

  - `authentication_policy`:当前系统使用的密码加密策略
  - `default_authentication_plugin`:默认密码加密策略
  - **`password_history`:密码重用策略参数之一**
  - **`password_reuse_interval`:密码重用策略参数之一**
  - **`default_password_lifetime`:默认的密码过期时间**
- **自增相关**

  - **`auto_increment_increment`:自增列的第一个记录的默认值**

  - **`auto_increment_offset`:自增列每一次自增的步长**
- **ORDER BY相关**
  - **`sort_buffer_size`:用于`File Sort`的内存空间的大小**
  - **`max_length_for_sort_data`:能够参与单路`File Sort`的记录的最大长度**
- **角色相关**

  - **`mandatory_roles`:强制角色**
  - **`activate_all_roles_on_login`:`0/1`是否在用户登入时激活其具备的所有角色**
- **JOIN相关**

  - `max_join_size`
  - **`join_buffer_size`:用于辅助`Block Neasted-Loop JOIN`操作的内存缓冲区大小**
- **Profile优化工具相关**

  - **`profiling`:是否开启`SHOW PROFILES`操作**

  - **`profiling_history_size`:``profiling`存储的历史`SQL`语句执行参数条数**
- **表约束相关**

  - `sql_require_primary_key`:是否在创建表时必须指定表的主键
  - **`autocommit`:`0/1`,是否启用一些`SQL`语句执行后自动执行`COMMIT`指令的功能**
  - `foreign_key_checks`:是否启用外键约束检查
- **其他**

  - `optimizer_switch`:`MySQL`优化器的一些设置
  - `block_neasted_loop`:`on/off`:是否开启`Block Neasted-Loop Join`
  - `use_invisible_indexes`:`on/off`:是否让我们的系统可以使用被隐藏的索引
  - `hash_join`:`on/off`:是否开启`hash_join`
  - `index_condition_pushdown`:`on/off`:是否开启索引条件下推
  - `engine_condition_pushdown`
  - `derived_condition_pushdown`
  - `index_merge`:
  - `index_merge_union`:
  - `index_merge_sort_union`:
  - `index_merge_intersection`:

### 事务相关

- **`autocommit`:是否开启自动提交**
- **`innodb_log_buffer_size`:用于指定`Redo Log`的内存缓冲区的大小**
- **`innodb_flush_log_at_trx_commit`:用于指定我们的`Redo Log Buffer`中的数据的刷盘策略**
- `innodb_log_group_home_dir`:指定`Redo Log File`文件的存储路径，默认值为`./`即我们`MySQL`数据库的数据目录
- `innodb_log_files_in_group`:明`Redo Log File`的个数，命名方式如:``ib_logfile0，ib_logfile1...ib_logfilen``.默认**2**个，最大**100**个。
- `innodb_log_file_size`:给单个`Redo Log File`文件设置大小,默认值为``48MB`` .最大值为``512GB``,注意最大值指的是整个`Redo Log File`系列文件之和，即（`innodb_log_files_in_group * innodb_log_file_size `）不能大于最大值`512GB`。

### 锁相关

- `innodb_table_locks`:
- **`innodb_autoinc_lock_mode`:用于指定自增锁的锁定模式,有`0/1/2`三种模式**
- **`innodb_lock_wait_timeout`:用于指定等待加锁的最大等待时长**
- **`innodb_deadlock_detect`:指定是否开启死锁检测**

### 日志相关

#### 慢查询日志

- **`slow_query_log`:是否开启慢查询日志**
- `slow_query_log_file`:慢查询日志的存储位置

- **`long_query_time`:指定超过多少秒的查询为慢查询**

#### 通用查询日志

- **`general_log`:指定通用查询日志的开启与关闭**
- `general_log_file`:指定通用查询日志文件的存储位置

#### 错误日志

- **`log_error`:指定错误日志文件的存储路径与文件名以及后缀**

#### 二进制日志

- **`log_bin`:用于指定二进制日志的开启与关闭**
- **`log_bin_basename`:二进制日志文件的数据文件存储路径与文件名前缀**
- **`log_bin_index`:二进制日志`index`文件的存储路径与文件名**
- **`binlog_expire_logs_auto_purge`:指定二进制日志数据文件的自动删除机制的开启与关闭**
- **`binlog_expire_logs_seconds`:指定二进制日志数据文件的有效时长(超过有效时长将被自动删除)**
- `log_bin_trust_function_creators`;
- `log_bin_use_v1_row_events`:
- `sql_log_bin`:
- **`max_binlog_size`:指定一个`binlog`数据文件的最大存储空间占用**
- `binlog_cache_size`:
- **`binlog_checksum`:指明`binlog`文件使用的差错检测算法**
- `max_binlog_cache_size`:
- **`sync_binlog`:用于指定`Bin Log Cache`中数据的刷盘策略**
  - `0`
    - 每次事务`COMMIT`后都**只会将`Bin Log Cache`中的数据转存到我们的操作系统的文件缓冲区**中,由我们的操作系统决定什么时候将数据刷盘到磁盘上
  - `1`
    - 每一次事务`COMMIT`后都会将`Bin Log Cache`中的数据转存到我们的操作系统的文件缓冲区,并且将其通过文件缓冲区又直接刷盘到磁盘上
  - `N`
    - 每一次事务`COMMIT`后都会将`Bin Log Cache`中的数据转存到我们的操作系统上的文件缓冲区,但是不会直接通过文件缓冲区将数据刷盘到磁盘上,而是等待知道有`N`个事务的`Bin Log Cache`的数据存储到我们的文件缓冲区,才会将这`N`个事务的数据刷盘到我们的磁盘上
- **`binlog_format`:指明`binlog`中的数据存储格式`ROW|STATEMENT|MIXED`**
  - **STATEMENT**
    - 每一条涉及到数据修改的`SQL`语句都会被记录
  - **ROW**
    - `5.1.5`版本的`MySQL`才开始支持 
    - 不记录`SQL`语句上下文相关信息，**仅保存哪条记录被修改**  
  - **MIXED**
    - 从`5.1.8`版本的`MySQL`开始支持
    - 是`Statement`与`Row`的结合  

### 中继日志

- `relay_log`:
- **`relay_log_basename`:指定中继日志数据文件的保存路径与文件名前缀**
- **`relay_log_index`:指定中继日志索引文件的保存路径与文件名(后缀为`.index`)**
- `relay_log_info_file`:
- `relay_log_info_repository`:
- `relay_log_info_repository`:
- `relay_log_recovery`:
- `sync_relay_log`:
- `sync_relay_log_info`

### 数据表导出相关

- **`secure_file_priv`:指定数据表导出的文本文件的存储路径,除了这个存储路径其他路径都是不合法的.**

### 基本信息相关

- `version`:当前``MySQL``服务的版本号
- `version_comment`:当前`MySQL`服务器的类型
- `time_zone`:当前`MySQL`服务的时区
- `timestamp`:当前时区下的当前时间的时间戳

### 存储引擎相关

- `innodb_%`:`innodb`存储引擎相关参数
- `myisam_%`:`MyISAM`存储引擎相关参数
- `innodb_buffer_pool_size`:`innodb`存储引擎的数据缓冲池的大小
- `innodb_page_size`:`innodb`存储引擎的数据页的大小
- `innodb_version`:`innodb`存储引擎的版本
- `have_%`:当前`MySQL`默认引擎支持的功能
- `default_storage_engine`:默认的存储引擎

### 字符集相关

- `character_set_client`
- `character_set_connection`
- `character_set_database`
- `character_set_filesystem`
- `character_set_results`
- `character_set_server`

- `character_set_system`

- `character_sets_dir`

### 比较字符集相关

- `collation_connection`
- `collation_database`
- `collation_server`
- `default_collation_for_utf8mb4`

### 服务相关

- `connect_timeout`
- `connection_memory_chunk_size`
- `connection_memory_limit`
- `basedir`:`MySQL`的根目录
- `datadir`:`MySQL`数据文件存储位置
- `hostname`:
- `max_connections`:最大的客户端接入数
- `performance_schema`:
- `port`:`MySQL`服务的端口
- `shared_memory_base_name`:`MySQL`服务的名称

## 18 `Profiling`性能分析工具

- **开启`PROFILE`工具**

  ```mysql
  #查看PROFILE工具是否开启
  SHOW VARIABLES LIKE "PROFILING"
  
  #临时设置
  	#开启
  SET PROFILING = ON
  	#关闭
  SET PROFILING - OFF
  
  #永久设置
  [mysqld]
  	#开启
  PROFILING = ON
  	#关闭
  PROFILING = OFF
  ```

- **使用`PROFILE`工具**

  ```mysql
  #查看所有执行过的语句以及他们的Query_ID
  SHOW PROFILES;
  
  #查看最近的一次SQL语句的系统资源消耗
  SHOW PROFILE;
  
  #查看指定Query_ID的SQL语句的指定的系统资源消耗
  SHOW PROFILE [字段列表] FOR query <Query_ID>
  ```

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220718113201072.png" alt="image-20220718113201072" style="zoom:67%;" />

## 19 `EXPLIAN`性能分析工具

### 字段解释

| 列名            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `id`            | 用于标识该条记录所属的`SELECT`关键字,当一个语句中出现了``N``个`SELECT`关键字时返回的所有记录就有`N`个不同的`id`<br />**注意**:由于我们优化器的存在,可能原本涉及多个`SELECT`的语句会被优化为只有一个`SELECT`关键字的多表查询,而此时我们的`id`会以优化器执行的语句来返回 |
| `select_type`   | 用于表示该记录对应的`SELECT`查询的类型                       |
| `table`         | 用于指明我们当前执行计划的操作目标表的表名                   |
| `partitions`    |                                                              |
| `type`          | 指明该`SELECT`查询对数据表进行访问的方式                     |
| `possible_keys` | 指明对该表进行该`SELECT`查询可以利用到的索引,**(ke能有多个)** |
| `key`           | 指明对该表进行该`SELECT`查询我们最终选取用于真正执行的索引**(只会有一个)** |
| `key_len`       | ![image-20220718140207735](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220718140207735.png)<br /><img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220718140325799.png" alt="image-20220718140325799" style="zoom: 50%;" /> |
| `ref`           | ![image-20220718140620704](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220718140620704.png) |
| `rows`          | <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220718140640082.png" alt="image-20220718140640082" style="zoom:67%;" /> |
| `filtered`      | ![image-20220718140748699](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220718140748699.png) |
| `Extra`         | 一些额外的信息,`MySQL`默认的额外信息字段有几十个             |

## 20 `SHOW WRANINGS`工具

> 当我们`EXPLAIN`一条语句后,可以通过`SHOW WARNINGS`获取到该语句在**经过解析器优化器优化后的真正用于执行的语句**

![image-20220718172857250](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220718172857250.png)

## 21 `Sys`数据库使用

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220718174048536.png" alt="image-20220718174048536" style="zoom:50%;" />

**索引情况**

```mysql
	#1. 查询冗余索引 
select * from sys.schema_redundant_indexes; 
#2. 查询未使用过的索引 
select * from sys.schema_unused_indexes; 
#3. 查询索引的使用情况 
select index_name,rows_selected,rows_inserted,rows_updated,rows_deleted from sys.schema_index_statistics where table_schema='dbname' ;
```

**表相关**

```mysql
# 1. 查询表的访问量 
select table_schema,table_name,sum(io_read_requests+io_write_requests) as io from sys.schema_table_statistics group by table_schema,table_name order by io desc; 
# 2. 查询占用bufferpool较多的表 
select object_schema,object_name,allocated,data
from sys.innodb_buffer_stats_by_table order by allocated limit 10; 
# 3. 查看表的全表扫描情况 
select * from sys.statements_with_full_table_scans where db='dbname';
```

**语句相关**

```mysql
#1. 监控SQL执行的频率 
select db,exec_count,query from sys.statement_analysis order by exec_count desc; 
#2. 监控使用了排序的SQL 
select db,exec_count,first_seen,last_seen,query
from sys.statements_with_sorting limit 1; 
#3. 监控使用了临时表或者磁盘临时表的SQL 
select db,exec_count,tmp_tables,tmp_disk_tables,query
from sys.statement_analysis where tmp_tables>0 or tmp_disk_tables >0 order by (tmp_tables+tmp_disk_tables) desc;
```

**``IO`相关**

```mysql
#1. 查看消耗磁盘IO的文件 
select file,avg_read,avg_write,avg_read+avg_write as avg_io
from sys.io_global_by_file_by_bytes order by avg_read limit 10;
```

**``Innodb``** **相关**

```mysql
#1. 行锁阻塞情况 
select * from sys.innodb_lock_waits;
```

## 22 索引失效的原因汇总

- 违反**最左前缀原则**
- 字段涉及**计算、函数、类型转换(自动或手动)**的使用导致索引失效
- **联合索引中涉及到范围条件**的字段的右边的字段在不使用索引下推的情况下无法使用索引
- 不等于(``!=``或者``<>``)索引失效
- ``IS NULL``可以使用索引，``IS NOT NULL``无法使用索引
- `LIKE`以通配符``%``开头索引失效
- `OR`前后存在非索引的列，索引失效
- **字符集或比较字符集未统一**

## 23 多表查询优化

### **经验总结**

- 对于内连接来说，**驱动表的每一条记录都会被遍历(不这样没法生成过滤的笛卡尔积，这是必须的)**，因此驱动表的连接条件的字段即便有索引也用不到
- 对于内连接来说，查询优化器可以决定谁来作为驱动表，谁作为被驱动表出现
- 对于内连接来讲，如果表的连接条件中**只有一个字段有索引**，则**有索引的字段所在的表会被作为被驱动表**
- **对于内连接来说，在两个表的连接条件都存在索引的情况下，会选择小表作为驱动表。`小表驱动大表`**
- 平均效率有`Index Neasted-Loop JOIN > Block Neasted-Loop JOIN > Simple Neasted-Loop JOIN`
- 即便连接条件涉及的字段均没有索引可以使用,也会遵循`小表驱动大表`的规则
- 尽量让我们的被驱动表的对应字段有索引可以使用
- 尽量减少`SELECT`中要呈现的驱动表中的字段,因为在进行`Block Neasted-Loop JOIN`时都会要实现将所有的驱动表需要使用到的字段读取到`JOIN Buffer`中,而我们驱动表要使用的字段越多就会导致使用``JOIN Buffer`中的空间被消耗的越多,从而导致`Loop JOIN`效率降低
- `Block Neasted-Loop JOIN`是对`Simple Neasted-Loop JOIN`的改进,采用了空间换时间的理念.只有当我们的被驱动表没有索引可以使用时才会考虑使用这两个算法.有索引可用时还是会使用`Index Neasted-Loop JOIN`

### `Join`算法汇总

> **注意**:
>
> - **`Block Neasted-Loop Join`是否开启可以通过`optimizer_switch`系统变量查看**

- **`Simple Neasted-Loop Join`**

  - ![image-20220719153407749](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719153407749.png)

- **`Index Neasted-Loop Join`**

  - <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\07MySQL进阶.assets\image-20220719153324005.png" alt="image-20220719153324005" style="zoom: 55%;" />

- **`Block Neasted-Loop Join`**

  - <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719153604165.png" alt="image-20220719153604165" style="zoom:50%;" />

- **`Hash Join`**

  > **`Hash Join`在`MySQL 8.0.20`之后引入了`Hash Join`并且将其设置为默认的连接方式**
  >
  > **注意**:
  >
  > - 连接条件非等值时还是得老实使用`Nested-Loop Join`
  > - 可以使用`Index Nested-Loop Join`且数据量很大时还是使用`Index Nested-Loop Join`

  - ![image-20220719155750392](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719155750392.png)

## 24 子查询优化

> - **尽量使用多表查询来代替子查询.以避免临时表的使用以及使得索引的使用成为可能**
> - **尽量利用`SHOW PROFILES`,`EXPLAIN`等性能分析工具分析我们的等价多表查询语句与原子查询语句的效率,再决定如何实现功能.**

- 对于涉及子查询的`SQL`语句,``MySQL``需要**为内层查询语句的查询结果`建立一个临时表`**，然后**外层查询语句从临时表中查询记录**。查询完毕后，再`撤销这些临时表`,这样会消耗过多的CPU和IO资源;并且这样的零时表是必然不具备索引的,因此这些综合起来就会导致**产生大量的慢查询**。

### 尽量减少`NOT IN`与`NOT EXIST`的使用

> - **尽量使用用``LEFT JOIN 表名 ON 连接条件 WHERE 过滤条件 IS NULL``等效解决问题**

## 25 `ORDER BY`排序优化

> **优化目的**:尽量避免`File Sort`的使用,尽可能借助索引来实现`ORDER BY`操作

### `LIMIT`相关

- **不使用`LIMIT`,且查询的字段不全在索引中,需要涉及回表操作会导致即便有索引也不使用,而是使用`File Sort`**
  - 当我们的数据表的记录数十分庞大,并且用于辅助排序的`索引`不是主键索引时,由于会涉及到大量的记录的索引,回表,因此如果不使用`LIMIT`关键字,那么`MySQL`可能会在综合评估下认为使用索引进行排序的消耗大于`File Sort`从而在可以使用索引辅助排序的情况下采取`File Sort`;如果使用了`LIMIT`关键字就限制了需要索引并回表的记录数,从而使得`MySQL`评估使用索引辅助排序相对于`File Sort`更高效
- **使用`LIMIT`,且查询的字段不全在索引中,但`LIMIT`的数量过大同样也会导致即便有索引也不使用,而是使用`File Sort`**
- **不使用`LIMIT`,但查询的字段全在索引中,有索引可以使用就会使用索引辅助排序**
- **使用`LIMIT`,且查询的字段全在索引中,有索引可以使用就会使用索引辅助排序**

### 其他

- **`ORDER BY`涉及了多个字段,这些字段有联合索引,但是`ORDER BY`子句违反了最左前缀规则会导致无法使用索引辅助排序,从而导致使用`File Sort`**

  - ```mysql
    CREATE INDEX salary_lastname ON employees(salary,lastname);
    SELECT last_name FROM employees ORDER BY last_name;
    ```

-  **`ORDER BY`涉及的字段的联合索引按照`ASC|DESC`排序,而我们`ORDER BY`中却是使用`DESC|ASC`,就会发生如下四种情况**

  > ![image-20220719183956337](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719183956337.png)

- **`SELECT`操作的子句优先级会决定`ORDER BY`是否能使用到索引**

  - 当`SELECT`语句中同时使用了`WHERE`子句与`ORDER BY`子句时,我们的`MySQL`优化器一定会优先保证`WHERE`子句可以使用索引,即便由于让`WHERE`子句使用了索引而导致`ORDER BY`子句使用不了索引也无所谓	
  - 当`SELECT`语句中同时使用了`GROUP BY`子句与`ORDER BY`子句时,我们的`MySQL`优化器一定会优先保证`GROUP BY`子句可以使用索引,即便由于让`GROUP BY`子句使用了索引而导致`ORDER BY`子句使用不了索引也无所谓

- 若

- ![image-20220719184353983](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719184353983.png)

- ![image-20220719191214903](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719191214903.png)

  -  **问题**
    - **我们发现方案2本来还可以继续做索引条件下推,但是`MySQL`直接跳出来不对`NAME`继续索引下推,而是直接出来做`FileSort`,其原因在于经过前面的过滤后剩下的要排序的字段很少了,排序很快,索引优化器制定了这样的执行计划**

### 常见`File Sort`算法

#### 双路排序

> **涉及到两次磁盘`IO`**

- **第一次读取磁盘**
  - 首先直接读取出指定数据表的每一行的行指针以及该行的`ORDER BY`涉及到的字段的值并一一对应起来建立出一个列表(在`MyISAM`时)
    - **在`Innodb`下是`主键值`与该行的`ORDER BY`涉及到的字段的值建立一一对应列表.**
- **然后对上一步得到的列表按照我们的`ORDER BY`涉及到的字段的值进行排序(不改变一一对应关系)**
- **第二次读取磁盘**
  - 按照上一步得到的排序后的列表根据行指针获取出排序后的记录并返回给用户

#### 单路排序

> **只涉及到一次磁盘`IO`**

- 首先将我们指定的数据表的所有记录的所有会使用到的所有字段全部读取到`Sort Buffer`
- 然后直接对`Sort Buffer`中的数据进行排序

#### **注意事项**

- **我们应该尽量使得`sort_buffer_size`充分大,这样才能应对使用单路排序时待排序记录所占空间过大的情况,避免多次`IO`使用磁盘辅助排序**
- **我们应该尽量让`max_length_for_sort_data`充分大,这样才能让我们的`MySQL`尽量使用单路排序**
- **我们的`SELECT`尽量不要用`SELECT *`因为这样使得我们所需呈现的记录的字段过多,占据的空间会很大,从而导致非常容易超出`max_length_for_sort_data`使得无法使用单路排序,或满足`max_length_for_sort_data`但记录的总占用空间大于`sort_buffer_size`从而导致系统使用磁盘来多次`IO`辅助查询**

## `GROUP BY`优化

- GROUP BY`使用索引的原则几乎跟`ORDER BY`一致 ，`GROUP BY`即使没有过滤条件用到索引，也可以直接使用索引。
- `GROUP BY`**先排序再分组**，遵照索引建的最佳左前缀法则
- 当无法使用索引列，可以增大`max_length_for_sort_data`和`sort_buffer_size`参数的设置
  - **因为`GROUP BY`要进行`FileSort`排序,所以扩大这两个参数有利于`FileSort`**
- `WHERE`效率高于`HAVING`，能写在`WHERE`限定的条件就不要写在`HAVING`中了
  - **如`Having name="汤凌"`完全可以改写为`WHERE name="汤凌"`这样过滤掉了非`name="汤凌"`的字段,所以得到的所以分组都会可以满足`HAVING name="汤凌"`这个条件**
- 减少使用`ORDER BY`，和业务沟通能不排序就不排序，或将排序放到程序端去做。`ORDER BY`、`GROUP BY`、`DISTINCT`这些语句较为耗费`CPU`，数据库的`CPU`资源是极其宝贵的。
- 包含了`ORDER BY`、`GROUP BY`、`DISTINCT`这些查询的语句，`WHERE`条件过滤出来的结果集请保持在`1000`行以内，否则`SQL`会很慢。

## `LIMIT`分页查询优化

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719191930891.png" alt="image-20220719191930891" style="zoom:60%;" />

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719191940950.png" alt="image-20220719191940950" style="zoom:70%;" />

## 重点注意事项

## 26 索引使用的注意事项

- **一个表的索引不要超过`6`个**
- **尽量避免重复的索引的建立**
- **尽量避免使用`SELECT *`**
  - 这样一般**无法利用到覆盖索引**
  - `MySQL`在解析的过程中,需要通过查询系统维护的`数据字典`来将``*``按序转换成所有列名，这会大大的耗费资源和时间。
  - 在涉及`ORDER BY`或`GROUP BY`时会影响到`单路排序`的使用
  - 在涉及到`JOIN`时可能会影响`Block Neasted-Loop Join`的使用
- **要善于使用`LIMIT`分页管理**
  - 考虑这样一种情况,我们有一张学生信息表,其中学生名字列不是`UNIQUE`约束字段并且没有建立索引,但是我们作为老师知道目前为止数据库中不存在重名的学生,那么对于`SELECT * FROM Students WHERE name="汤凌"`我们就可以加上`LIMIT 1`来提高查询效率
  - 若我们不添加`LIMIT 1 `那么我们的查询语句就一定会遍历表中的每一条记录来完成我们的查询
  - 但是如果添加了`LIMIT 1`那么当遍历时查到了一条`name="汤凌"`的记录,那么查询语句就会由于`LIMIT 1`的存在而直接终止查询,直接返回这一条记录.显然这样就节约了一定时间
- **要养成多使用`COMMIT`的习惯**
- **结合实际情况使用`COUNT(*)|COUNT(1)|COUNT(具体字段)`**
  - `MyISAM`下由于引擎给每个表维护了`row_num`值因此`COUNT(*)`明显会快于`COUNT(1)`,但`Innodb`下则相差不大
  - 在`Innodb`下使用`COUNT(具体字段)`尽量保证该字段有非聚簇索引,因为`Innodb`下数据是完全存储在聚簇索引文件中的,因此如果涉及到聚簇索引的使用就会导致极大的内存与磁盘`IO`开销
- **综合情况选择`IN`与`EXIST`的使用**
  - `IN`是把外表和内表作``HASH连接``，而``EXISTS``是对外表作``Loop循环``，每次``Loop循环``再对内表进行查询，一直以来认为`EXISTS`比`IN`效率高的说法是不准确的。
  - 如果查询的两个表大小相当，那么用``IN``和`EXISTS`差别不大；如果两个表中一个较小一个较大，则子查询表大的用`EXISTS`，子查询表小的用`IN`；
- **尽量用多表查询替代子查询**
  - 对于涉及子查询的`SQL`语句,``MySQL``需要**为内层查询语句的查询结果`建立一个临时表`**，然后**外层查询语句从临时表中查询记录**。查询完毕后，再`撤销这些临时表`,这样会消耗过多的CPU和IO资源;并且这样的零时表是必然不具备索引的,因此这些综合起来就会导致**产生大量的慢查询**。
- **尽量避免使用`NOT IN`与`NOT EXISTS`**
  - 

## ==**27 `MySQL`各个关键字使用索引的优先级**==

> **前提**:**`MySQL`优化器会在选取使用的索引时尽量使得`SQL`语句中的各个子句可以尽量使用上索引.**

```mysql'
ON
#大于
WHERE
#大于
GROUP BY
#大于
HAVING
#大于
ORDER BY
```

## 28 索引条件下推(`ICP`)

> **索引条件下推是否开启由`optimizer_switch`系统变量的`index_condition_pushdown`参数决定**

### 索引下推的特性

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\MySQL重要原理汇总.assets\image-20220719130634915.png" alt="image-20220719130634915" style="zoom: 65%;" />

### 使用前提

- **`EXPLAIN`获取到的`SQL`语句的`type`参数为`range|ref|eq_ref|ref_or_null`**
- **表使用的引擎为`Innodb`或`MyISAM`**
- **`SQL`语句不使用覆盖索引,因为在使用覆盖索引的情况下进行索引条件下推大部分情况下不如直接读取出索引中的字段进行操作**
- **相关子查询的条件语句(`WHERE|HAVING|ON`)**

## 29 覆盖索引

- **覆盖索引**:索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据；当能通过读取索引就可以得到想要的数据，那就不需要读取行了。**一个索引包含了满足查询结果的数据就叫做覆盖索引。**

### 覆盖索引的优缺点

#### 优点

- **避免Innodb表进行索引的二次查询（回表）**

- **可以把随机IO变成顺序IO加快查询效率**

#### 缺点

- **`索引字段的维护`总是有代价的。因此，在建立冗余索引来支持覆盖索引时就需要权衡考虑了。这是业务DBA，或者称为业务数据架构师的工作。**

## 30 `WHERE a=常量 AND b>常量 ORDER BY b,c`在`INDEX(a,b,c)`的情况下`ORDER BY`可以使用到索引

## 31数据库设计

### 前置知识

- **`超键`：即能够唯一地标识一条记录的任意索引字段组合**
- **`候选键`：即所有超键中字段数最小的超键**
- **`主键`：我们自己从候选键中选取的一个索引字段组合**
- **`外键`：受到外键约束的字段**
- **`主属性`：出现在任意一个候选键中的任意一个字段**
- **`非主属性`：没有出现在任何一个候选键中的任意一个字段**

### 数据库范式

- `1NF`:数据表的任意一个字段都必须满足原子性
- `2NF`:第二范式要求，在满足第一范式的基础上，还要**满足数据表里的每一条数据记录，都是可唯一标识的。而且所有非候选键字段，都必须完全依赖于任意一个候选键，不能只依赖任意一个候选键的一部分。**如果知道候选键的所有属性的值，就可以检索到任何元组（行）的任何属性的任何值。
  - **完全依赖即选取任意一个候选键中字段的任意组合,只要字段数少于该候选键就一定无法唯一标识数据表的每一条记录**
- `3NF`:第三范式是在第二范式的基础上，确保数据表中的每一个非主键字段都和主键字段直接相关，也就是说，**要求数据表中的所有非主键字段不能依赖于其他非主键字段。**（即，不能存在非主属性A依赖于非主属性B，非主属性B依赖于主键C的情况，即存在"A-->B-->C"的决定关系）通俗地讲，该规则的意思是所有`非主键属性`之间不能有依赖关系，必须`相互独立`。
  - **即不能存在类似于存在两个非主属性的字段``A,B``,当我们知道`A`的值时就能知道`B`的值这样的情况**
- `BCNF`:在3NF的基础上消除了主属性对与主属性的部分依赖或者传递依赖关系。
  - **即若存在任意两个主属性`A,B`存在知道主属性`A`的取值就能得到主属性`B`的取值的情况,那么我们的数据表就不符合`BCNF`**

### `ER`模型

### 基本组成

- **实体**:`ER`模型中实体用`矩形`表示,如某个地区的超市,学生,老师等都是实体
- **属性**:即实体的特性,如学生的名字,年龄,老师的住址,教授的课程等
- **关系**:即实体与实体之间的联系,如超市卖产品给顾客,那么超市与顾客就是买卖关系.

### 关系的分类

- **一对一关系**:实体与实体的关系是一一对应的,**如一个人在某一个时刻只能有一个伴侣**
- **一对多关系**:实体与实体是一对多的,**如一个人可以有多个孩子,但一个孩子只能有一个父亲**
- **多对多关系**:实体与实体是多对多的,**如顾客可以去超市买多个商品,而超市可以卖商品给多个顾客**