## `                                                                                                               MySQL`数据库系统初始自带数据库与数据表剖析

### `mysql`数据库

> `mysql`数据库存储了 `MySQL` 服务器正常运行所需的各种系统信息，包含了关于数据库对象元数据（``metadata``）的数据字典表和系统表。
>
> 从 `MySQL 8.0` 开始，``mysql`` 系统表和数据字典表使用 `InnoDB` 存储引擎，存储在 `MySQL` 数据目录下的 `mysql.ibd` 表空间文件中。在 `MySQL 5.7` 之前，这些系统表使用 `MyISAM` 存储引擎，存储在 `mysql` 数据库文件目录下各自的表空间文件中。

#### 权限信息表

> 这些系统表存储了用户账户的授权信息以及它们拥有的权限。
>
> 从 `MySQL 8.0` 开始，这些权限表使用事务型的 `InnoDB` 存储引擎替代了之前的 `MyISAM` 存储引擎。存储引擎的改变也带来了账户管理行为的变化。例如之前的 `CREATE USER` 和 `GRANT` 语句如果同时操作多个用户，可能导致部分用户操作成功而其他用户操作失败；现在这些操作具有事务性，要么全部用户都操作成功，要么出现错误回滚所有的操作。
>
> `MySQL 8.0` 中的权限信息表如下：

- `user`：用户账户、全局权限以及其他信息。
- `global_grants`：用户的动态全局权限。
- `db`：数据库级别的权限。
- `tables_priv`：表级别的权限。
- `columns_priv`：字段级别的权限。
- `procs_priv`：存储过程和函数上的权限。
- `proxies_priv`：代理用户权限。
- `default_roles`：用户连接并认证后默认激活的角色，或者执行 SET ROLE DEFAULT 命令后设置的角色。
- `role_edges`：角色的授予关系。user 表中的一行数据既可能代表一个用户账户，也可能代表一个角色。
- `password_history`：密码修改历史。

### 对象信息表

> 这些系统表包含了关于存储过程、组件、用户定义函数以及服务器端插件的信息

- `component`：服务器组件的注册信息。

- `func`：用户自定义函数（UDF）信息。
- `plugin`：服务器端插件信息。

### 查询日志表

- `general_log`：通用查询日志表。
- `slow_log`：慢查询日志表。
- 这些日志表的存储引擎为`CSV`。

#### 服务器端帮助信息表

> 这些表中存储了服务器端相关的帮助信息

- `help_category`：帮助信息分类。

- `help_keyword`：帮助信息关键字。
- `help_relation`：关键字和帮助主题之间的关系。
- `help_topic`：帮助主题的具体内容。

#### 时区信息表

> 这些系统表包含了时区相关的信息

- `time_zone`：时区 ID 以及是否包含闰秒。

- `time_zone_leap_second`：闰秒发生的情况。
- `time_zone_name`：时区 ID 和名称的映射。
- `time_zone_transition、time_zone_transition_type`：时区描述。

#### 复制信息表

> `MySQL` 服务器使用这些表维护复制功能

- `gtid_executed`：存储 `GTID` 数据。

- `ndb_binlog_index`：`` NDB`` 集群复制的二进制日志信息。
- `slave_master_info、slave_relay_log_info、slave_worker_info`：在从服务器上存储复制信息。

#### 优化器系统表

> 这些系统表会被优化器使用

- `innodb_index_stats、innodb_table_stats`：``InnoDB`` 优化器持久性统计信息。

- `server_cost、engine_cost`：优化器成本模型需要使用这些表中存储的各种操作的评估成本进行优化。
- ``server_cost`` :包含了通用服务器操作的优化器成本估计
  
- `engine_cost`: 包含了特定存储引擎操作的优化器成本估计。

#### 其他系统表

- `audit_log_filter、audit_log_user`：如果安装了 `MySQL Enterprise Audit`，这些审计日志表中会存储关于审计日志过滤器和审计的用户账户信息。

- `firewall_users、firewall_whitelist`：如果安装了 `MySQL Enterprise Firewall`，这些表中会存储企业防火墙使用的信息。

- `servers`：`FEDERATED `存储引擎使用的远程服务器连接信息。

- `innodb_dynamic_metadata`：`InnoDB` 存储的快速变化的元数据，例如 `auto-increment` 计数值和索引损坏标识。该表用于替代 `InnoDB` 系统表空间中的数据字典缓冲表。

### `information_schema`数据库

### `performance_schema`数据库

> - `performance_schema`性能数据库为``MySQL``服务器的运行时状态提供了一个底层的监控功能。``MySQL``默认启动了性能数据库，也可以在启动服务时通过参数``performance_schema`指定是否启用
>
> - 性能数据库中的表的存储引擎为 `PERFORMANCE_SCHEMA`，数据存储在内存中
>
> - `MySQL`服务每次启动时都会重新初始化性能数据库
>
>   - ```mysql
>     show variables like 'performance_schema';#查看是否成功初始化
>     ```

> `performance_schema`数据库中的表可以按照收集信息的类型分成不同的组：

- **设置表**，显示和修改监控配置。这些表的名称都以``setup_``开始，例如``setup_objects``表存储了需要进行监控的对象。

- **当前事件表**,``events_waits_current``表包含了每个线程最新的等待事件。其他类似的表包含了不同级别的等待事件:``events_stages_current`代表每个线程当前执行阶段的事件,`events_statements_current` 代表了当前语句事件，`events_transactions_current` 代表了当前事务事件。
- **历史事件表**，这些表的结构和当前事件表相同，但是包含了更多的历史数据。例如 `events_waits_history` 表包含了每个线程最近的 `10` 个等待事件，``events_waits_history_long`` 表包含了所有线程最近的 `10000 `个事件。阶段事件、语句事件以及事务事件也存在类似的历史事件表。
- **事件汇总表**，包含了按照不同事件分组汇总的信息，包括已经从历史事件表中移除的事件。例如，``events_waits_summary_by_instance` 代表了每个监测实例的等待事件汇总。
- **监测实例表**，记录了被检测的对象类型。每个监测对象会产生一个事件，这些表存储了事件名称和解释性说明或者状态信息。例如，``file_instances` 表存储了 `I/O `监测涉及到的文件。
- **其他表**，例如 `threads` 表包含了每个线程的信息。

### `sys`数据库

> - 其实际上存储的是大量的视图,而这些视图的基表来自于`information_schema`与`performance_schema`两个数据库
> - `sys`数据库下的表一般是成对存在的由`X$表名`和`表名`组成,其中`表名`指代的表更易于人类查看,而`X$表名`指代的表的计时单位为皮秒,其存在价值在于提供给其他工具对数据进行处理



## `Linux`下`MySQL`的文件的存在情况以及其用途

### 数据相关

- **`/var/lib/mysql`**

> 类似于`Windows`中`MySQL`的`Data`文件夹,其内存放着我们各个数据库的数据表中的数据等内容

### 指令相关

- **`/usr/bin`**

> 其中存储着大量的`MySQL`指令

-  **`/usr/sbin`**

> 其中存储着大量的`MySQL`指令

### 配置文件相关

-  **`/usr/share/mysql`**

- **`/usr/share/mysql-common`**

-  **`/etc/mysql`**

## `MySQL`字符集剖析

### 不同选项的含义

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6?token=AOAPFCJ2ZVVXM7X2DGVYMA3C57XKE" alt="image-20220711142617890" style="zoom:75%;" />

![image-20220711144230495](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-2?token=AOAPFCI72YSQ2UY7VWQCAQ3C57XOQ)

- `character_set_client`:服务器编解码请求时使用的字符集
- `character_set_connection`:服务器处理请求时会把请求字符串从`character_set_client`转为`character_set_connection`
- `character_set_database`:服务器在创建数据库时,数据库默认使用的字符集
- `character_set_filesystem`:
- `character_set_results`:服务器对请求客户端进行结果返回时,结果字符串使用的字符集
- `character_set_server`:
- `character_set_system`:
- `character_sets_dir`:

### 一般字符集(如`utf8mb4`)与比较字符集(如`utf8mb4_0900_ai_ci`)

- **后缀(如`_ai,_ci`)**

![image-20220711143200471](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-3?token=AOAPFCPU2AQVUSH2XFHBRALC57XNY)

- **查看常用字符集**

```mysql
SHOW CHARACTER SET;
```

- **查看常用比较字符集**

```mysql
SHOW COLLATION;
```

- **修改字符集以及比较字符集**

```shell
#修改字符集
CAHRACTER SET 字符集名
#修改比较字符集
COLLATE 比较字符集名
```

## `MySQL`大小写

> **注意**在`Linux`下`MySQL`遵循如下大小写规范
>
> - 数据库名,表名,表别名,变量名是严格区分大小写的
> - 关键字与函数名在`SQL`语句中是不区分大小写的
> - 字段名与字段别名是不区分大小写的

### `lower_case_table_names`大小写规则参数

```mysql
SHOW VARIABLES LIKE '%lower_case_table_names%';
```

| 参数 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| `0`  | 全局大小写敏感                                               |
| `1`  | 大小写不敏感,所有语句都会全部转为小写后执行                  |
| `2`  | 创建表与数据库的语句严格按照创建时语句的大小写进行,但查找语句全部转为小写进行 |

### 大小写使用建议

> - 关键字与函数名均使用大写
> - 数据库名,表名,字段名,表别名,字段别名等用户自定义属性使用小写 

## `SQLModel`系统变量

## `MySQL`数据库文件

> ``/var/lib/mysql``

> `.ibd`:独立表空间
>
> `ibdata1`:系统表空间
>
> `MySQL 8.0`不再使用`.opt`文件

### `innodb`

#### `MySQL 5.7`

- `.opt`:存储对应数据库的默认字符集,默认引擎等配置信息
- `.frm`:存储对应表的结构与索引
- `.ibd`:默认情况下存储对应表的数据
- **注意**
  - **`MySQL 5.6.6`之前的版本数据表的文件都默认存在系统表空间,而之后则默认存储在独立表空间**
  - 我们的表数据可以经过设置保存在`/var/lib/mysql/ibdata1`文件中

#### `MySQL 8.0`

> 在`MySQL 8.0`下不再由`.frm`独立存储表结构与索引,也不再由`.opt`存储数据库的默认设置,而是直接合并在`.ibd`	文件中一并存储,

#### 修改默认存储位置

> `SHOW VARIABLES LIKE "innodb_file_per_table"`:查看当前存储位置

```shell
#临时配置
SET @@global.innodb_file_per_table = 0;

#永久设置
#在my.cnf文件中
[server]
innodb_file_per_table=0 #0:代表系统表空间,1:代表独立表空间
```

### `MyISAM`

#### `MySQL 5.7`

- `.frm`:存储表结构
- `.MYD`:存储表数据
- `.MYI`:存储表索引

#### `MySQL 8.0`

- `.sdi`:存储表结构
- `.MYD`:存储表数据
- `.MYI`:存储表索引

## `MyISAM`与`Innodb`的区别

- 在`Innodb`中数据表的索引是和表结构一同存储在`.frm`或`.ibd`文件中的,但在`MyISAM`中索引与表结构是分开存储的,`.MYI`存储表索引,`.frm`存储表结构

## `MySQL`用户创建与基本设置

### 登陆`MySQL`服务器

![image-20220711160110363](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-4?token=AOAPFCPDQA5AHH65X54USJTC57XQM)

### 创建新用户

> **注意**:`mysql.user`表的唯一性约束是以`host与user`两个字段组成的,因此同时拥有`'tl'@'localhost'`与`'tl'@'%'`两个字段是被允许的

> 在初始状态下`MySQL`自带以下用户(`SELECT user FROM mysql.user;`)
>
> - `root`
> - `debian-sys-maint`
> - `mysql.infoschema`
> - `mysql.session`
> - `mysql.sys`

- **创建一个新用户**

  ```mysql
  CREATE user '用户名' IDENTIFIED BY '初始密码'
  #等价于CREATE user '用户名'@'%' IDENTIFIED BY '初始密码'
  
  CREATE user '用户名'@'主机IP' IDENTIFIED BY '初始密码'
  #如CREATE user '用户名'@'localhost' IDENTIFIED BY '初始密码'
  
  #主机名用于设置user表的host字段
  #用户名用于设置user表的user字段
  ```

- **修改用户信息**

  > 实际上修改用户信息就是对``user``表中该用户对应的`字段`进行修改

  ```mysql
  #修改用户名
  UPDATE user SET user="新用户名" WHERE user='旧用户名'
  #修改用户host
  UPDATE user SET host='主机IP' WHERE user='用户名'
  ```

- **删除用户**

  ```mysql
  #系统维护方式
  DROP USER '用户名';#等价于DROP USER '用户名'@`%`
  DROP USER '用户名'@'主机IP';
  
  #删除user表字段方式(不推荐)
  DELETE FROM user WHERE user='用户名';
  flush privileges;
  
  #两种方式的区别在于第一种自带flush privileges
  ```

### 用户密码的修改

> **注意**:``MySQL5.7``之后是可以对密码强度进行规定的,因此我们设置的密码必须符合我们设置的密码强度

- **修改当前用户密码**

  ```mysql
  #在MySQL5.7
  SET PASSWORD = PASSWORD('密码');
  UPDATE user SET authentication_string = PASSWORD('密码') WHERE user='用户名';
  #在MySQL8.0
  SET PASSWORD = SHA('密码');
  SET PASSWORD = MD5('密码');
  UPDATE user SET authentication_string = SHA('密码') WHERE user='用户名';
  UPDATE user SET authentication_string = MD5('密码') WHERE user='用户名';
  #MySQL官方推荐
  ALTER user USER() IDENTIFYIED BY '密码'
  ```

- **修改指定用户密码**

  ```mysql
  #在MySQL5.7
  SET PASSWORD = PASSWORD('密码') FOR '用户名';
  UPDATE user SET authentication_string = PASSWORD('密码') WHERE user='用户名';
  #在MySQL8.0
  SET PASSWORD = SHA('密码') FOR '用户名';
  SET PASSWORD = MD5('密码') FOR '用户名';
  UPDATE user SET authentication_string = SHA('密码') WHERE user='用户名';
  UPDATE user SET authentication_string = MD5('密码') WHERE user='用户名';
  #MySQL官方推荐
  ALTER user '用户名'@'主机IP' IDENTIFYIED BY '密码'
  ```

### `MySQL`用户密码过期策略与重用策略

#### 密码过期策略

> **用于设置多久之后必须重新设置密码**

- **设置指定用户账号过期**

  > **过期后可以登陆,可以修改密码,但不能对数据进行操作**

  ```mysql
  ALTER USER '用户名'@'主机IP' PASSWORD EXPIRE
  ```

- **设置全局密码过期策略**

  > `MySQL`使用`default_password_lifetime`系统变量来维护密码过期策略
  >
  > - 值得注意的是,若我们以前设置**全局密码过期策略为不过期**,那么**以前创建的用户的密码还是不过期,**只不过**设置了**全局密码过期策略为指定时间过期**后**,**我们创建的用户的密码就会过期**
  > - 如果我们**使用了临时设置**,那么设置后创建的用户的密码就**会过期**,但是当`MySQL`服务**重启后**创建的用户的密码就**不会过期**了

  ```mysql
  #临时设置,一旦mysql服务重启便失效
  #0:密码不过其,N:密码N天后会过期,过期后必须修改密码才可以继续访问数据
  SET PERSIST default_password_lifetime = N
  
  #永久设置,my.ini或my.cnf
  [mysqld]
  default_password_lifetime=N 
  ```

- **设置指定用户的密码的过期策略**

  ```mysql
  #设置90天过期
  CREATE user '用户名'@'主机IP' IDENTIFIED BY 密码 PASSWORD EXPIRE INTERVAL 90 DAY;
  ALTER user '用户名'@'主机IP' PASSWORD EXPIRE INTERVAL 90 DAY;
  
  #设置永不过期
  CREATE user '用户名'@'主机IP' IDENTIFIED BY 密码 PASSWORD EXPIRE INTERVAL NEVER;
  ALTER user '用户名'@'主机IP' PASSWORD EXPIRE INTERVAL NEVER;
  
  #设置为默认
  CREATE user '用户名'@'主机IP' IDENTIFIED BY 密码 PASSWORD EXPIRE INTERVAL DEFAULT;
  ALTER user '用户名'@'主机IP' PASSWORD EXPIRE ITERVAL DEFAULT;
  ```

#### 密码重用策略

> **密码重用策略设定了用户修改密码时新密码是否可以与以前的密码相同**,具体分为两类
>
> - `基于数量`:设置一个值`N`,用户的新密码不能与最近`N`次密码相同
>   - `password_history`
> - `基于时间`:设置一个天数`N`,用户的新密码不能与最近`N`天内使用过的密码相同
>   - `password_reuse_interval`

- **设置全局密码过期策略**

  > 设置为`0`时表示无限制

  ```mysql
  #临时设置
  SET PERSIST password_history = N;
  SET PERSIST password_reuse_interval = N;
  
  #永久设置
  [mysqld]
  password_history = N
  password_reuse_interval = N
  ```

- **设置指定用户的密码重用策略**

  ```mysql
  #设置5次
  CREATE user '用户名'@'主机IP' IDENTIFIED BY 密码 PASSWORD HISTORY 5;
  ALTER user '用户名'@'主机IP' PASSWORD HISTORY 5;
  
  #设置365天
  CREATE user '用户名'@'主机IP' IDENTIFIED BY 密码 PASSWORD REUSE INTERVAL 365 ;
  ALTER user '用户名'@'主机IP' PASSWORD REUSE INTERVAL 365;
  
  #同时设置密码过期与密码重用,180天过期,不能与上5次相同,不能与365天内使用的相同
  CREATE user '用户名'@'主机IP' IDENTIFIED BY 密码
  PASSWORD EXPIRE INTERVAL 180
  PASSWORD REUSE INTERVAL 365
  PASSWORD HISTORY 5;
  
  ALTER user '用户名'@'主机IP'
  PASSWORD EXPIRE INTERVAL 180
  PASSWORD REUSE INTERVAL 365
  PASSWORD HISTORY 5;
  ```

## `MySQL`用户权限管理

> **权限基本分为三个级别**
>
> ![image-20220711172026881](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-5?token=AOAPFCIJ27JW5FZUYTA2F2DC57XRA)

#### 权限授予基本原则

![image-20220711172037647](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-6?token=AOAPFCJ5PT22YCERLCOWESDC57XRS)

#### 用户权限相关的信息的存储位置

#### 权限基本操作

- **查看`MySQL`所有的权限**

  ```mysql
  SHOW privileges;
  ```

- **查看当前用户所具有的权限**

  ```mysql
  SHOW GRANTS;
  ```

- **查看指定用户所具备的权限**

  ```mysql
  SHOW GRANTS FOR '用户名'@'主机IP'
  ```

#### 常见权限示例

- `ALL PRIVILEGES`:所有权限
- `SELECT`:查询权限
- `UPDATE`:更新权限
- `DELETE`:删除权限
- `INSERT`:插入权限
- `ALTER`:修改权限
- **…**

#### 权限授予(`GRANT`)

> `WITH GRANT OPTION`是用于授予该用户授予别的用户权限的权限的语句(**当然其也只能将自己拥有的权限授予给其他用户,自己没有的权限是无法授予给别的用户的**)
>
> **注意**
>
> - 可以对**同一个用户多次赋予权限**,并且**曾经赋予的**权限**不会被取消**,是可以**叠加赋予**的
> - **在用户登陆期间赋予的权限只有用户重新登陆后才会具备**

- **基本语法**

  ```mysql
  GRANT 权限 ON 数据库名.数据表名 TO '用户名'@'主机IP' [WITH GRANT OPTION];
  #WITH GRANT OPTION是用于授予该用户授予别的用户权限的权限的语句
  ```

- **给用户授予所有权限**

  ```mysql
  #给用户赋予了在任意数据库的任意数据表下的所有操作的权限
  GRANT all privileges ON *.* TO '用户名'@'主机IP' [WITH GRANT OPTION];
  ```

#### 权限回收(`REVOKE`)

> **注意**
>
> - 收回前必须保证该用户有`SYSTEM_USER`权限
> - **在用户登陆期间收回的权限只有用户重新登陆后才会真正收回**

- **基本语法**

  ```mysql
  REVOKE 权限 ON 数据库名.数据表名 FROM '用户名'@'主机IP'
  ```

- **收回用户所有权限**

  ```mysql
  REVOKE ALL PRIVILEGES ON *.* FROM '用户名'@'主机IP'
  ```

## `MySQL`用户权限相关的表(`mysql`数据库下)

### `mysql.user`表

#### `user`表

- `user`:存储用户

- `host`:存储可以访问该用户的主机`IP`

- `authentication_string`:存储该用户的密码

- **用户权限相关列**

  <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-7?token=AOAPFCP3HFD5KXJU2QGJPZ3C57XSE" alt="image-20220711215753035" style="zoom: 80%;" />

  ![image-20220711222222509](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-8?token=AOAPFCLPYTICNXWATKUIRDLC57XS4)

- `password_lifetime`：用于在密码过期策略生效情况下，存储该用户的密码有效时长

- `password_last_changed`:用于存储该用户密码最近一次修改密码的时间

- `Password_expired`:用于存储该用户是否密码过期

- `plugin`:使用的密码加密方案

  - `mysql_native_password`:
  - `caching_sha2_password`:

- `authentication_string`:加密后的密码

- `account_locked`:

- `Password_reuse_history`:

- `Password_reuse_time`:

- `Password_require_current`:

- `User_attributes`:

- `max_questions`:标识用户每小时最多允许查询的次数

- `max_updates`:标识用户每小时最多更改的次数

- `max_connections`:标识用户每小时最多允许的连接次数

- `max_user_connections`:标识同时最多连接的用户的个数

### `mysql.db`表

> **存储数据库级别的权限**

### `table_priv`表

> **表级权限**

### `column_priv`表

> **列级权限**

### `proc_priv`表

> **过程权限**

## `MySQL`访问控制

> **  **

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-9?token=AOAPFCLVR6QOELNL4Q45G5DC57XTU" alt="image-20220711233256995" style="zoom:80%;" />

## `MySQL`角色(`ROLE`)管理

> **角色的概念**
>
> - **角色**可以理解为**一个权限的集合**,**一个虚拟的用户.**
>
> - **角色可以被添加或移除权限.**
> - 实际的**用户可以被授予一个角色**,并且当该用户**获得这一角色的同时**,也就**获得了这个角色所具备的所有权限**.
> - **同一个角色**可以赋予**多个用户**
>
> **引入角色的优势**:
>
> - 我们可以更少地关心某个特定用户所具备的权限,转而将大部分精力都用于对角色的权限的控制上,某个角色的权限改变,那么扮演该角色的所有用户的权限也会随之改变
>
>   <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-10?token=AOAPFCORAUANH4OGFSUAE3LC57XUO" alt="image-20220712000300966" style="zoom:80%;" />

### 角色初始化操作

#### 角色的创建

- **基本语法**

  ```mysql
  CREATE ROLE '角色名'@'主机IP'[,'角色名2'@'主机IP2'...]
  ```

#### 角色权限的授予

- **基本语法**

  ```mysql
  GRANT 权限 ON 数据库名.数据表名 TO '角色名'@'主机IP' [WITH GRANT OPTION]
  ```

#### 角色权限的查看

- **基本语法**

  ```mysql
  SHOW GRANTS FOR '角色名'@'主机IP'
  ```

#### 角色权限的收回

- **基本语法**

  ```mysql
  REVOKE 权限 ON 数据库.数据表 FROM '角色名'@'主机IP'
  ```

#### 删除角色

- **基本语法**

  ```mysql
  DROP ROLE '角色名'@'主机IP'
  ```

**查看当前用户具备的角色**

- **基本语法**

  ```mysql
  SELECT CURRENT_ROLE();
  ```

### 角色的使用

> 

#### 给用户赋予角色

- **基本语法**

  ```mysql
  GRANT '角色名'@'主机IP' TO '用户名'@'主机IP'
  ```

#### 激活角色

> **注意**：·``MySQL``中角色默认情况下是不激活的,因此如果一个用户被赋予了一个未激活的角色,那么无论是`SELECT CURRENT_ROLE()`还是具体的操作上,其都不会具有该角色的任何权限.我们必须要手动激活角色,才能够正确投入使用

- **方式1**

  ```mysql
  SET DEFAULT ROLE ALL TO '用户名'@'主机IP',['用户名1'@'主机IP1'...]
  ```

- **方式2**

  ```mysql
  SET GLOBAL avtivate_all_roles_on_login = ON;
  ```

#### 切换角色

#### 撤销用户的角色

- **基本语法**

  ```mysql
  REVOKE '角色名'@'主机IP' FROM '用户名'@'主机IP' 
  ```

#### 设置强制角色

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

##  ==**`MySQL`配置文件使用**==

### 配置文件基本组成

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-11?token=AOAPFCLEQLFLL7DXAMGTXKLC57XVE" alt="image-20220712104041082" style="zoom: 50%;" />

### 配置文件中各个选项的用途

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-12?token=AOAPFCI3OT46JZSRAATAYS3C57XV2" alt="image-20220712104009564" style="zoom:50%;" />

### 特定版本下的选项组

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-13?token=AOAPFCK4EG6XQBRX2V6TQQDC57XWI" alt="image-20220712104255142" style="zoom:50%;" />

### 选项组的优先级关系

> **这里的优先级关系是指,若两个选项组对同一个系统变量进行了设置,并且``mysql``命令会同时访问这两个选项组,那么最终该系统变量会遵循哪一个选项组下的设置,遵循谁,谁的优先级就高.**

- **基本规则**:若在配置文件中,我们的选项组A的位置相比于选项组B更靠近句首,那么选项组A的优先级就会更高,反之若选项组B更靠近句首,那么选项组B优先级更高
- **注意**:我们在调用`MySQL`服务命令时,是可以在命令行通过后缀`--系统变量名1=取值1 [--系统变量名2=取值2]`的方式给系统变量名赋值的,并且此时,我们的系统中该命令行的赋值优先级高于我们的配置文件,不过要注意的是,``MySQL``命令不会去同步修改我们的配置文件,因此如果重启`MySQL`服务,那么在我们不加命令行指定的情况下,其对应的系统变量就会以配置文件中的为准.

## `MySQL`系统变量设置

### 基于命令行的`MySQL`系统变量设置

```mysql
--系统变量名1=取值1 [--系统变量名2=取值2]
```

### 基于配置文件的`MySQL`系统变量设置

## `MySQL`服务器逻辑架构

### 逻辑架构剖析

> **前提**
>
> - `MySQL`的基础服务是典型的`C/S`架构,即客户端,服务端结构.而客户端与服务端在这里指的并不是一台机器,而是一个运行中的计算机进程
> - `MySQL`服务的最基本流程为`,**客户端进程向服务端进程发送一段SQL语句,服务端在接收并处理该SQL语句后,将最终结果返回给客户端进程.**

#### `SQL`语句处理的基础流程结构图

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-14?token=AOAPFCL74JEAE5AI4G6FB43C57XXI" alt="image-20220712111544733" style="zoom: 67%;" />

- **处理连接**:
- **查询缓存**:
- **语法解析**:
- **查询优化**:

#### `MySQL 5.7`的`SQL`处理服务具体组成架构

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-15?token=AOAPFCNYBOMYLCJDP72X3ETC57XXY" alt="image-20220712111737970" style="zoom: 50%;" />

##### 外部结构

- `Connectors`:**连接器**,用于向客户端上不同的编程语言提供数据库服务``API``

##### 连接层

- `Connection Pool`:**海量客户端连接管理器**,运行于服务器上,其由多个`MySQL`服务器进程的线程组成,每一个线程都能够且同时只能够连接到一个客户端.通过该管理器良好地管理了客户端进程与服务器端进程的交互问题.当某个线程的连接中断后,其会恢复可用状态,供我们的管理器分配给后续的客户端进程

##### 服务层

- `SQL Interface`:**SQL接口**,接收``SQL``指令返回查询结果
- `Parser`:**`SQL`语句解析器**,对``SQL``语句进行语法与语义解析生成语法树,我们的查询过程需要借助这一数据结构来进行.如果我们的`SQL`语句存在错误,就会在这一步被解析发现并向用户报错
- `Optimizer`:**`SQL`语句优化器**,用于对我们的`SQL`语句进行优化
- ``Caches$Buffers``:**`SQL`语句存储缓存器**,以`Key-Value`的结构保存我们曾经使用过的查询语句,以及其对应的结果,当我们客户端进程发出请求给我们的服务器端时,如果该请求通过``键值对``能查询到该存储器中的结果,那么`MySQL`服务就会以该结果作为我们客户端进程请求的回复.
- `Management Service&Utilitis`:**数据服务管理与调度器**,

##### 引擎层

- `Pluggable Storage Engines`:**插件式存储引擎**,用于根据我们`MySQL`的系统上存储文件进行文件结构分析,与处理,将我们的文件内容合理地组织为数据表,**并提供大量的`API`以让我们的`MySQL`服务器进程可以容易地对数据进行访问**

##### 存储层

- `OS File system`:**操作系统提供的文件系统**,

### `SQL`语句执行流程

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-16?token=AOAPFCIPI5ESDU6FPQNJJCLC57XYI" alt="image-20220712134625543" style="zoom:80%;" />

#### 查询缓存

> 在`MySQL 8.0`中被**移除**,主要原因如下
>
> - 其底层存储模式为键值对格式,以`SQL`语句为键以其返回结果为值.我们利用查询缓存时,单反我们的查询语句多一个字符,就不会成功命中,因此命中概率低
> - 考虑函数调用,如`NOW()`函数每次都会返回最新的时间,因此,任何含有这一类函数的查询都不应该进入查询缓存,这极大地减少了查询缓存的适用范围
> - 并且在底层我们的查询缓存会实时监控每一条查询语句所涉及的表,任何一张该查询语句涉及到的表发生了任何修改,系统都会删除该查询语句对应的键值对

- **按需设置查询缓存的使用**

  ```shell
  [mysqld]
  
  ```

- **查看查询缓存命中率**

  ```mysql
  SHOW VARIABLES LIKE '%Qcache%';
  #Qcache_free_block:查询缓存内存空间上的空闲内存块的个数
  #Qcache_free_memory:查询缓存内存空间上剩余的
  #Qcache_hits:查询缓存的命中率
  #Qcache_inserts:添加到查询缓存中的键值对数目
  #Qcache_lowmem_prunes:由于缓存空间不足而被移除的键值对个数
  #Qcache_not_cached:在我们设置的按需使用时,由于不需要使用查询缓存而没有被缓存的语句数
  #Qcache_total_blocks:查询缓存内存空间上总内存块数
  ```

#### 解析器

> **解析器分为两步工作**
>
> - **语义分析**:其会分析我们语句中每一个词的具体含义,如`SELECT`是查询的关键字,其后跟随的词一般是``字段名``,`FROM`是关键字,其后跟随的词一般是`表名`
> - **语法分析**:其会对我们的`SQL`语句的合法性做出判断.
>
> **若成功解析则会生成语法树**
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-17?token=AOAPFCPGFFTACIZIRJM3WADC57XY2" alt="image-20220712141049837" style="zoom:67%;" />
>
> **解析过程**
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-18?token=AOAPFCMEPQNLV5Y2D6ED4ULC57XZM" alt="image-20220712141419062" style="zoom:67%;" />

#### 优化器

> 一条查询可以有**很多种不同的执行方式**来获得正确的结果,优化器的目的就在于**根据解析器的结果**分析我们的`SQL`语句**找到一个最优的执行方式**.
>
> **优化分为两个阶段**
>
> - **逻辑查询优化**
> - **物理查询优化**

#### 执行器

> 执行器根据我们的优化器给出的执行计划,有规划地调用存储引擎的`API`获取数据并得到查询的返回结果

### `Oracle`中的`SQL`执行流程

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-19?token=AOAPFCNO6JQ2RCRKSIK4WB3C57X2C" alt="image-20220712150043732" style="zoom:67%;" />

- **共享池检查**:其主要作用与`MySQL`的查询缓存基本一致
  - **软解析**:即找到了匹配的查询语句,直接使用该缓存的语句的执行计划
  - **硬解析**:即没找到匹配的查询语句,需要对语句进行解析与优化.

#### 绑定变量

### 数据缓冲池

> - 我们`MySQL`的数据都是存储在我们的磁盘上的,而磁盘相对于高速的`CPU`而言是一个十分低速的设备,因此我们的操作系统一般都会通过`内存`来建立缓冲区,将磁盘上的数据不断地加载到我们的`内存缓冲区`上,由于在缓冲区充足的情况下我们不必等待`CPU`处理完传输过去的数据,可以一直连续不间断地将要使用到的数据从磁盘上导入到我们的``内存缓冲区``,我们知道`内存`是一个高速传输设备,因此通过建立`内存缓冲区`的方式可以极大地节约我们的`SQL`语句执行用时中用于数据`IO`的时间(当然在这个概念下是没有减少``IO``时间的,只不过是将原本串行的程序流程组织为了并行从而缩短了总用时).
> - 在上面的基础上,`MySQL`还建立了内存上的缓冲池,其中持久化地固定存储有大量的数据,当我们查询涉及到对这些数据的提取时,不再需要从`磁盘`进行读取,而是直接从内存读取,直接省略了`磁盘IO`这一步骤.当然值得注意的是,一般而言内存的存储量相对于磁盘而言是极小的,因此我们的`MySQL`系统不能考虑将所有的数据都导入到我们的内存中,而是要通过`数据的重要性=位置*访问频次`来决定一个数据的导入优先级.优先级高的数据才会被导入内存
> - **预读特性**:

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-20?token=AOAPFCOHFCLBKTHP2BZZORDC57X22" alt="image-20220712154223111" style="zoom: 50%;" />

数据缓冲池对数据的缓冲机制

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

### `MySQL`数据更新的同步性问题

> **问题**:我们知道,`MySQL`服务会在内存上维护数据缓冲池,当我们查询中涉及到了存储在缓冲池中的数据时,便会直接使用缓冲池中存储的数据,而不会再去磁盘上读取,并且涉及到更新时,也只会实时更新数据缓冲池内的数据.由此就引发了这样的问题
>
> - **问题1**:**我们数据缓冲池中的数据什么时候刷新到我们的磁盘中**
> - **问题2:如果在数据缓冲池中的数据在刷新到磁盘中之前,服务器进程挂掉了,内存中数据全部被释放了,那么有没有办法解决这一数据丢失的问题?**
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-21?token=AOAPFCIUKTKLU3ZERYBUMM3C57X3M" alt="image-20220712161216222" style="zoom:67%;" />

#### 问题1

> 

### `Innodb`存储引擎下基于页的数据缓存池

## `MySQL`存储引擎

### 基本操作

- **查看系统支持的所有引擎**

  ```mysql
  SHOW engines;
  #Support:是否被当前DMBS支持
  #Comment:引擎的简要介绍
  #Transaction:是否支持事务原子化
  #XA:是否支持分布式事务原子化
  #SavaPoint:保存点
  ```

  ![image-20220712163951695](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-22?token=AOAPFCJJJBFMLVTGMKZP7FTC57X4G)

- **查看当前系统的默认引擎**

  ```mysql
  SHOW VARIABLES LIKE '%storage_engine%';
  ```

### 指定存储引擎

- **修改系统默认存储引擎**

  > **注意**:**临时设置**是**即刻生效**的,而**永久设置**如果是在`MySQL`服务运行期间进行的,则必须在`MySQL`服务**重启后才会生效**

  ```mysql
  #临时设置
  SET DEFAULT_STORAGE_ENGINE = MyISAM;
  #永久设置
  [mysqld]
  default_storage_engine = MyISAM;
  ```

- **修改数据库的默认存储引擎**

  ```mysql
  
  ```

- **修改数据表的默认存储引擎**

  ```mysql
  #创建时指定存储引擎
  CREATE TABLE 表名(字段列表) ENGINE = 引擎名;
  #创建好后修改存储引擎
  ALTER TABLE 数据表名 ENGINE = 引擎名
  ```

### 存储引擎介绍

#### `Innodb`引擎

- `MySQL`从``3.23.34a``开始就包含``InnoDB``存储引擎。 大于等于``5.5``之后，**默认采用``InnoDB``引擎 。**
- ``InnoDB``是``MySQL``的 默认**事务型引擎** ，它被设计用来**处理大量的短期(short-lived)事务**。可以**确保事务**
- **的完整提交(Commit)和回滚(Rollback)**。
- 除了增加和查询外，还需要更新、删除操作，那么，应优先选择``InnoDB``存储引擎。
- **除非有非常特别的原因需要使用其他的存储引擎，否则应该优先考虑``InnoDB``引擎。**
- 数据文件结构：（在《第02章_MySQL数据目录》章节已讲）
  - 表名``.frm`` 存储**表结构**（MySQL8.0时，合并在表名.ibd中）
  - 表名``.ibd`` 存储**数据和索引**
- ``InnoDB``是 为处理**巨大数据量**的最大性能设计 。
- 在以前的版本中，字典数据以元数据文件、非事务表等来存储。**现在这些元数据文件被删除了**。比如： .frm ,``.par`` ，``.trn`` ，``.isl`` ，``.db``,``.opt`` 等都在``MySQL8.0``中不存在了。
- 对比``MyISAM``的存储引擎， ``InnoDB``**写的处理效率差一些** ，并且**会占用更多的磁盘空间以保存数据和索引。**
- **`MyISAM`只缓存索引，不缓存真实数据**；``InnoDB``**不仅缓存索引还要缓存真实数据**， 对内存要求较高 ，而且**内存大小对性能有决定性的影响**  

#### `MyISAM`引擎

- `MyISAM`提供了大量的特性，包括**全文索引、压缩、空间函数(GIS)**等，但``MyISAM`` **不支持事务、行级锁、外键** ，有一个毫无疑问的缺陷就是**崩溃后无法安全恢复 。**
- `5.5`之前默认的存储引擎
- **优势是访问的速度快,对事务完整性没有要求或者以``SELECT、INSERT``为主的应用**
- 针对数据统计有额外的常数存储。故而``count(*)``的查询效率很高
- 数据文件结构：（在《第02章_MySQL数据目录》章节已讲）
  - 表名``.frm``存储表结构
  - 表名``.MYD``存储数据 (``MYData``)
  - 表名``.MYI``存储索引 (``MYIndex``)
- **应用场景：只读应用或者以读为主的业务**  

#### `Arichive`引擎

> **`Arichive`引擎常用于对数据进行单纯的存储,不进行任何`SELECT,UPDATE,INSERT`等`DML`操作**

**下表展示了``ARCHIVE``存储引擎功能**  

| 特征                                                     | 支持         |
| -------------------------------------------------------- | ------------ |
| B树索引                                                  | 不支持       |
| ``备份/时间点恢复(在服务器中实现，而不是在存储引擎中) `` | 支持         |
| 集群数据库支持                                           | 不支持       |
| 聚集索引                                                 | 不支持       |
| ``压缩数据 ``                                            | 支持         |
| 数据缓存                                                 | 不支持       |
| 加密数据（加密功能在服务器中实现）                       | 支持         |
| 外键支持                                                 | 不支持       |
| 全文检索索引                                             | 不支持       |
| 地理空间数据类型支持                                     | 支持         |
| 地理空间索引支持                                         | 不支持       |
| 哈希索引                                                 | 不支持       |
| 索引缓存                                                 | 不支持       |
| ``锁粒度 ``                                              | 行锁         |
| MVCC                                                     | 不支持       |
| 存储限制                                                 | 没有任何限制 |
| 交易                                                     | 不支持       |
| `更新数据字典的统计信息`                                 | 支持         |

#### `Blackhole`引擎

> **丢弃写操作，读操作会返回空内容**  

#### `CSV`引擎

> - 一个基于`CSV`文件格式的引擎,其会将我们的数据表中的数据按照类似`Excel`中的形式以`逗号`分隔符进行分隔
> - 使用`CSV`引擎存储的数据表会具备两个文件(在`/var/lib/mysql`文件夹下)
>   - `.CSM`文件:用于存储表的状态,表的行数等表的基本结构性信息
>   - `.CSV`文件:用于存储数据表中的数据,没错他就是我们理解的那个`.CSV`文件,我们无论是使用`Excel`还是使用`Pandas-Python`都可以对该文件进行正确的读取与解析,并获取正确的数据呈现格式

#### `Memory`引擎

##### 概述：

> `Memory`采用的逻辑介质是 内存 ， 响应速度很快 ，但是当``mysqld``守护进程崩溃的时候 数据会丢失 。另
> 外，**要求存储的数据是数据长度不变的格式**，比如，``Blob``和``Text``类型的数据不可用(长度不固定的)。

##### 主要特征：

- `Memory`同时支持**哈希（HASH）索引 和 B+树索引 。**
- `Memory`表至少比``MyISAM``表要 快一个数量级 。
- ``MEMORY``表的**大小是受到限制**的。表的大小主要取决于两个参数，分别是``max_rows``和``max_heap_table_size ``.其中,``max_rows``可以在创建表时指定;``max_heap_table_size``的大小默认为``16MB``，可以按需要进行扩大。
- **数据文件**与**索引文件分开存储**。
- **缺点**：其数据易丢失，生命周期短。基于这个缺陷，选择``MEMORY``存储引擎时需要特别小心。

##### 使用``Memory``存储引擎的场景：

-  目标数据比较小 ，而且非常频繁的进行访问 ,在内存中存放数据,如果太大的数据会造成内存溢出.可以通过参数``max_heap_table_size``控制``Memory``表的大小,限制``Memory``表的最大的大小。
- 如果数据是临时的 ,而且必须立即可用得到,那么就可以放在内存中。
- 存储在``Memory``表中的数据如果突然间丢失的话也没有太大的关系 。  

#### `Federated`引擎

> `Federated`引擎是访问其他``MySQL``服务器的一个 代理 ，尽管该引擎看起来提供了一种很好的 跨服务
> 器的灵活性 ,但也经常带来问题，因此**默认是禁用的** 。  

#### `Merge`引擎

> 管理多个``MyISAM``表构成的表集合  

#### `NDB`引擎

> 也叫做``NDB Cluster``存储引擎，主要用于``MySQL Cluster``分布式集群环境，类似于``Oracle``的``RAC``集
> 群。  

#### 各个引擎的对比

| 特 点                | MyISAM                                                      | InnoDB                                                       | MEMORY | MERGE | NDB   |
| -------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ | ------ | ----- | ----- |
| 存 储 限 制          | 有                                                          | `64TB`                                                       | 有     | 没有  | 有    |
| `事 务安 全`         |                                                             | 支持                                                         |        |       |       |
| `锁 机 制`           | 表锁，即使操作一条 记录也会锁住整个 表，不适合高并发的 操作 | 行锁，操作时只锁某一行，不 对其它行有影响，适合高并发 的操作 | 表锁   | 表锁  | 行 锁 |
| B树 索 引            | 支持                                                        | 支持                                                         | 支持   | 支持  | 支 持 |
| 哈 希 索 引          |                                                             |                                                              |        | 支持  | 支 持 |
| 全 文 索 引          | 支持                                                        |                                                              |        |       |       |
| 集 群 索 引          |                                                             | 支持                                                         |        |       |       |
| 数 据 缓 存          |                                                             | 支持                                                         | 支持   |       | 支 持 |
| `索 引缓 存`         | 只缓存索引，不缓存 真实数据                                 | 不仅缓存索引还要缓存真实数 据，对内存要求较高，而且内 存大小对性能有决定性的影响 | 支持   | 支持  | 支 持 |
| 数 据 可 压 缩       | 支持                                                        |                                                              |        |       |       |
| 空 间 使 用          | 低                                                          | 高                                                           | N/A    | 低    | 低    |
| 内 存 使 用          | 低                                                          | 高                                                           | 中等   | 低    | 高    |
| 批 量 插 入 的 速 度 | 高                                                          | 低                                                           | 高     | 高    | 高    |
| `支 持外 键`         |                                                             | 支持                                                         |        |       |       |

### `Innodb`与`MyISAM`对比

| 对比 项          | MyISAM                                                    | InnoDB                                                       |
| ---------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| 外键             | 不支持                                                    | 支持                                                         |
| 事务             | 不支持                                                    | 支持                                                         |
| 行表锁           | 表锁，即使操作一条记录也会锁住 整个表，不适合高并发的操作 | 行锁，操作时只锁某一行，不对其它行有影响， 适合高并发的操作  |
| 缓存             | 只缓存索引，不缓存真实数据                                | 不仅缓存索引还要缓存真实数据，对内存要求较 高，而且内存大小对性能有决定性的影响 |
| 自带系 统表使 用 | Y                                                         | N                                                            |
| 关注点           | 性能：节省资源、消耗少、简单业 务                         | 事务：并发写、事务、更大资源                                 |
| 默认安 装        | Y                                                         | Y                                                            |
| 默认使 用        | N                                                         | Y                                                            |

### `Innodb`架构

- **缓冲池** 
  - 缓冲池是主内存中的一部分空间，用来缓存已使用的表和索引数据。缓冲池使得经常被使用的
    数据能够直接在内存中获得，从而提高速度。
- **更改缓存**
  - 更改缓存是一个特殊的数据结构，当受影响的索引页不在缓存中时，更改缓存会缓存辅助索
    引页的更改。索引页被其他读取操作时会加载到缓存池，缓存的更改内容就会被合并。不同于集群索
    引，辅助索引并非独一无二的。当系统大部分闲置时，清除操作会定期运行，将更新的索引页刷入磁
    盘。更新缓存合并期间，可能会大大降低查询的性能。在内存中，更新缓存占用一部分InnoDB缓冲池。
    在磁盘中，更新缓存是系统表空间的一部分。更新缓存的数据类型由innodb_change_buffering配置项管
    理。
- **自适应哈希索引**
  - 自适应哈希索引将负载和足够的内存结合起来，使得InnoDB像内存数据库一样运行，
    不需要降低事务上的性能或可靠性。这个特性通过innodb_adaptive_hash_index选项配置，或者通过--
    skip-innodb_adaptive_hash_index命令行在服务启动时关闭。
- **重做日志缓存**
  - 重做日志缓存存放要放入重做日志的数据。重做日志缓存大小通过
    innodb_log_buffer_size配置项配置。重做日志缓存会定期地将日志文件刷入磁盘。大型的重做日志缓存
    使得大型事务能够正常运行而不需要写入磁盘。
- **系统表空间**
  - 系统表空间包括InnoDB数据字典、双写缓存、更新缓存和撤销日志，同时也包括表和索引
    数据。多表共享，系统表空间被视为共享表空间。
- **双写缓存**
  - 双写缓存位于系统表空间中，用于写入从缓存池刷新的数据页。只有在刷新并写入双写缓存
    后，InnoDB才会将数据页写入合适的位置。
- **撤销日志**
  - 撤销日志是一系列与事务相关的撤销记录的集合，包含如何撤销事务最近的更改。如果其他
    事务要查询原始数据，可以从撤销日志记录中追溯未更改的数据。撤销日志存在于撤销日志片段中，这
    些片段包含于回滚片段中。
- **每个表一个文件的表空间**
  - 每个表一个文件的表空间是指每个单独的表空间创建在自身的数据文件中，
    而不是系统表空间中。这个功能通过innodb_file_per_table配置项开启。每个表空间由一个单独的.ibd数
    据文件代表，该文件默认被创建在数据库目录中。
- **通用表空间**
  - 使用CREATE TABLESPACE语法创建共享的InnoDB表空间。通用表空间可以创建在MySQL数
    据目录之外能够管理多个表并支持所有行格式的表。
- **撤销表空间**
  - 撤销表空间由一个或多个包含撤销日志的文件组成。撤销表空间的数量由
    innodb_undo_tablespaces配置项配置。
- **临时表空间**
  - 用户创建的临时表空间和基于磁盘的内部临时表都创建于临时表空间。
    innodb_temp_data_file_path配置项定义了相关的路径、名称、大小和属性。如果该值为空，默认会在
    innodb_data_home_dir变量指定的目录下创建一个自动扩展的数据文件。
- **重做日志**
  - 重做日志是基于磁盘的数据结构，在崩溃恢复期间使用，用来纠正数据。正常操作期间，
    重做日志会将请求数据进行编码，这些请求会改变InnoDB表数据。遇到意外崩溃后，未完成的更改会自
    动在初始化期间重新进行。

## `MySQL`索引的数据结构

### `MySQL`数据表的存储结构

- 我们在计算机操作系统的课程中知道，计算机中无论是``内存``还是``磁盘``设备,其存储空间的划分大概率是基于段页式的,每一个段中存储着大量的页的物理地址相关的信息,而每一个页中又存储着大量的存储空间块的物理地址以及相关信息.这些物理块就是我们最终用于存储数据的`内存`或`磁盘`上的物理空间.而我们的段与页都只是一个数据结构,其由我们的操作系统维护,其主要作用就是高效地将物理内存空间块按照逻辑结构组织起来,提升数据访问速度(段与页所需的存储空间则不适用于这种分块存储的方式,因此一般段与页都是整体地存在`内存`与`磁盘上`,并由我们的操作系统自行维护其物理地址以及相关信息.
- 由上面我们知道我们的`MySQL`数据表只要是需要存储就必然不可能离开段页式的存储管理机制,而我们的后续讨论为了简便起见忽略掉段结构,直接假设文件系统以页式管理结构管理我们的存储空间
- 由于一个页所占据的存储空间块的总空间有限,因此往往我们一张数据表上的数据无法存储在一个页上,而是需要在多个页上分别存储.
- 为了简单起见,我们假设数据表的存储的最小单元为一个记录,即若当前页所剩的存储空间无法完整存储下我们需要保存的记录,那么我们就申请新的页来进行记录的保存.
- 最终我们的`MySQL`数据表存储结构就变成了,一个数据表被拆分为大量的页,每一个页中保存着一定数目的记录
  - ![image-20220712211522415](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-23?token=AOAPFCKKMNR4NJP7QAH3D2TC57X5I)

### 为什么引入索引

> 在操作系统中数据页常规情况下是通过双向链表进行维护的,即每一个页都会维护两个指针,一个指向下一个页,一个指向上一个页.那么此时就会导致一个问题,如果我们想要访问指定记录下的数据,那么么我们就必须遍历整个页结构为节点的双向链表,这就导致了我们`MySQL`服务会十分低效,因此必须引入索引,将这些页结构以一种更加合理的方式存储起来

### 索引初探(``主键索引``)

> **前提**:我们把前面提到的数据页称我**用户数据页**
>
> 我们能想到的最简单的一种索引办法就是让各个用户数据页中的记录**严格按照主键进行排序**,使得越接近双向链表头部的页中存储的记录具备越小的主键,越解决双向链表尾部的记录具备越大的主键.然后我们额外再维护一个页,这个页我们称为**目录页**,其以单向链表的方式(以一个页的基本信息为一个节点),按照对应页存储的最大主键的大小,按序存储着我们所有的用户数据页的物理地址以及对应用户数据页的主键的最大值.当我们需要查询数据时,就只需要首先查询该目录页,找到我们需要的记录所在的用户数据页的物理地址,然后访问该物理地址页,然后在该用户数据页中找到对应的记录即可完成我们对记录的操作.

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-24?token=AOAPFCP4PP3Q26IFROF4XLTC57X54" alt="image-20220712212923599" style="zoom:80%;" />

### 主键索引进阶

> 在使用中我们发现,我们的数据记录实在过多以至于用户数据页数量过多,单个目录页无法存储我们所有的用户数据页的信息,因此我们就演化形成如下的形式

- **使用多个目录项**页
  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-25?token=AOAPFCJPDA3ZOFSAMHGA2HTC57X6Q" alt="image-20220712213207457" style="zoom:80%;" />
- **进一步增加目录页的目录页**
  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-26?token=AOAPFCO7GKQJIQPFMHNHAZDC57X7E" alt="image-20220712213311861" style="zoom:80%;" />
- **然后增加目录页的目录页的目录页,一直增加逐渐演化称如下结构**(==**也就是我们经常听说的`B+`树**==)
  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-27?token=AOAPFCP2POW2B4TQITSCVYTC57X7Y" alt="image-20220712213457991" style="zoom:50%;" />

### `B+`树应用于主键索引

> 假如我们令`B+`树有四层(包括用户记录层),即我们有**三层目录层**,**一层用户数据层**,不妨设一个目录页节点可以存储``1000个子节点``,一个用户数据页可以存储`100`个数据记录
>
> - 第一层目录层共一个目录页节点
> - 第二层目录由第一层目录的`1000`个子节点组成
> - 第三层目录由`1000`个第二层节点的共`1000*1000=1000000`个节点组成
> - 由此我们可以知道,这样一个`4层`的`B+`树可以索引`1000000`个用户数据表,总共`1000*1000*100=100000000=1亿`条记录

### `HASH`索引

#### 什么是`HASH`索引

> `HASH`索引即我们通过`HASH`表的数据结构将我们的数据记录组织起来,当我们要访问某一条记录时就可以先通过特殊的值来获取到`HASH`值,从而在`HASH`表中获取到对应的数据记录.一般来说`HASH`索引只是作为一个聚簇与非聚簇索引方式的**辅助索引**方式出现.**如果我们对一张数据表的大量查询中某一个该表的字段大量出现在`WHERE`子句,`HAVING`子句中,那么我么的存储引擎就会自动地建立起关于该字段的一个`HASH`索引,其以我们的该字段的值来生成`HASH`值进而生成关于各个数据记录的`HASH`表,并以我们的记录的物理位置作为`HASH`表的值.当我们需要查找该字段为指定值的记录时,只需要用该值生成`HASH`值,然后在`HASH`索引表中找到对应的值即可.**

#### 为什么`HASH`索引十分高效却没有被广泛使用呢?

- 哈希索引**的各个结点只保存该节点的哈希码,指向下一个节点的指针,指向记录存储地址的指针**，而**不存储字段值**，所以不能使用索引中的值来避免读取数据。不过访问内存中的行速度非常快(因为是MEMORY引擎/),所以对性能影响并不大

- 哈希索引数据并不是按照索引值顺序存储的，所以无法用于排序

- 哈希索引**不支持部分索引列查找**，因为哈希索引始终是使用索引列的全部内容来计算哈希码。 如,以字段`A,B`为基础计算`HASH`值从而建立哈希索引，如果查询这涉及到字段``A``的筛选而不涉及到字段`B`，则无法使用该哈希索引(**无法计算HASH值**)

- 哈希索引**只支持等值查询**，包括`=、IN()、<=>`,**不支持范围查询**,如``where price > 100``

- 哈希冲突(不同索引列会用相同的哈希码)会影响查询速度,此时需遍历索引中的指向记录存储地址的指针,**逐个记录进行比较**。    

![img](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-28?token=AOAPFCLCLNGWSU5GDW35DA3C57YAM)

- 如果哈希冲突很多，一些索引维护操作的代价会很高。

![img](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-29?token=AOAPFCN3F6SOAPHRR4YNMHDC57YBA)

- 如果从表中删除一行，需要遍历链表中的每一行，找到并删除对应行的引用，冲突越多，代价越大。

#### 为什么`MEMORY`引擎支持`HASH`索引,而其他引擎不支持呢?

### ==**索引的分类**==

> **索引按照物理实现方式，索引可以分为 2 种：聚簇（聚集）和非聚簇（非聚集）索引。我们也把非聚集
> 索引称为二级索引或者辅助索引. **
>
> **聚簇索引与非聚簇索引的主要区别**
>
> - 聚簇索引的`B+`树的叶子节点中**保存着我们的实际的用户记录数据**
> - 非聚簇索引的`B+`树的叶子节点中保存着的**只不过是我们实际的用户的记录数据的物理存储位置.**

#### 聚簇索引

> - **记录在页内的链表节点以主键大小排序**
> - **在同级页组成的双向链表中每一个节点为一个页,并且他们也按照主键大小排序**
> - **叶子节点存储实际用户数据,而不是物理地址**

##### 特点：

- **使用记录主键值的大小进行记录和页的排序**，这包括三个方面的含义：
  - **页内的记录**==**是按照主键的大小顺序**==排成一个**单向链表** 
  - 各个**存放用户记录的页**也是根据页中用户记录的主键大小顺序排成一个**双向链表** 
  - **存放目录项记录的页**分为不同的层次，在**同一层次中**的页也是根据页中目录项记录的主键大小顺序排成一个**双向链表**。
- ==**B+树的叶子节点存储的是完整的用户记录。所谓完整的用户记录,就是指这个记录中存储了所有列的值（包括隐藏列）。**==

##### 优点：

- **数据访问更快**,因为聚簇索引将索引和数据保存在同一个B+树中，因此从聚簇索引中获取数据比非聚簇索引更快
- 聚簇索引对于主键的**排序查找和范围查找速度非常快**
- 按照聚簇索引排列顺序，**查询显示一定范围数据的时候**，由于**数据都是紧密相连**，数据库不用从多个数据块中提取数据，所以**节省了大量的``io``操作** 。

##### 缺点：

- **插入速度严重依赖于插入顺序**,按照主键的顺序插入是最快的方式，否则将会出现页分裂,严重影响性能。因此，对于使用``InnoDB``存储引擎的表，我们一般都会定义一个**自增的ID列为主键**
- **更新主键的代价很高** ，因为将会导致被更新的行移动。因此，对于使用``InnoDB``存储引擎的表，我们**一般定义主键为不可更新**

#### 非聚簇索引(二级索引,辅助索引)

> - **记录在页内的链表节点可以以任意一个合法的字段的大小进行排序**
> - **在同级页组成的双向链表中每一个节点为一个页,并且他们也可以以任意一个合法的字段的大小进行排序**
> - **叶子节点存储物理地址或用户的主键,而不是实际用户记录数据**
>
> **注意**:
>
> - 在`Innodb`中的非聚簇索引的叶子节点存储的是用户记录的主键
> - 在`MyISAM`中的非聚簇索引的叶子节点存储的是用户记录的物理存储地址

##### 特点:

##### 优点:

##### 缺点:

#### 补充2:非聚簇索引下的回表机制

> **前提**:我们知道`Innodb`与`MyISAM`的`B+`树的叶子节点存储的数据是不同的
>
> - `MyISAM`为用户数据物理存储地址
> - `Innodb`为记录的主键的值
>
> **回表的概念**:若我们仅仅只对我们的当前索引文件进行一次查找无法获取到我们希望得到的数据,那么此时我们就说第二次进行文件读取的操作为回表
>
> - **`MyISAM`下在索引文件中找到物理地址后,还需要去数据文件中按照物理地址获取数据,而这一数据获取过程被称之为`MyISAM`下的回表**
> - **`Innodb`的非聚簇索引下,在非聚簇索引文件中找到我们的需要的记录的主键后,还需要到我们的`.ibd`文件下按照获取到的主键进行`聚簇索引`查找才能获得数据,而这一``聚簇索引``查找的过程被称之为`Innodb`下的回表**

#### 补充2:联合索引

> **前提**:联合索引可以认为是**非聚簇索引的一个拓展**
>
> - **记录在页内的链表节点可以以任意的多个合法的字段的大小进行排序**
> - **在同级页组成的双向链表中每一个节点为一个页,并且他们也可以以任意的多个合法的字段的大小进行排序**
> - **叶子节点存储物理地址或用户的主键,而不是实际用户记录数据**

##### 联合索引下的排序规则

- 假如我们指定通过字段`C1,C2`作为排序规则

  ```shell
  #记录1:C1=1,C2=5
  #记录2:C1=2,C2=1
  #记录3:C1=1,C2=2
  #记录4:C1=3,C2=6
  #记录5:C1=2,C2=2
  
  #排序优先级为
  记录3>记录1>记录2>记录5>记录4
  ```

- **其含义为：**

  - 先把各个记录和页按照``C1字段``进行排序。

  - 在记录的``C1字段``**相同的情况下**,采用``C2字段``进行排序

注意一点,以``C1字段和C2字段``的大小为排序规则建立的``B+树``称为**联合索引** ,**本质上也是一个二级索引**。它的意思与分别为``C1字段与C2字段``**分别建立索引的表述是不同的**,不同点如下：

- **建立联合索引**只会建立**1棵非聚簇的B+树**。
- 为``C1和C2字段``**分别建立索引**会分别以``C1和C2字段``列的大小为排序规则**建立2棵f非聚簇B+树**。  

### 两种主流的索引方式对比(`B+`树索引与`Hash`索引)

### `MyISAM`下的索引数据结构

> **`MyISAM`采取的是基于`B+`树的非聚簇索引**

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-30?token=AOAPFCLZEU32OTAE4RJXJ7DC57YBY" alt="image-20220712233316893" style="zoom:130%;" />

### `MyISAM`与`Innodb`在索引使用上的对比

- 在``InnoDB``存储引擎中，常规情况下我们的数据都是存储在聚簇索引结构的`.ibd`文件中,因此我们只需要根据**主键值**,对``聚簇索引``进行一次查找就能找到对应的记录，而在``MyISAM``中却由于索引与数据分离的特性,而需要进行一次``回表``操作，意味着``MyISAM``中建立的索引相当于全部都是 二级索引 。

- `InnoDB`的数据文件本身就是索引文件，而``MyISAM``索引文件和数据文件是分离的,**索引文件仅保存数据记录的地址**。
- `InnoDB`的**非聚簇索引**``data``域**存储相应记录主键的值** ，而``MyISAM``索引**记录的是地址** 。换句话说，``InnoDB``的所有非聚簇索引都引用主键作为data域。
- `MyISAM`的回表操作是十分快速的，**因为是拿着地址偏移量直接到文件中取数据**的，反观``InnoDB``是通过**获取主键之后再去聚簇索引**里找记录，虽然说也不慢，但还是比不上直接用地址去访问。
- `InnoDB`要求表**必须有主键**(`MyISAM`可以没有)。如果**没有显式指定**，则`MySQL`系统会==**自动选择一个可以非空且唯一标识数据记录的列作为主键**==。如果**不存在**这种列，则`MySQL`自动为`InnoDB`表==**生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整型**。==

​	![image-20220713000457848](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-31?token=AOAPFCJYDBA77VQ7KRUWQL3C57YC2)

### 使用索引的代价

#### 空间上的代价

- 每给数据表建立一个索引都要为它建立一棵B+树，**每一棵B+树的每一个节点都是一个数据页**，一个页默认会占用``16KB``的存储空间，一棵很大的B+树由许多数据页组成，那就**是很大的一片存储空间**。

#### 时间上的代价

- 每次对表中的数据进行``增、删、改``操作时，都需要去修改各个B+树索引。而且我们讲过，==**B+树每层节点都是按照该节点直接或间接关联到的所有用户数据页中包含的所有记录的索引字段的值中的最小值``按照从小到大的顺序排序``而组成了``双向链表``**==。不论是**叶子节点**中的记录，还是**内节点**中的记录（**也就是不论是用户记录还是目录项记录**）==**都是按照该节点直接或间接关联到的所有用户数据页中包含的所有记录的索引字段的值中的最小值**``从小到大的顺序排序``而形成了一个``单向链表``==。而``增、删、改``操作可能会**对节点和记录的排序造成破坏**，所以存储引擎需要**额外的时间**进行一些``记录移位``,``页面分裂``、``页面回收``等操作来维护好节点和记录的排序。**如果我们建了许多索引，每个索引对应的B+树都要进行相关的维护操作，会给性能拖后腿。**  

### `二叉树`,``AVL树``,``B树``,``B+树``,`R树`

#### 二叉搜索树

> 特点:每一个节点最多可以拥有两个子节点,并且两个子节点的键的值应该满足以下要求
>
> - 左子节点的键的值应小于(大于)父节点的键的值
> - 右子节点的键的值应大于(小于)父节点的键的值

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-32?token=AOAPFCIEH3UZ7STEIYY5WLDC57YDW" alt="image-20220713124603214" style="zoom:50%;" />

#### `AVL`树(平衡二叉树)

> **特点**:
>
> - 平衡二叉树的左子树与右子树也是平衡二叉树
> - 平衡二叉树的左子树与右子树的高度差应该**小于等于1**
> - 平衡二叉树是基于二叉树的,因此其也同时满足二叉树的特性

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-33?token=AOAPFCJTVU6EFZGT7WBNCVDC57YEK" alt="image-20220713125026175" style="zoom:67%;" />

#### `B`树

> **`M`阶`B`树的特性**:
>
> - 无论是根节点还是中间节点还是叶子节点都会存储用户记录,存储的记录条数由`k1,k2,k3`指示
>
> - 根节点与中间节点都是由**用户记录**(以``KEY``指代)与**指向子节点的指针**两个部分组成
>
> - 根节点可以有`[2,M]`个子节点,其内必然包含`k1-1`个用户记录与`k1`个指向子节点的指针
>
> -  每个中间节点内必然包含`k2-1`个用户记录与`k2`个指向子节点的指针
> - 叶子节点只保存有用户记录,而不再保存指向子节点的指针.并且叶子节点应具备`k3-1`个用户记录
> - `k`的取值范围为``[ceil(M/2),M]**``**
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-34?token=AOAPFCOI7GE245IZK3JZ53LC57YFA" alt="image-20220713130220355" style="zoom: 80%;" />
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-35?token=AOAPFCJT5HQJMIB5KCBXJNDC57YFQ" alt="image-20220713130241424" style="zoom:80%;" />

- **`B`树的查询流程**(以上面的第一个图为例,查询主键值为**36**的记录)
  - 首先查询根结点,我们发现`36>17且36>35`,因此我们取得`P3`指针,并根据该指针得到下一个节点
  - 查询`P3`指向的节点,发现`36<65<87`,因此我们取得`P1`指针,并根据该指针得到下一个节点
  - 查询`P1`指向的节点(即磁盘块9),至此我们查找到了我们主键值为`36`的记录

#### `B+`树

> **`B+`树的特性**
>
> - 与`B`树不同,`B+`树并==**不具备**==**父节点的左子树下的所有用户记录的主键值都小于该父节点的主键值,父节点的右子树下的所有用户记录的主键值都大于该父节点的主键值的这一特性**
>
> - `B+`树的根节点与中间节点是不存储用户记录的,用户记录全部在我们的叶子节点中存储
> - 根节点与中间节点主要由**一些主键值**与**一些指向子节点的指针**两个部分组成
> - 叶子节点主要由**一些带有主键值的用户记录**组成
> - 无论是根节点还是中间节点,有`k1`个**主键值**就一定会**一一对应**地具有`k1`个指向子节点的指针
> - 根节点或中间节点的各个主键值是来自于用户记录的,其主要生成过程如下
>   - 根节点与中间节点的每一个主键值都会一一对应一个指向子节点的指针.而这个指针就会指向一个子树,由此我们可以发现,最终我们的根节点与中间节点都会直接或者间接地关联到一些磁盘块的集合,也就等价于关联到一些用户记录的集合
>   - 因此我们的根节点与中间节点的主键值的取值就会根据该节点关联到的大量用户记录的主键值来选取,有以下两种选取策略
>     - 选取关联到的所有用户记录中**主键值最小**的用户记录的主键值为我们该节点的主键值
>     - 选取关联到的所有用户记录中**主键值最大**的用户记录的主键值为我们该节点的主键值
>
> ==**重要注意事项**==:
>
> - 由于`B+`树的特性`1`,导致了我们的`B+`树的同一级别的页之间必须通过双向链表结构按照主键值大小为顺序有序地整合起来.
>   - 如果不整合起来,我们要查主键值为`4`的节点,我们假设根据搜索规则找到了`页34`,但是很明显此时`页35`中也具备主键值为4的记录,因此我们需要将页按顺序连接起来,这样我们访问到`页34`的同时也可以通过双向链表访问到`页33,页35等`从而我们就可以遍历上下文,从而解决我们`B+`树特性``1``导致的问题

- **`B+`树在最小主键值选取规则情况下的查询流程**<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-36?token=AOAPFCMSHSSAT3HZO2QBDJLC57YGE" alt="image-20220713132234953" style="zoom:75%;" />

- **`B+`树在最大主键值选取规则情况下的查询流程**

  <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-37?token=AOAPFCM43FOOXMZJXAV4ODDC57YGW" alt="image-20220713132936748" style="zoom:75%;" />

#### `R`树

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-41%E3%80%81?token=AOAPFCOYWJRGQX7HOBZDPKTC57YLA" alt="image-20220713001907535" style="zoom:80%;" />

### 为什么使用`B+`树不使用二叉树?

### `B+`树的特性

### 各种树的算法复杂度

![image-20220712220223352](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-40?token=AOAPFCIILUZSSGEPAMQO7VTC57YKG)

## `Innodb`存储引擎

### `页,区,段,表空间`

#### 页

> - 页是`Innodb`存储引擎下最小的数据交互单位
> - 页的默认大小为`16KB`
> - 一个页中可以包含多个记录

#### 区

> - `Innodb`存储引擎下一个区由`64`个连续的页组成
> - 区的默认大小为`64×16KB=1MB`

#### 段

> - `Innodb`中一个段由多个区组成,这些区可以是**连续存储**的也可以是**非连续存储**的
> - 段是`Innodb`存储引擎下**数据对象的基本分配单位**,一个数据表就会有其对应的段,一个数据表的索引也会有其对应的段.**每一个数据对象只能对应于一个段**

**表空间**

> - 表空间中存储着一个或多个段
> - 同一个段只能保存在一个表空间中
> - 一个数据库由一个或多个表空间组成
> - 表空间有`系统表空间,用户表空间,撤销表空间,临时表空间组成`

### `页`的基本结构

#### 页的头部的基本页信息存储空间

> - 页的头部占`38Bytes`
> - 页的头部存储着该页的一些基本信息

| 类型名称                           | 占用空间大小 | 描述                                                         |
| ---------------------------------- | ------------ | ------------------------------------------------------------ |
| `FIL_PAGE_SPACE_OR_CHKSUM`         | `4`字节      | 页的校验和（checksum值）                                     |
| `FIL_PAGE_OFFSET`                  | `4`字节      | 页号                                                         |
| `FIL_PAGE_PREV`                    | `4`字节      | 上一个页的页号                                               |
| `FIL_PAGE_NEXT`                    | `4`字节      | 下一个页的页号                                               |
| `FIL_PAGE_LSN`                     | `8`字节      | 页面被最后修改时对应的日志序列位置                           |
| `FIL_PAGE_TYPE`                    | `2`字节      | 该页的类型                                                   |
| `FIL_PAGE_FILE_FLUSH_LSN`          | `8`字节      | 仅在系统表空间的一个页中定义，代表文件至少被刷新到了对应的LSN值 |
| `FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID` | `4`字节      | 页属于哪个表空间                                             |

##### `FIL_PAGE_TYPE`页类型的取值

| 类型名称                | 十六进制 | 描述                             |
| ----------------------- | -------- | -------------------------------- |
| FIL_PAGE_TYPE_ALLOCATED | `0x0000` | 最新分配，还没有使用             |
| `FIL_PAGE_UNDO_LOG`     | `0x0002` | Undo日志页                       |
| FIL_PAGE_INODE          | `0x0003` | 段信息节点                       |
| FIL_PAGE_IBUF_FREE_LIST | `0x0004` | Insert Buffer空闲列表            |
| FIL_PAGE_IBUF_BITMAP    | `0x0005` | Insert Buffer位图                |
| `FIL_PAGE_TYPE_SYS`     | `0x0006` | 系统页                           |
| FIL_PAGE_TYPE_TRX_SYS   | `0x0007` | 事务系统数据                     |
| FIL_PAGE_TYPE_FSP_HDR   | `0x0008` | 表空间头部信息                   |
| FIL_PAGE_TYPE_XDES      | `0x0009` | 扩展描述页                       |
| FIL_PAGE_TYPE_BLOB      | `0x000A` | 溢出页                           |
| `FIL_PAGE_INDEX`        | `0x45BF` | 索引页，也就是我们所说的`数据页` |

#### `FIL_PAGE_SPACE_OR_CHKSUM`与`FIL_PAGE_LSN`页校验

#### 页的记录存储空间

> - 对于页中的数据存储,`Innodb`将一个数据页划分为如下三个部分
>   - `Free Space`空闲空间
>   - `User Records`用户记录空间
>   - `Infimum&Supremum`最大最小记录空间

##### `Free Space`空闲空间

- 当我们有用户记录要存入我们的数据页中时,系统会向`Free Space`空闲空间申请空间,将其转为`User Records`空间,然后我们的数据记录会被存入我们的`User Records`空间

##### `User Records`用户记录空间

- **用于存储用户记录**,其初始状态下存储空间大小为`0`,每当有用户记录需要存入都会向`Free Records`空间进行空间分配申请,申请到空间后再接收我们的用户记录的存入

##### `Infimum&Supremum`最大最小记录空间

> 最大最小记录是`MySQL`自动生成的记录,最小记录在`User Records`左端,最大记录在`User Records`空间右端

- 空间由最大记录与最小记录两个部分组成.这两个部分都由`5Bytes`的基本信息头与`8Bytes`的固定部分组成

#### `Page Directory`页目录空间

> - 所有的用户记录通过`单向链表`连接并且按照`主键`取值排序.而单向链表的问题在于其检索效率低下,若要检索就必须对整个单向链表进行遍历
> - **页目录存储在一段连续的存储空间上**
>
> - **记录分组**
>   - `MySQL`会对我们的记录(包括最大最小记录)进行分组,我们的页目录空间会保存每一个分组的最后一个记录的**相对于数据页的第一个`Bit`的物理偏移量**(由于记录的主键排序规则,最后一个记录的主键值一定是其所在分组的最大值.)
>   - 页目录空间的每一个物理偏移量都被称为一个**槽(``slot``)**
>   - **在不插入任何记录的情况下页目录空间中会存储两个组(一个组中包含最小记录,一个组中包含最大记录**)

##### 分组规则

- 第一组中只能拥有最小记录这一条记录
- 最后一组中只能拥有`1~8`条记录(包括最大记录)
- 中间的组中必须保证记录数在`4~8`条

##### 记录的插入规则

- 初始情况下,一个数据页里**只有最小记录和最大记录两条记录**，它们**分属于两个分组**。
- 之后**每插入一条记录**，都会**从页目录中**将其插入到==**主键值比本记录的主键值大**==,==**差值最小的槽所在的分组中**==，然后把**该槽对应的记录的``n_owned``值加1**，表示本组内又添加了一条记录，直到该组中的记录数等于8个。
  - 插入方式很简单,我们通过槽可以访问到该分组的最后一个记录,通过该记录我们又可以访问它前面的记录,从而找到合适的位置就可以让前面的记录的指针指向它,它的指针指向前一个记录原本指向的记录即可实现插入
- **在一个组中的记录数等于8个后再插入一条记录时，会将组中的记录拆分成两个组，一个组中4条记录，另一个5条记录。这个过程会在页目录中新增一个槽来记录这个新增分组中最大的那条记录的偏移量。**
  - ==**注意**==:**拆分方式是将该分组的前四个记录作为一个分组,后四个记录以及新插入的一条记录作为一个分组**

#### **`Page Header`页面头部空间**

> - 该空间在`页`空间中占据固定位置的固定的`56Bytes`的空间
>   - **固定位置是指无论是在哪个页中该页的`Page Header`空间相对于页的起始位置的偏移量都相同**

| 名称              | 占用空间大小 | 描述                                                         |
| ----------------- | ------------ | ------------------------------------------------------------ |
| PAGE_N_DIR_SLOTS  | 2字节        | 在页目录中的槽数量                                           |
| PAGE_HEAP_TOP     | 2字节        | 还未使用的空间最小地址，也就是说从该地址之后就是`Free Space` |
| PAGE_N_HEAP       | 2字节        | 本页中的记录的数量（包括最小和最大记录以及标记为删除的记录） |
| PAGE_FREE         | 2字节        | 第一个已经标记为删除的记录的记录地址（各个已删除的记录通过`next_record`也会组成一个单链表，这个单链表中的记录可以被重新利用） |
| PAGE_GARBAGE      | 2字节        | 已删除记录占用的字节数                                       |
| PAGE_LAST_INSERT  | 2字节        | 最后插入记录的位置                                           |
| PAGE_DIRECTION    | 2字节        | 记录插入的方向                                               |
| PAGE_N_DIRECTION  | 2字节        | 一个方向连续插入的记录数量                                   |
| PAGE_N_RECS       | 2字节        | 该页中记录的数量（不包括最小和最大记录以及被标记为删除的记录） |
| PAGE_MAX_TRX_ID   | 8字节        | 修改当前页的最大事务ID，该值仅在二级索引中定义               |
| PAGE_LEVEL        | 2字节        | 当前页在B+树中所处的层级                                     |
| PAGE_INDEX_ID     | 8字节        | 索引ID，表示当前页属于哪个索引                               |
| PAGE_BTR_SEG_LEAF | 10字节       | B+树叶子段的头部信息，仅在B+树的Root页定义                   |
| PAGE_BTR_SEG_TOP  | 10字节       | B+树非叶子段的头部信息，仅在B+树的Root页定义                 |

### `Innodb`数据记录的基本存储格式

- **指定数据表的数据记录基本存储格式**

  ```mysql
  CREATE TABLE <表名>(字段列表) ROW_FORMAT = <格式名>;
  ALTER TABLE <表名> ROW_FORMAT = <格式名>
  ```

#### 常用行格式

##### `COMPACT`行格式

- **变长字段长度列表**

- **`NULL`值列表**

- **记录头信息(固定的`5Bytes`)**

  | 名称            | 大小（单位：bit） | 描述                                                         |
  | --------------- | ----------------- | ------------------------------------------------------------ |
  | `预留位1`       | 1                 | 没有使用                                                     |
  | `预留位2`       | 1                 | 没有使用                                                     |
  | `delete_mask`   | 1                 | 标记该记录是否被删除                                         |
  | `mini_rec_mask` | 1                 | B+树的每层非叶子节点中的最小记录都会添加该标记               |
  | `n_owned`       | 4                 | 表示当前记录所在的分组拥有的记录数                           |
  | `heap_no`       | 13                | 表示当前记录在记录堆的位置信息                               |
  | `record_type`   | 3                 | 表示当前记录的类型，`0`表示普通记录，`1`表示B+树非叶子节点记录，`2`表示最小记录，`3`表示最大记录 |
  | `next_record`   | 16                | 表示下一条记录的相对位置                                     |

- **记录的实际数据**

  | 列名             | 是否必须 | 占用空间 | 描述                   |
  | ---------------- | -------- | -------- | ---------------------- |
  | `row_id`         | 否       | 6字节    | 行ID，唯一标识一条记录 |
  | `transaction_id` | 是       | 6字节    | 事务ID                 |
  | `roll_pointer`   | 是       | 7字节    | 回滚指针               |

  - `row_id`:若一个表没有手动定义主键，则会自动选取一个``Unique``键的字段作为主键，如果连``Unique``键都没有定义的话，则**会为表默认添加一个名为``row_id``的隐藏列作为主键**。所以``row_id``是在**没有自定义主键以及``Unique``键的情况下才会存在的**。

#### `Dynamic`,`Compressed`,`Compact`,`Redundant`四种格式对比

#### 数据溢出问题的解决方案

> **概念**:有一些字段,如`VARCHAR,TEXT,BLOB`等字段的最大存储空间是明显大于单个数据页所占据的存储空间的.因此此时我们肯定无法通过单个页来存储这些数据,`MySQL`中则是通过引入溢出页的概念来实现这些字段的存储的
>
> **一般解决方案**:对于占用存储空间非常大的列，**在记录的真实数据处只会存储该列的一部分数据，把剩余的数据分散存储在几个其他的页中进行`分页存储`，然后记录的真实数据处用20个字节存储指向这些页的地址（当然这20个字节中还包括这些分散在其他页面中的数据的占用的字节数），从而可以找到剩余数据所在的页。这称为`页的扩展`。**

- **``Compressed和Dynamic``**:对于存放在``BLOB``中的数据采用了**完全的行溢出的方式**。如图，在数据页中只存放20个字节的指针（溢出页的地址），实际的数据都存放在Off Page（溢出页）中
- **`Compact和Redundant`**:在记录的真实数据处存储一部分数据(存放``768``个前缀字节),剩下的数据存储在`溢出页`中

### `区,段,碎片区`的存在意义

> **随机`I/O`**:对我们磁盘上物理位置分布十分没有规律的数据进行读取的过程就称之为随机`I/O`
>
> - 如对于机械硬盘而言,数据的物理位置分布随机就意味着我们的磁头要在磁盘上大跨度的地跳转来读取数据
>
> 顺序`I/O`:对我们磁盘上物理位置分布十分接近的数据进行读取的过程就称之为顺序`I/O`

#### `区`的存在意义

> - 对于我们`Innodb`的`B+`树而言,虽然同级的页都会通过双向链表组织起来.但是当我们需要读取当前页的前一个或后一个页时,也需要根据链表上的指针来获取物理位置然后读取,如果`页与页`之间物理位置相距过远,就会导致我们对数据进行随机`I/O`,其效率是地下的.
> - **我们的`MySQL`通过引入区的概念,一次向操作系统申请一个区的连续空间,然后将该区划分为`64`个页再分配给我们的`B+`树就能够很好地避免随机`I/O`的问题**

#### `段`的存在意义

> - 段是一个逻辑数据结构,并不涉及到存储空间的设计,只是一个抽象概念
> - 对于范围查询，其实是对B+树叶子节点中的记录进行顺序扫描，而如果不区分叶子节点和非叶子节点，统统把节点代表的页面放到申请到的区中的话，进行范围扫描的效果就大打折扣了。所以InnoDB对B+树的`叶子节点`和`非叶子节点`进行了区别对待，也就是说叶子节点有自己独有的区，非叶子节点也有自己独有的区。存放叶子节点的区的集合就算是一个`段（segment）`，存放非叶子节点的区的集合也算是一个段。也就是说一个索引会生成2个段，一个`叶子节点段`，一个`非叶子节点段`。
> - **除了索引的叶子节点段和非叶子节点段之外，``InnoDB`中还有为存储一些特殊的数据而定义的段，比如回滚段。所以，常见的段有`数据段`、`索引段`、`回滚段`。数据段即为B+树的叶子节点，索引段即为B+树的非叶子节点。**

#### `碎片区`的存在意义

> **问题**:默认情况下，一个使用InnoDB存储引擎的表只有一个聚簇索引，一个索引会生成2个段，而段是以区为单位申请存储空间的，一个区默认占用1M（64*16KB=1024KB）存储空间，所以**默认情况下一个只存在几条记录的小表也需要2M的存储空间么？**以后每次添加一个索引都要多申请2M的存储空间么？这对于存储记录比较少的表是天大的浪费
>
> - `碎片区`是`Innodb`为了考虑以完整的区为单位分配给某个段对于`数据量较小`的表太浪费存储空间的这种情况而引入的
> - **在一个碎片区中，并不是所有的页都是为了存储同一个段的数据而存在的，而是碎片区中的页可以用于不同的目的，比如有些页面用于段A，有些页面用于段B，有些页甚至哪个段都不属于。`碎片区直属于表空间`，并不属于任何一个段。**

### `Innodb`空间分配策略

- 在刚开始向表中插入数据的时候，段是从某个碎片区以单个页面为单位来分配存储空间的。
- 当某个段已经占用了`32个碎片区`页面之后，就会申请以完整的区为单位来分配存储空间。

#### 区的分类

- `空闲的区(FREE)`：现在还没有用到这个区中的任何页面。
- `有剩余空间的碎片区(FREE_FRAG)`：表示碎片区中还有可用的页面。
- `没有剩余空间的碎片区(FULL_FRAG)`：表示碎片区中的所有页面都被使用，没有空闲页面。
- `附属于某个段的区(FSEG)`：每一索引都可以分为叶子节点段和非叶子节点段

处于`FREE`、`FREE_FRAG`以及`FULL_FRAG`这三种状态的区都是独立的，直属于表空间。而处于`FSEG`状态的区是附属于某个段的。**当然`FULL_FRAG`与`FREE_FRAG`并不是不与段构成关系,实际情况是这两个区可能会被多个不同的段所占用,这与`FREE`是完全不同的,`FREE`区是完全不与任何段有任何联系的**

## `MySQL`索引的创建

### 为什么要有索引?

### 索引的分类

- **普通索引**
- **唯一性索引**
  - 一个表可以有多个唯一性索引
  - 只要我们指定了某个字段为`UNIQUE`的,那么`MySQL`就会为其自动创建唯一性索引
- **主键索引**
  - 一种``特殊的唯一性索引``,在唯一性索引的基础上添加了字段非空的要求
  - 一张表最多具有一个主键索引
- **全文索引**
  - 全文索引会在创建后实时地对我们的数据表的指定的一个或多个字段进行算法分析,并构建特殊的搜索结构.当我们对全文索引的字段进行搜索时,就会调用全文索引,从而获得我们需要的字段的**主键(或物理地址)**
  - **若设置多列全文索引,那么其内在操作为直接把涉及到的多列看作一个内容进行索引创建操作**
  - **只能用于`CHAR,VARCHAR,TEXT`等文本类型的字段**
  - 一般**用于数据量较大的表,数据量较小的表一般没有使用的意义**
  - 全文索引分为`自然语言全文索引`与`布尔全文索引`两种
    - ![image-20220717173346444](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-42?token=AOAPFCJMB2XSDFANTP7QFQ3C57YNI)
- **单列索引**
  - 一个索引只涉及单个字段
  - 一个表可以有多个单列索引
- **多列索引**
  - 一个索引涉及两个及以上的字段
  - 一个表可以有多个多列索引
  - 多列索引遵循最左前缀规则

### 索引的管理

- **索引的查看**

  ```mysql
  #方式一
  SHOW CREATE TABLE 表名
  
  #方式二
  SHOW INDEX FROM 表名
  ```

### 索引的创建,修改,删除

> **前提**:`MySQL`专门为索引的创建,修改,删除创建了一套关键字搭配.
>
> - **主键索引**:`PRIMARY KEY`
> - **常用索引关键字**:`INDEX`
>
> **索引操作的三大类别**
>
> - 创建表时基于以下方式的索引建立
>
>   ```mysql
>   CREATE TABLE 表名(
>   字段列表
>   [UNIQUE|FULLTEXT|SPATIAL|PRIMARY] [INDEX|KEY] [索引名](字段1[(长度)][,字段2[(长度)]...]) [ASC|DESC]
>   );
>   ```
>
> - **创建表时系统会自动创建的索引**
>
>   - `PRIMARY KEY`字段的主键索引
>   - `UNIQUE`的字段的唯一性索引
>   - `FOREIGN KEY REFERENCE`外键约束索引
>
> - **基于`ALTER ADD|ALTER MODIFY|ALTER DROP`的表创建完成后的索引创建修改删除操作**
> - **基于`MySQL`特意设置的`CREATE INDEX|DROP INDEX|DROP PRIMARY KEY`等语句的索引创建修改删除操作**
>
> **注意**:若我们没有对索引进行显式的命名,那么`MySQL`则会自动为其命名
>
> - 只涉及一个字段时,直接命名为该字段
> - 涉及多个字段时,命名为涉及到的字段中定义索引时处于最左端的字段的名称

#### 创建表时显示设置索引

##### 基本语法

```mysql
CREATE TABLE 表名(
字段列表
[UNIQUE|FULLTEXT|SPATIAL|PRIMARY] [INDEX|KEY] [索引名](字段1[(长度)][,字段2[(长度)]...]) [ASC|DESC]
);
```

- `UNIQUE|FULLTEXT|SPATIAL|PRIMARY`都是可选项,用来告诉系统我们创建的是`唯一性索引|全文索引|空间索引|主键索引`,当我们不加这些关键字时,则表示创建普通索引
- `INDEX|KEY`用于告诉我们的系统,我们正在创建索引
- `索引名`用于指定我们创建的索引的名称
- `(字段1[(长度)][,字段2[(长度)]...])`用于指定我们用于创建索引的字段,`长度`选项只有在`TEXT,CHAR,VARCHAR`等文本型字段上才能使用,其作用为告诉系统该字段的前`长度`个字符参与索引的建立过程
- `ASC|DESC`用于指定是按照我们指定的字段按照升序创建索引还是降序创建索引

##### 实例

- **普通索引**

```mysql
#用INDEX创建单列普通索引并指定索引名称
CREATE TABLE 表名(
字段列表
INDEX 索引名(字段1) ASC
);

#用INDEX创建单列普通索引但不指定索引名称
CREATE TABLE 表名(
字段列表
INDEX (字段1) ASC
);

#用KEY创建单列普通索引并指定索引名称
CREATE TABLE 表名(
字段列表
KEY 索引名(字段1) ASC
);

#用KEY创建单列普通索引但不指定索引名称
CREATE TABLE 表名(
字段列表
KEY (字段1) ASC
);

#用INDEX创建多列普通索引并指定索引名称
CREATE TABLE 表名(
字段列表
INDEX 索引名(字段1[(长度)][,字段2[(长度)]...]) ASC
);

#用KEY创建多列普通索引并指定索引名称
CREATE TABLE 表名(
字段列表
KEY 索引名(字段1[(长度)][,字段2[(长度)]...]) ASC
);

#用INDEX创建多列普通索引但不指定索引名称
CREATE TABLE 表名(
字段列表
INDEX (字段1[(长度)][,字段2[(长度)]...]) ASC
);

#用KEY创建多列普通索引但不指定索引名称
CREATE TABLE 表名(
字段列表
KEY (字段1[(长度)][,字段2[(长度)]...]) ASC
);
```

- **唯一性索引**

> **注意:唯一性索引对应的字段必须要是没有重复的**

```mysql
#第一部分
	#创建单列唯一性索引
CREATE TABLE 表名(
字段列表
UNIQUE INDEX 索引名(字段1) DESC
);

CREATE TABLE 表名(
字段列表
UNIQUE KEY 索引名(字段1) DESC
);
	#创建单列唯一性索引但不指定索引名称
CREATE TABLE 表名(
字段列表
UNIQUE INDEX (字段1) DESC
);

CREATE TABLE 表名(
字段列表
UNIQUE KEY (字段1) DESC
);

#第二部分
	#创建单列唯一性索引
CREATE TABLE 表名(
字段列表
UNIQUE 索引名(字段1) DESC
);

CREATE TABLE 表名(
字段列表
UNIQUE 索引名(字段1) DESC
);
	#创建单列唯一性索引但不指定索引名称
CREATE TABLE 表名(
字段列表
UNIQUE (字段1) DESC
);

CREATE TABLE 表名(
字段列表
UNIQUE (字段1) DESC
);
```

- **主键索引**

> **主键索引只能涉及一个字段**

```mysql
#第一部分
	#创建主键索引
CREATE TABLE 表名(
字段1及相关设置 PRIMARY KEY,
字段列表
);

CREATE TABLE 表名(
字段列表,
PRIMARY KEY 索引名(字段1) ASC
);

CREATE TABLE 表名(
字段列表,
PRIMARY INDEX 索引名(字段1) ASC
);

#第二部分
CREATE TABLE 表名(
字段列表,
PRIMARY 索引名(字段1) ASC
);

CREATE TABLE 表名(
字段列表,
PRIMARY 索引名(字段1) ASC
);
```

#### 表创建后基于`ALTER`关键字设置索引

##### 创建索引

```mysql
#利用系统对某些约束的索引自动建立功能
ALTER TABLE 表名
MODIFY 字段名 [添加唯一性约束,主键约束或外键约束]

#利用表级索引创建方式
ALTER TABLE 表名
ADD [UNIQUE|FULLTEXT|SPATIAL|PRIMARY] [INDEX|KEY] [索引名](字段1[(长度)][,字段2[(长度)]...]) [ASC|DESC]
```

##### 删除索引

```mysql
#第一种
ALTER TABLE 表名
DROP	 [UNIQUE|FULLTEXT|SPATIAL|PRIMARY] [INDEX|KEY] [索引名]
```

#### 表创建后基于`CREATE INDEX ON`等语句的索引设置

##### 创建索引

```mysql
CREATE [UNIQUE|FULLTEXT|SPATIAL|PRIMARY] [INDEX|KEY] [索引名]
ON 表名(字段1[长度][,字段2[长度]...) [ASC|DESC]
```

##### 删除索引

```mysql
DROP [UNIQUE|FULLTEXT|SPATIAL|PRIMARY] [INDEX|KEY] [索引名] ON 表名
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

### 多列索引创建与使用的最左前缀原则

## `MySQL 8.0`索引新特性

### `DESC`降序索引

### 隐藏索引

> **问题**:对于我们的数据表,当我们删除某个索引后我们的系统发生了报错,我们为了解决这一报错就必须将该索引再创建回来.而这就会导致重新创建时消耗大量的系统资源
>
> **隐藏索引的优势**
>
> - ​	我们可以先设置索引为隐藏状态,然后使用一段时间系统,确保该**索引去除后不会影响**到我们系统的使用后,再**真正的将其删除.**
> - 若系统**发生了报错**,那么我们又可以**立即将该索引设置为可用状态.**
> - 这样就可以**有效地减少重新创建的开销.**
>
> **注意事项**
>
> - 我们应该尽量避免将主键设置为`隐藏索引`
>
> **特性**
>
> - ==**隐藏的索引会随着表的更新而更新**,只不过我们在**查询时系统不会利用该隐藏索引**==
> - ==在**创建表时指定隐藏索引效果与正常创建相同**,唯一的**区别就是隐藏的索引系统不会使用,而正常创建的索引会被使用**==
> - ==**隐藏的索引还是可以通过`SHOW INDEX FROM`展现出来**==
>
> **相关关键字**
>
> - `INVISIBLE`
> - `VISIBLE`

#### 创建表时创建隐藏索引

> **在正常创建的基础上添加`INVISIBLE`属性即可**

#### 表创建后基于`ALTER TABLE`创建隐藏索引

> **在正常创建的基础上添加`INVISIBLE`属性即可**

#### 表创建后基于`CREATE INDEX ON`创建隐藏索引

> **在正常创建的基础上添加`INVISIBLE`属性即可**

#### 表创建后基于`ALTER TABLE ALTER INDEX [INVISIBLE|VISIBLE]`设置字段是否隐藏

```mysql
ALTER TABLE 表名 ALTER INDEX 索引名 [INVISIBLE|VISIBLE]
```

## `MySQL`索引优化与查询优化

### 注意事项汇总

### 索引设计原则

#### 明确创建索引的13个经验

- ``经验1``:**单列字段或多列组合字段具有唯一性时**,尽量创建唯一性索引

> - 单字段时我们只要给`字段`添加`UNIQUE`唯一性约束系统就会自动建立唯一性索引
> - 多字段唯一性时,我们只要通过`UNIQUE INDEX`添加多列唯一性索引即可
> - **原因**:

- `经验2`:**经常出现在`SELECT`查询的``WHERE``子句中的单字段或多字段组合尽量创建索引**

> - 单列添加单列普通索引,多列添加多列普通索引即可,当然多列的普通索引创建时,要注意找到字段的最优顺序.
>
> - **原因**:

- `经验3`:**经常出现在`GROUP BY`或`ORDER BY`子句中的单列或多列尽量创建索引**

> - **原因**:

- `经验4`:**经常出现在`UPDATE|DELETE`语句的`WHERE`子句的单字段或多字段组合尽量创建索引**

> **原因与``场景2``相同**

- `经验5`:**经常出现在`DISTINCT`子句的单字段或多字段组合尽量创建索引**

> **原因**:我们知道我们的`MySQL`创建`B+`树索引**会保证叶子节点按照`ASC|DESC`有序的排列**,并且作为叶子节点的页还通过`双向链表`组合了起来,因此我们当我们**查询时若使用到了根据该字段建立的索引**,那么**我们的结果记录就会按照该字段有序排列**,该字段**重复的记录都会连续在一起**,这样毫无疑问我们的**去重算法就可以相比一般情况下更快地完成**

- `经验6`:**对于多表`JOIN`连接操作,我们创建索引遵循如下三个规则可以保证更好的效率**

  - **不要连接三个以上的表**,即**不要**使用**超过两个JOIN**

  > **原因**:

  - **对经常在`WHERE`子句中涉及到的单列或多列字段创建单列或联合索引**

  > **原因**:

  - **对于经常用于作为连接条件的单列或多列字段创建索引,并保证不同表之间这些字段应尽量保证类型相同**

  > **原因**:	

- `经验7`:**尽量在保证目标功能不受影响的条件下使用占用空间小的字段来创建索引**

> **原因**
>
> - ==**减小索引文件的空间占用**==,我们知道`MySQL`的`B+`树索引文件中是需要保存我们用于作为索引的字段的各个记录下的具体的值的,因此用于创建索引的字段占用空间越小,索引文件占用的存储空间就越小
> - ==**缩小索引`B+`树的规模,提升索引查询速度**==,我们考虑**在`Innodb`的以主键为索引的聚簇索引下**,如果我们主键占用的空间小,那么叶子节点的单个数据页中能够存储的由主键所指代的记录就多,这样就可以**让我们的`B+`树的规模缩小,从而使得索引速度加快**

- `经验8`:**在保证正常使用的情况下,主键使用的类型占据的存储空间越小越好**

> **原因**:对于`Innodb`存储引擎而言无论是聚簇索引,还是非聚簇索引都必须要存储各个记录的主键的值在索引文件中.**非聚簇索引是因为要根据主键值进行聚簇索引表的回表操作**,因此主键占用的空间越小,数据表对应的索引文件就越小

- `经验9`:**尽量在使用`CHAR,VARCHAR,TEXT`等字符串类型的字段创建索引时,尽量使用``字段名 长度``指定好该字段的前`长度`个字符用于索引的创建**

> **原因**:
>
> **注意事项**:
>
> **问题**:
>
> ​	<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-43?token=AOAPFCK4NPFSQF6QNDA2E6DC57YOY" alt="image-20220717234145869" style="zoom: 67%;" />

- `经验10`:**在不影响正常使用情况下,尽量使用区分度高的字段创建索引**
- `经验11`:**在非特殊情况下,创建联合索引时,尽量将区分度最高的字段放在第一位**
- `经验12`:**创建联合索引时,尽量将使用最频繁的字段放在第一位**
- `经验13`:**在多个字段都要创建索引时,尽量将他们用联合索引组合起来**

#### 限制单表的索引数目

> ![image-20220717235625992](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-44?token=AOAPFCMREMHSTVPRCYNKCN3C57YPO)

#### 明确七个不适合创建索引场景

- `场景1`:**不出现在`WHERE|GROUP BY|ORDER BY`的单列或多列字段尽量不要加索引**

> **原因**:索引的作用就是让我们通过`B+`树进行快速的记录查找,若不涉及到使用该单列或多列字段对记录的筛选,我们没必要对其设置索引

- `场景2`:**数据量少的数据表不要使用索引**
- `场景3`:**区分度低的字段尽量不要作为索引的创建字段**

> **如**:一个学生信息表中的性别字段,我们对该字段建立单列索引是没有多大作用的,原因我们可以从`B+`索引树的建立过程很容易推知

- `场景4`:**经常需要更新的表我们尽量少创建索引**
- `场景5`:**尽量不要用无序的字段建立索引**
- `场景6`:**及时删除不经常使用的索引**
- `场景7`:**不要定义重复的索引**

> **我们对``字段1``建立了一个单列索引,然后还按照`字段1,字段2`的顺序建立一个双列索引.很显然我们的双列索引完全可以起到单列索引的作用,因此我们的单列索引是重复的,不必要存在**

## `MySQL`中的隐式转换

## `MySQL`性能分析工具

> 我们必须明确,对数据库进行性能分析的目的在于让我们的数据库``吞吐量提升,响应速度加快``

### 基本知识

#### 数据库服务器的一般优化步骤

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-45?token=AOAPFCJGEXJWDZ6NAE3TE33C57YP6" alt="image-20220718103836807" style="zoom: 40%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-46?token=AOAPFCK4M2KKU2H4JX5IGGLC57YQU" alt="image-20220718103931131" style="zoom: 42%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-47?token=AOAPFCPBHKWT7ULTTUJJWEDC57YRC" alt="image-20220718104102011" style="zoom:40%;" />

#### 数据库优化的代价

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-48?token=AOAPFCNEAYFYD3WZAYRK27LC57YRO" alt="image-20220718104156713" style="zoom:50%;" />

### 数据库性能参数的查看

#### 常用`status`性能参数

> 查询方法
>
> ```mysql
> SHOW STATUS LIKE "参数名"
> ```

- `Connections`:`MySQL`服务器启动以来被客户端连接的总次数
- `Uptime`:`MySQL`服务器上线的时间
- `Slow_queries`:`MySQL`服务器启动以来慢查询的次数
- `Innodb_rows_read`:`MySQL`服务器启动以来所有`SELECT`语句返回的记录总数
- `Innodb_rows_inserted`:`MySQL`服务器启动以来执行`INSERT`指令插入的记录总数
- `Innodb_rows_updated`:`MySQL`服务器启动以来执行`UPDATE`指令更新的记录总数
- `Innodb_rows_deleted`:`MySQL`服务器启动以来执行`DELETE`指令删除的记录总数
- `Com_select`:`MySQL`服务器启动以来查询操作的总次数
- `Com_insert`:`MySQL`服务器启动以来插入操作的总次数(一次插入多个记录只算一次插入)
- `Com_update`:`MySQL`服务器启动以来更新操作的总次数
- `Com_delete`:`MySQL`服务器启动以来删除操作的总次数
  - **注意**:**当`MySQL`服务器重启后这些参数都会被重置**

### 通过`last_query_cost`查看上一条`SQL`语句的成本

> `MySQL`解析器,优化器会针对我们的`SQL`语句做出多种操作方式,然后通过对每一条执行方式的消耗的方式,找到代价最小的执行方式来进行执行,而其**评估标准就是该操作需要读取的页的数目**

```mysql
SHOW STATUS LIKE "last_query_cost"
```

![image-20220718105958705](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-49?token=AOAPFCOZXXCY762ILEBUDV3C57YSK)

### 合理利用慢查询日志

> **慢查询指那些执行时间超过我们的`long_query_time`系统变量指定的时长的查询语句**
>
> **慢查询日志的作用即,若一条查询被定义为慢查询,那么其就会被写入我们的慢查询日志.我们后续就可以根据慢查询日志有针对性地对数据库进行优化**
>
> **注意**
>
> - 默认情况下`MySQL`并不会开启慢查询日志,因为开启后其会带来额外的系统开销

临时设置

- **开启与关闭慢查询日志**

  ```mysql
  #查看慢查询日志是否开启
  SHOW VARIABLES LIKE "%slow_query_log%";
  #开启慢查询日志
  SET GLOBAL slow_query_log = ON;
  #关闭慢查询日志
  SET GLOBAL slow_query_log = OFF;
  #查看慢查询日志的存储位置
  SHOW VARIABLES LIKE "%slow_query_log%";
  ```

- **设置`long_query_time`**

  ```mysql
  #查看lone_query_time 
  SHOW VARIABLES LIKE "lone_query_time"
  #设置
  SET long_query_time = <时间>
  ```

#### 永久设置

### 借助`mysqldumpslow`工具优化慢查询

### 借助`SHOW PROFILE`查看我们的查询语句的消耗

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

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-50?token=AOAPFCKKRZV6ONKUJKN43UTC57YTG" alt="image-20220718113201072" style="zoom:67%;" />

### ==**借助`EXPLAIN`查看我们查询语句使用的执行计划**==

> **注意**
>
> - `MySQL 5.6.3`之前的版本**只能查看`SELECT`语句的执行计划**
> - `MySQL5.6.3`之后**可以查看`SELECT|UPDATE|DELETE`语句的执行计划**
> - **加了`EXPLAIN`前缀后我们的语句不会真正执行,而是只会执行到优化器与解析器指定出执行计划那一个步骤就会将执行计划返回给我们.而后续的真正执行则不会继续进行**
> - `EXPLAIN`得到的结果是我们的语句对于涉及到的各个表的操作计划(而一条记录只能保存对于一个表的操作计划),因此一个`EXPLAIN`可能会返回多个记录,这些记录分别记载着对于不同的表的操作计划.

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
| `key_len`       | ![image-20220718140207735](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL进阶-51?token=AOAPFCLCUD7SS5YLIWJAPT3C57YTW)<br /><img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-52?token=AOAPFCMA6B7RH7SZLCBGZVLC57YUY" alt="image-20220718140325799" style="zoom: 50%;" /> |
| `ref`           | ![image-20220718140620704](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-53?token=AOAPFCNBMPY2GW644PULOMTC57YVO) |
| `rows`          | <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-54?token=AOAPFCMID2VUXIVWJSRQPLTC57YWK" alt="image-20220718140640082" style="zoom:67%;" /> |
| `filtered`      | ![image-20220718140748699](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-55?token=AOAPFCLFB7T3FL6AVKKP5PLC57YW6) |
| `Extra`         | 一些额外的信息,`MySQL`默认的额外信息字段有几十个             |

#### `select_type`

- | 名称                   | 描述                                                         |
  | ---------------------- | ------------------------------------------------------------ |
  | `SIMPLE`               | 简单的单表查询                                               |
  | `PRIMARY`              | 多表查询时的主查询                                           |
  | `UNION`                | 涉及两个查询`UNION`时的第二个查询                            |
  | `UNION RESULT`         | `UNION`联合的临时表                                          |
  | `SUBQUERY`             | 子查询语句的第一个非主查询                                   |
  | `DEPENDENT SUBQUERY`   | 相关子查询的第一个非主查询                                   |
  | `DEPENDENT UNION`      | 相关`UNION`                                                  |
  | `DERIVED`              | 当子查询中,用查询结果作为表进入`FROM`子句,那么这个查询为`DERIVED`类型 |
  | `MATIERIALIZED`        | 作为子查询出现并且被物化                                     |
  | `UNCACHEABLE SUBQUERY` | ![image-20220718134441970](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\07MySQL进阶.assets\image-20220718134441970.png) |
  | `UNCACHEABLE UNION`    | ![image-20220718134451830](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\07MySQL进阶.assets\image-20220718134451830.png) |

#### `type`

- `system`
  - 在表中只有一条记录,并且存储引擎的统计数据十分准确时
  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-56?token=AOAPFCNZK3XWVN4SHQXWJ33C57YXG" alt="image-20220718135351869" style="zoom:50%;" />
- `const`
  - ![image-20220718135310246](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-57?token=AOAPFCLIMFIGBFPK5W3JXC3C57YX2)
- `eq_ref`
  - ![image-20220718135253779](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-58?token=AOAPFCMWIXJNJGH2DGXLCV3C57YYI)
- `ref`
  - ![image-20220718135426494](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-59?token=AOAPFCLQNFL55ZWZCZLRJY3C57YY6)
- `fulltext`
  - 
- `ref_or_null`
  - ![image-20220718135451139](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-60?token=AOAPFCP327P4HW7N72RYNLDC57YZK)
- `index_merge`
  - 
- `unique_subquery`
  - ![image-20220718135543866](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-61?token=AOAPFCKDFPKBCXRHQSDX3OLC57Y2A)
- `index_subquery`
  - 
- `range`
  - ![image-20220718135641636](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-62?token=AOAPFCKS26SOSUHN57OGDCTC57Y2O)
- `index`
  - ![image-20220718135715793](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-64?token=AOAPFCMB2LKC7TASNZ3Y7W3C57Y32)
- `ALL`
  - 对数据表的每一条记录进行直接遍历
- ![image-20220718135827012](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-63?token=AOAPFCOAUGTF2H5SWOHIY4DC57Y3C)

#### `Extra`

- `No tables used`:查询语句没有定义`FROM`子句
- `Impossible WHERE `:我们的`WHERE`子句的条件是永假的
- `Using WHERE`:
  - ![image-20220718170222163](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-65?token=AOAPFCJ23UYRZ7WNVIKAHJLC57Y4O)
- `No Matching min/max row`
  - ![image-20220718170505744](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-66?token=AOAPFCM4X3TAGOD3XQR3UZ3C57Y5C)
- `Using Index`
  - ![image-20220718170532210](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-67?token=AOAPFCLBLUDRRPRC7OGRJTTC57Y5Q)
- `Using Index condition`
  - ![image-20220718170648735](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-68?token=AOAPFCIMWDEJ5QLASOBHZWDC57Y6A)
- `Using join buffer`
  - ![image-20220718170847773](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-69?token=AOAPFCKGEMMC34OTE2S4RRDC57Y6U)
- `Not exists`
  - ![image-20220718170936576](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-70?token=AOAPFCMH57R3LXIBNUZCSYLC57Y7I)
- `Using union`
  - ![image-20220718171048128](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-71?token=AOAPFCN2PMMITVATTGUOPHLC57Y7Y)
- `Zero limit`
  - ![image-20220718171059821](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-72?token=AOAPFCNMFCDI334KOJKENJ3C57ZAK)
- `Using filesort`
  - ![image-20220718171146693](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-73?token=AOAPFCPKOSZTS7FNHZTCTITC57ZBA)
- `Using temporary`
  - ![image-20220718171254121](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-74?token=AOAPFCI5N4OSC2KHQRCCCU3C57ZBQ)

### `EXPLAIN`拓展

#### `EXPLAIN`的四种输出格式

##### 传统格式

- **使用方式**

  ```mysql
  EXPLAIN 语句
  ```

  

- ![image-20220718171638694](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-75?token=AOAPFCPOBD5PH5IXJ2TIWQLC57ZCG)

##### `JSON`格式

> **其特点在于相对于传统格式可以呈现出我们的查询语句的各种成本**

- **使用方式**

  ```mysql
  EXPLAIN FORMAT = JSON 语句
  ```

- ![image-20220718172003683](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-76?token=AOAPFCI2GLNBMF6WH33FCJTC57ZC2)

##### `TREE`格式

> TREE格式是`MySQL8.0.16`版本之后引入的新格式，主要根据查询的`各个部分之间的关系`和`各部分的执行顺序`来描述如何查询。

- **使用方法**

  ```mysql
  EXPLAIN FORMAT = TREE 语句
  ```

##### 可视化输出

> 可视化输出，可以通过``MySQL Workbench`可视化查看`MySQL`的执行计划。

- ![image-20220718172556313](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-77?token=AOAPFCJCPYDVQDGSQ27VHRDC57ZDM)

#### `EXPLAIN`与`SHOW WARNINGS`组合

> 当我们`EXPLAIN`一条语句后,可以通过`SHOW WARNINGS`获取到该语句在**经过解析器优化器优化后的真正用于执行的语句**

![image-20220718172857250](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-78?token=AOAPFCMQPOBEN5NUEY23HVTC57ZEE)

### `TRACE`优化器执行计划制定的决策跟踪工具的使用

> 

```mysql
#这里设置的是会话系统变量,因此如果更换一个会话就需要重新再开启
	#开启
SET optimizer_trace="enabled=on",end_markers_in_json=on; 
	#设置大小
set optimizer_trace_max_mem_size=1000000;
#使用
select * from student where id < 10;
select * from information_schema.optimizer_trace\G
```

### `MySQL`监控分析视图`sys schema`

> ![image-20220718174023460](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-79?token=AOAPFCJEQMZGGQ5KXCWWCN3C57ZE4)

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-80?token=AOAPFCPS65KO3JV3LCQI6FTC57ZFM" alt="image-20220718174048536" style="zoom:50%;" />

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

## `MySQL`索引优化与查询优化

### 前言

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-81?token=AOAPFCOEIDKJFKO7ZKZLCLDC57ZGA" alt="image-20220718174550128" style="zoom: 50%;" />

### 索引失效的``11``种情况

#####   `情况1`:全值匹配我最爱

![image-20220718195623345](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-82?token=AOAPFCOMBERDHPXSW5RPCLLC57ZGM)

#####  `情况2`:违反最左前缀原则

> - 在``MySQL``建立联合索引时会遵守最左前缀匹配原则，即最左优先，在检索数据时从联合索引的最左边开始匹配。
>
>
> - **结论**：``MySQL``可以为多个字段创建索引，一个索引可以包括``16个字段``。对于多列索引，==**过滤条件要使用索引必须按照索引建立时的顺序，依次满足，一旦跳过某个字段，索引后面的字段都无法被使用。**==如果查询条件中没有使用这些字段中第1个字段时，多列（或联合）索引**不会被使用**。
> - **注意**:**虽然我们的优化器可能会帮我们优化,但是我们能够自己解决的问题应该尽量不要依靠优化器,那样不稳定**
>

![image-20220718195845140](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-83?token=AOAPFCLTRUFERSMIXN24IP3C57ZHE)

#####   `情况3`:尽量保证按照主键逐步增大的插入顺序对记录进行插入

> 对于一个使用`InnoDB`存储引擎的表来说，在我们没有显示的创建索引时，表中的数据实际上都是存储在`聚簇索引`的叶子节点的。而记录又存储在数据页中的，数据页和记录又是按照记录`主键值从小到大`的顺序进行排序，所以如果我们`插入`的记录的`主键值是依次增大`的话，那我们每插满一个数据页就换到下一个数据页继续插，而如果我们插入的`主键值忽小忽大`的话，则可能会造成`页面分裂`和`记录移位`。
>

#####  `情况4`:计算、函数、类型转换(自动或手动)导致索引失效

> - **很显然,当我们可以索引的字段被函数(计算,类型转换)进行处理后,**==**极大可能导致值发生变化**==**,而变化后的值是没有被建立索引的(同样的类型转换也会涉及到字段的值发生改变).因此为了顾全大局,`MySQL`便默认被作为函数输入的字段,类型转换的字段,计算的字段无法作为索引的字段**
>
> **如**:
>
> - `SELECT * FROM Students WHERE ((age/10)+5) =15;`

![image_73](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-84?token=AOAPFCNIPMJKTIE5YQU2NTLC57ZHS)

![image_75](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-85?token=AOAPFCJXR73OHOUNU7JXCM3C57ZIG)

#####  `情况5`:联合索引中涉及到范围条件的字段的右边的字段在不使用索引下推的情况下无法使用索引

> **如**:按照`字段1,字段2,字段3`建立联合索引
>
> - `字段1`涉及范围,那么``字段1``可以使用索引,``字段2,字段3``无法使用
> - `字段2`涉及范围,那么``字段1,字段2``可以使用索引,``字段3``无法使用
> - ``字段3``涉及范围,都可以使用索引
>
> 应用开发中范围查询，例如：金额查询，日期查询往往都是范围查询。
>
> **有效的避免手段**
>
> - **所以首先我们编写查询语句时应尽量将查询中的范围条件作为``WHERE``语句最后一个条件**
> - **其次创建联合索引时,我们务必要把涉及范围条件的字段写在最右**
>
> **原因**:
>
> - 由于我们的`B+`树联合索引是按照先按最左字段排序,最左字段相同就按次左字段排序,再相同就按再次左字段进行排序.因此`MySQL`的`B+`树下的联合索引必须遵循最左前缀原则,并且满足最左前缀后也要次左字段能够使用索引才能轮到再次左字段使用索引.这一切都是由我们的`B+`树索引的创建规则决定好了的
> - 而我们上面的情况`classId`使用了范围条件,范围条件由于可能涉及多个值因此在这一步`MySQL`会索引到大量的记录,**再次左字段在不开启索引条件下推的情况下,面对如此多的字段,便使用不了索引**
> - 而当我们改变联合索引中的字段的顺序后最左字段与次左字段就可以使用到索引了
> - **究其一切,这些都是`B+`树构建联合索引时遵守的规则带来的问题,且只要使用`B+`树索引就无法解决**

![image-20220718203510393](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-86?token=AOAPFCPWW5PYB54LK7XDTKLC57ZIY)

#####  `情况6`:不等于(!= 或者<>)索引失效

#####  `情况7`:``is null``可以使用索引，``is not null``无法使用索引

> 结论：最好在设计数据表的时候就将`字段设置为 NOT NULL 约束`，比如你可以将INT类型的字段，默认值设置为0。将字符类型的默认值设置为空字符串('')
>
> 拓展：同理，在查询中使用`not like`也无法使用索引，导致全表扫描

#####  `情况8`:like以通配符%开头索引失效

> 拓展：``Alibaba``《``Java``开发手册》
>
> 【强制】页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎来解决。

##### `情况9`:OR 前后存在非索引的列，索引失效

> 在``WHERE``子句中，如果在``OR``前的条件列进行了索引，而在``OR``后的条件列没有进行索引，那么索引会失效。也就是说，**OR前后的两个条件中的列都是索引时，查询中才使用索引。**
>

#####  `情况10`:数据库和表的字符集统一使用``utf8mb4``

> 统一使用``utf8mb4`( `5.5.3`版本以上支持)兼容性更好，统一字符集可以避免由于字符集转换产生的乱码。**不同的`字符集`下的字符在进行比较前需要进行字符集`转换`,这种转换很可能会造成索引失效**。
>

### 经验总结

- **``经验1``:在不开启索引条件下推的条件下只要是联合索引的字段中不是最后一个字段涉及到范围,就必然会导致其后的字段使用不到索引**
- `经验2`:**`IS NOT NULL,<>,!=,NOT LIKE`**我们稍加思考就知道在这些关键字下我们无法获取一个准确的值取在`B+`树中进行查找
  - 而类似`WHERE age>18`的过滤条件,我们完全可以用``18``取检索`B+`索引树,然后利用叶子节点之间会用`双向链表连接起来`的特性来遍历链表获取符合条件的记录

- `经验3`:`LIKE`模式匹配不允许左模糊与全模糊是由于**`B+`树索引对于字符串类型的字段的排序规则是,先比第一个字符第一个字符的``编码``越靠前其排序优先级越大,第一个字符优先级相同才比较第二个,第二个相同才比较第三个**.因此如果我们使用了左模糊或全模糊,显然就完全使用不了`B+`树索引

- `经验4`:`情况9`发生的原因在于,

### 索引使用的建议

![image-20220719135636862](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\07MySQL进阶.assets\image-20220719135636862.png)

### 练习题

> **前提**:以`a,b,c`的顺序建立了联合索引

![image-20220719135446674](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-87?token=AOAPFCOE6YUOKGWCXYBCI3DC57ZJU)

### ==**多表查询优化**==

> **前提**
>
> - 对于内连接来说，**驱动表的每一条记录都会被遍历(不这样没法生成过滤的笛卡尔积，这是必须的)**，因此驱动表的连接条件的字段即便有索引也用不到
> - 对于内连接来说，查询优化器可以决定谁来作为驱动表，谁作为被驱动表出现
> - 对于内连接来讲，如果表的连接条件中**只有一个字段有索引**，则**有索引的字段所在的表会被作为被驱动表**
> - **对于内连接来说，在两个表的连接条件都存在索引的情况下，会选择小表作为驱动表。`小表驱动大表`**

#### 经验总结

![image-20220719155415999](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-88?token=AOAPFCM6NRPD3HOMQAACGSTC57ZKM)

#### 驱动表与被驱动表介绍

> **无论是内连接还是外连接,我们都无法直接判断哪个表最后会被作为驱动表,哪个表会被作为被驱动表,因为我们的优化器会综合代价来决定哪个表做驱动表,哪个表做驱动表**

#### `JOIN`在没有索引时的实现原理

##### 内连接

- 首先遍历取出驱动表的所有记录,以及被驱动表的所有数据
- 取出驱动表的一条记录,**并遍历所有的被驱动表记录**,满足连接条件的就拼接起来
  - 一条驱动表记录可以拼接多条被驱动表记录
- 重复第二步直到遍历完驱动表的所有记录
- 至此就得到了我们`JOIN`的结果

##### 外连接

- 首先遍历取出驱动表的所有记录,以及被驱动表的所有数据
- 取出驱动表的一条记录,**并遍历所有的被驱动表记录**,满足连接条件的就拼接起来
  - 一条驱动表记录可以拼接多条被驱动表记录
- 重复第二步直到遍历完驱动表的所有记录
  - 然后添加左(右)表中没有被拼接过的记录

- 至此就得到了我们`JOIN`的结果

#### `JOIN`语句在具有索引的情况下的实现原理

- 首先遍历取出驱动表的所有记录
- 然后取出驱动表的一条记录,**根据连接条件字段的索引对被驱动表的索引文件进行检索**找到可以满足连接条件的被驱动表记录
  - 一条驱动表记录可以拼接多条被驱动表记录
- 重复第二步直到遍历完驱动表的所有记录
- 至此就得到了我们`JOIN`的结果

#### 有索引与无索引的区别

> 有索引时就可以不用遍历被驱动表,而无索引的情况下就需要遍历整个被驱动表

#### `Simple Nested-Loop Join`嵌套循环连接

##### 原理

> **与我们前面讲的无索引情况下的`JOIN`工作原理相同**

![image-20220719153407749](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-89?token=AOAPFCKZVXSMTM2RHL5A2W3C57ZLA)

##### 开销统计

![image-20220719152737858](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-90?token=AOAPFCKUTOPC3MLBVKEQXSLC57ZLW)

####  `Index Nested-Loop Join`(索引嵌套循环连接)

##### 原理

> **与我们前面讲的有索引情况下的`JOIN`工作原理相同**

​	

##### 开销统计

![image-20220719152813369](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-91?token=AOAPFCIB6GTMZB5IDEFXG5LC57ZMG)

#### `Block Nested-Loop Join`(块嵌套循环连接)

##### 基本原理

> **前提**:
>
> - 如果`Index Nested-Loop Join`可用那么是不会使用这种方式的
> - 这种方式通过空间换时间的方式改进了`Simple Nested-Loop Join`,提高了速度
>
> **算法的开发背景**:
>
> - 在使用`Simple Nested-Loop JOIN`的条件下被驱动表要进行巨量的扫描操作,并且每次访问被驱动表，其表中的记录都会被**加载到内存中**，**然后再从驱动表中取一条与其匹配**，**匹配结束后清除内存**，然后再从驱动表中加载一条记录，然后把被驱动表的记录**再加载到内存匹配**，这样周而复始，大大增加了``IO``的次数。为了减少被驱动表的``IO``次数，就出现了``Block Nested-Loop Join``的方式。
>
> **算法原理**:
>
> - ``Block Nested-Loop Join``不再是逐条获取驱动表的数据，而是一块一块的获取，引入了`Join Buffer缓冲区`，将驱动表``Join``相关的部分数据列的我们要查询的字段载入`Join Buffer`(载入的记录数量受``Join Buffer大小``的限制)缓存到``Join Buffer``中，然后**全表扫描被驱动表**，**被驱动表的每一条记录一次性和``Join Buffer`中的所有驱动表记录进行匹配（内存中操作**），将简单嵌套循环中的多次比较合并成一次，**降低了被驱动表的访问频率**。

![image-20220719153604165](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-92?token=AOAPFCM77CW25UFY5AQYC3LC57ZNG)

##### 开销统计

![image-20220719153622155](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-93?token=AOAPFCL77IC5FGOMKP5UDEDC57ZNU)

##### 开启关闭以及`Join Buffer`大小设置

![image-20220719155159013](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-94?token=AOAPFCK2TZ2YKTT3V6AALCLC57ZOS)

#### `Hash Join`

> **`Hash Join`在`MySQL 8.0.20`之后引入了`Hash Join`并且将其设置为默认的连接方式**
>
> **注意**:
>
> - 连接条件非等值时还是得老实使用`Nested-Loop Join`
> - 可以使用`Index Nested-Loop Join`且数据量很大时还是使用`Index Nested-Loop Join`更好 

##### 基本原理

> ![image-20220719155750392](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-95?token=AOAPFCMXPB5JBWPZOL65LPLC57ZPI)

##### 开销统计

##### 与`Nested-Loop Join`的对比

![image-20220719155908766](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-96?token=AOAPFCMYOBJ54GYCAGUGQODC57ZQE)

### 子查询优化

#### 子查询低效的内在原因

> - 执行子查询时，``MySQL``需要**为内层查询语句的查询结果`建立一个临时表`**，然后**外层查询语句从临时表中查询记录**。查询完毕后，再`撤销这些临时表`。这样会消耗过多的CPU和IO资源，**产生大量的慢查询**。
>
>
> - 子查询的结果集存储的临时表，**不论是内存临时表还是磁盘临时表都`不会存在索引`**，所以查询性能会受到一定的影响。
>
>
> - 对于返回结果集比较大的子查询，其对查询性能的影响也就越大。

#### 解决方案

> 我们应该**尽量通过多表连接查询的方式来改写我们的子查询语句**,其优点在于
>
> - 无需建立临时表
> - 由于是多表查询,因此有机会使用到索引

#### 优化实例

> **相比于子查询快了一倍多**

![image-20220719160447667](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-97?token=AOAPFCKUUUJIYL2M2ZFANYLC57ZQS)

#### 不要使用`NOT IN`与`NOT EXISTS`

##### 原因

##### 解决方案

> 用``LEFT JOIN 表名 ON 连接条件 WHERE 过滤条件 IS NULL``替代

### `ORDER BY`排序优化

#### 经验总结

- `SQL`中，可以在``WHERE``子句和``ORDER BY``子句中使用索引，目的是在``WHERE``子句中 **避免全表扫描**，在``ORDER BY`子句**避免使用`FileSort`排序**。

- 某些情况下全表扫描，`FileSort`排序不一定比索引慢。但总的来说，我们还是要避免，以提高查询效率。

- 尽量使用``Index``完成``ORDER BY``排序。如果`WHERE`和``ORDER BY``后面是相同的列就使用单索引列；如果不同就使用联合索引。

- 无法使用``Index``时，需要对``FileSort``方式进行调优。

#### 问题汇总

##### `LIMIT`的使用对于排序优化的影响

> **前提**
>
> - 数据表的数据量很大
> - 辅助排序的索引字段不是主键
>
> ![image-20220719171915036](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-98?token=AOAPFCP7R3OR43XXUPPVEUDC57ZRE)

- **查询的字段并没有全部出现在数据表,需要回表时**

- **查询的字段全部出现在索引表时**

##### `ORDER BY`时用于排序的字段的顺序使用错误导致使用不上索引

> ![image-20220719172033966](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-99?token=AOAPFCLNU7UO7DS3LYC5CQ3C57ZRU)

- **原因**
  - **第一,二个失效原因是其不满足最左前缀规则**
  - **后面三个都用上了索引**

##### `ORDER BY`涉及的字段的联合索引按照`ASC|DESC`排序,而我们`ORDER BY`中却是使用`DESC|ASC`

> ![image-20220719183956337](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-100?token=AOAPFCN6OTOKHCXNZOZ37E3C57ZSE)

#####   无过滤不索引

![image-20220719184236462](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-101?token=AOAPFCNR4GHOR3GJHO2VSMLC57ZSW)

#### **总结**

![image-20220719184353983](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-102?token=AOAPFCIYHXIAOBSTJNIK2PLC57ZTI)

#### **拓展**

![image-20220719191214903](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-103?token=AOAPFCIDHXJ3FQRZ75PGXHTC57ZT4)

##### 问题

- **我们发现方案2本来还可以继续做索引条件下推,但是`MySQL`直接跳出来不对`NAME`继续索引下推,而是直接出来做`FileSort`,其原因在于经过前面的过滤后剩下的要排序的字段很少了,排序很快,索引优化器制定了这样的执行计划**

![image-20220719184531262](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-104?token=AOAPFCM7W7RPAZNXSOSYGNLC57ZUM)

### `FileSort`算法介绍

#### **双路排序**

![image-20220719184738451](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-105?token=AOAPFCL2QO3OBT74ER5ES53C57ZVG)

#### **单路排序**

![image-20220719184747212](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-106?token=AOAPFCJQI7BCLP2HBMOOCM3C57ZVY)

#### **结论与问题讨论**

![image-20220719184812919](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-107?token=AOAPFCMPQCWC2G6MZ2S3WRDC57ZXE)

#### **优化方法**

![image-20220719185002583](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-108?token=AOAPFCOOZNJBEPY6SI5ZBJDC57ZYK)

![image-20220719185017203](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-109?token=AOAPFCKCEFV5PLGX6MW75RLC57ZZC)

### `GROUP BY`分组优化

- `GROUP BY`使用索引的原则几乎跟`ORDER BY`一致 ，`GROUP BY`即使没有过滤条件用到索引，也可以直接使用索引。
- `GROUP BY`**先排序再分组**，遵照索引建的最佳左前缀法则
- 当无法使用索引列，可以增大`max_length_for_sort_data`和`sort_buffer_size`参数的设置
  - **因为`GROUP BY`要进行`FileSort`排序,所以扩大这两个参数有利于`FileSort`**
- `WHERE`效率高于`HAVING`，能写在`WHERE`限定的条件就不要写在`HAVING`中了
  - **如`Having name="汤凌"`完全可以改写为`WHERE name="汤凌"`这样过滤掉了非`name="汤凌"`的字段,所以得到的所以分组都会可以满足`HAVING name="汤凌"`这个条件**
- 减少使用`ORDER BY`，和业务沟通能不排序就不排序，或将排序放到程序端去做。`ORDER BY`、`GROUP BY`、`DISTINCT`这些语句较为耗费`CPU`，数据库的`CPU`资源是极其宝贵的。
- 包含了`ORDER BY`、`GROUP BY`、`DISTINCT`这些查询的语句，`WHERE`条件过滤出来的结果集请保持在`1000`行以内，否则`SQL`会很慢。

### `LIMIT`分页查询优化

![image-20220719191930891](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-110?token=AOAPFCKZBXMMQSZZQCFUVI3C57ZZY)

![image-20220719191940950](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-111?token=AOAPFCOQ3WCHCOX6P7KWPL3C57Z2G)

### 有效利用覆盖索引

> **覆盖索引**:索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据；当能通过读取索引就可以得到想要的数据，那就不需要读取行了。**一个索引包含了满足查询结果的数据就叫做覆盖索引。**

#### 覆盖索引的优缺点

##### 优点

- **避免Innodb表进行索引的二次查询（回表）**

- **可以把随机IO变成顺序IO加快查询效率**

##### 缺点

- `索引字段的维护`总是有代价的。因此，在建立冗余索引来支持覆盖索引时就需要权衡考虑了。这是业务DBA，或者称为业务数据架构师的工作。

### 合理利用索引条件下推原则

#### 索引下推`ICP`的特性

![image-20220719130634915](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-112?token=AOAPFCN5KMPTWNVYRV675LDC57Z3A)

- **注意**:这里提到的`WHERE`条件可以仅使用索引中的列进行筛选,指的是由于我们的联合索引文件中会保存各个记录的我们用于建立联合索引的字段的具体的值,因此在索引文件中这些字段的值就会存在,而不用回表来获取

#### 可以利用索引条件下推`ICP`的例子

- <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-113?token=AOAPFCKWJJTLASP55UI543DC57Z3O" alt="image-20220719132834547" style="zoom:67%;" />

- <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-114?token=AOAPFCMWZYC7VXPCED7DBPDC57Z4A" alt="image-20220719132811140" style="zoom: 67%;" />

#### 开启与关闭索引条件下推`ICP`

```mysql
#临时设置
	#关闭
SET optimizer_switch = "index_condition_pushdown=off"
	#开启
SET optimizer_switch = "index_condition_pushdown=on"

#永久设置
	#关闭
[mysqld]
optimizer_switch = "index_condition_pushdown=off"
	#开启
[mysqld]
SET optimizer_switch = "index_condition_pushdown=on"
```

#### 使用索引条件下推的前提条件

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-115?token=AOAPFCNATOG3L6BDPYSC7QLC57Z4Q" alt="image-20220719133757984" style="zoom: 67%;" />

### `SQL`调优拓展

#### `EXISTS` 和 ``IN`` 的选择

> - `IN`是把外表和内表作``HASH连接``，而``EXISTS``是对外表作``Loop循环``，每次``Loop循环``再对内表进行查询，一直以来认为`EXISTS`比`IN`效率高的说法是不准确的。
>
> - **如果查询的两个表大小相当，那么用``IN``和`EXISTS`差别不大；如果两个表中一个较小一个较大，则子查询表大的用`EXISTS`，子查询表小的用`IN`；**

#### `COUNT(*)`与``COUNT(具体字段)``效率对比

- **环节1：**`COUNT(*)`和`COUNT(1)`都是对所有结果进行`COUNT`，`COUNT(*)`和`COUNT(1)`本质上并没有区别（二者执行时间可能略有差别，不过你还是可以把它俩的执行效率看成是相等的）。如果有``WHERE``子句，则是对所有符合筛选条件的数据行进行统计；如果没有``WHERE``子句，则是对数据表的数据行数进行统计。

- **环节2：**如果是``MyISAM``存储引擎，统计数据表的行数只需要`O(1)`的复杂度，这是因为每张``MyISAM``的数据表都有一个``meta``信息存储了`row_count`值，而一致性则是由表级锁来保证的。如果是``InnoDB``存储引擎，因为``InnoDB``支持事务，采用行级锁和``MVCC``机制，所以无法像``MyISAM``一样，维护一个``row_count``变量，因此需要采用`扫描全表`，是`O(n)`的复杂度，进行循环+计数的方式来完成统计。

- **环节3：**在``InnoDB``引擎中，**如果采用`COUNT(具体字段)`来统计数据行数，要尽量采用二级索引。因为主键采用的索引是聚簇索引，聚簇索引包含的信息多，明显会大于二级索引（非聚簇索引）**。对于`COUNT(*)`和`COUNT(1)`来说，它们不需要查找具体的行，只是统计行数，系统会**自动采用占用空间更小的二级索引来进行统计**。如果有多个二级索引，会使用``key_len``小的二级索引进行扫描。当没有二级索引的时候，才会采用主键索引来进行统计。

#### `SELECT(*)`的使用

##### 原因

- 这样一般**无法利用到覆盖索引**
- `MySQL`在解析的过程中,需要通过查询系统维护的`数据字典`来将``*``按序转换成所有列名，这会大大的耗费资源和时间。

#### `LIMIT 1`对于速度的影响

> 考虑这样一种情况,我们有一张学生信息表,其中学生名字列不是`UNIQUE`约束字段并且没有建立索引,但是我们作为老师知道目前为止数据库中不存在重名的学生,那么对于`SELECT * FROM Students WHERE name="汤凌"`我们就可以加上`LIMIT 1`来提高查询效率

##### 原因

- 若我们不添加`LIMIT 1 `那么我们的查询语句就一定会遍历表中的每一条记录来完成我们的查询
- 但是如果添加了`LIMIT 1`那么当遍历时查到了一条`name="汤凌"`的记录,那么查询语句就会由于`LIMIT 1`的存在而直接终止查询,直接返回这一条记录.显然这样就节约了一定时间

#### 养成多使用`COMMIT`的习惯

> 由于在`COMMIT`之前我们**对数据表做的操作都只是存储在内存上而并不会写入磁盘**的,因此`MySQL`**为了保证安全等需要,会消耗一定的系统资源来维护我们做的这些操作**.而若我们调用了`COMMIT`这些操作就会**被写入磁盘,从而也就可以释放掉用于维护它们的系统资源**

##### `COMMIT`可以释放的资源

- 回滚段上用于恢复数据的信息

- 被程序语句获得的锁

- `redo / undo log buffer` 中的空间

- 管理上述 3 种资源中的内部花费

## `MySQL`主键设计

> **主键的设计十分重要,要考虑隐私问题,安全问题,稳定性问题等等,如果主键设计的不合理,会导致众多严重的问题**

### 自增`ID`做主键的问题

> **不合适**

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-116?token=AOAPFCMT7B62ZZP5E54UWADC57Z5M" alt="image-20220719192617279" style="zoom: 65%;" />

### 业务字段做主键

> **也难以进行**

#### 会员卡号做主键

> **结论:不好**

#### 会员电话号码或身份证号做主键

> ![image-20220719193016331](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-117?token=AOAPFCMQFZJ2X76JZUXWGJTC57Z56)

### 淘宝的主键设计

![image-20220719193137942](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-118?token=AOAPFCNX4VOZVNU46EXXIQTC57Z6O)

### **主键设计方法论**

![image-20220719193320591](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-120?token=AOAPFCLS36WTBTYLCNV7J7DC57Z7E)

![image-20220719193427933](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-121?token=AOAPFCJAXZSYWLP33RVNAXTC572AA)

![image-20220719193620376](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-122?token=AOAPFCPIOOVLSJKVNL2OZJDC572AS)

![image-20220719193821108](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-123?token=AOAPFCJZ7MJGKJR5ENQFSLTC572BE)

## `MySQL`数据库设计规范

### 为什么要好好设计数据库?

![image-20220719194245343](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-124?token=AOAPFCIDZPYXLHXTKUKEC3TC572BU)

![image-20220719194256001](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-125?token=AOAPFCLS67D4VXIFC53BMKTC572CA)

### 数据库设计范式

#### 什么是范式?

> **范式即数据库设计应该满足的基本规则,其可以辅助我们建立起一个的数据库**
>
> **注意**:一般开发中最多遵守到`3NF-BCNF`并且有时为了保证效率还会连`3NF`都做不到

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-126?token=AOAPFCMZKCUSPKSNNRLNQUTC572CQ" alt="image-20220719194719426" style="zoom: 67%;" />

#### 前置知识

- `超键`
- `候选键`
- `主键`
- `外键`
- `主属性`
- `非主属性`

#### 第一范式`1NF`

> **数据库的每一个记录的每一个字段都必须满足原子性**
>
> **注意**:第一范式是必须遵守的

##### 例1

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-127?token=AOAPFCMUQY3CDPLLU3WIIHTC572DC" alt="image-20220719195355200" style="zoom: 60%;" />

##### 例2

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-128?token=AOAPFCMZOIXNOF4Y2GSKEPTC572DY" alt="image-20220719195448910" style="zoom: 50%;" />

##### 例3

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-130?token=AOAPFCLPB64TCZHQ2M23ON3C572EK" alt="image-20220719195525065" style="zoom: 50%;" />

#### 第二范式`2NF`

> ![image-20220719200232803](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-131?token=AOAPFCNP27SAE5MP4L5OIXDC572E4)

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-132?token=AOAPFCPNBAHOQVRAD2BTO4LC572FM" alt="image-20220719200013370" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-133?token=AOAPFCLQBGC6XXJFQJWGIXLC572GC" alt="image-20220719200045059" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-134?token=AOAPFCJQF6DVDBNKVAKHZZTC572GQ" alt="image-20220719200204625" style="zoom:50%;" />

#### 第三范式`3NF`

> ![image-20220719200311340](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-135?token=AOAPFCJ6E6PBUMUUTO3CRPTC572HI)

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-137?token=AOAPFCLTPGVIFFQIYWSRC5LC572II" alt="image-20220719200348822" style="zoom: 60%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/image-20220719200416490.png?token=AOAPFCJ4CH7XVUGTTO5XEETC572HY" alt="image-20220719200416490" style="zoom:60%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-138?token=AOAPFCIVYOUPK3TKW2Y5ZJTC572JQ" alt="image-20220719200442944" style="zoom:60%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-139?token=AOAPFCLYDBFYJEMJEYCUSXTC572J4" alt="image-20220719200457823" style="zoom:60%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-140?token=AOAPFCKXS4DVHJ5MHIXPYR3C572KM" alt="image-20220719200508983" style="zoom:60%;" />

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-141?token=AOAPFCJQ3YUDZBA3ELA6ONDC572LA" alt="image-20220719200553936" style="zoom:60%;" />

#### 巴斯科德范式`BCNF`

> 主属性（仓库名）对于候选键（管理员，物品名）是部分依赖的关系，这样就有可能导致异常情况。因此引入BCNF，**它在** **3NF** **的基础上消除了主属性对候选键的部分依赖或者传递依赖关系**。
>
> 如果在关系R中，U为主键，A属性是主键的一个属性，若存在A->Y，Y为主属性，则该关系不属于BCNF。

#### 第四范式`4NF`

> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-142?token=AOAPFCL45D5LI6ILTLDKHU3C572LQ" alt="image-20220719202320067" style="zoom:60%;" />

#### 第五范式`5NF`

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-143?token=AOAPFCNMMX343FXHNXA6XPLC572ME" alt="image-20220719202345225" style="zoom:60%;" />

#### **范式的优缺点**

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-1474?token=AOAPFCIB2ATWAKQISMTEUPLC572MS" alt="image-20220719200632496" style="zoom: 60%;" />

#### 反范式化

##### 概述

> **我们知道如果完全遵守范式,很可能带来数据库的性能低,那么我们在建立数据库时如何平衡规范与性能呢?**
>
> - 为满足某种商业目标 , 数据库性能比规范化数据库更重要
>
> - 在数据规范化的同时 , 要综合考虑数据库的性能
>
> - 通过在给定的表中添加额外的字段，以大量减少需要从中搜索信息所需的时间
>
> - 通过在给定的表中插入计算列，以方便查询
>

##### 如何反范式化地设计一个数据库?

##### 反范式导致的新问题

- 由于冗余信息的大量使用导致**数据库占用的存储空间变大了**
- 一个表中字段做了修改，另一个表中冗余的字段也需要做同步修改，否则`数据不一致`
- 若采用存储过程来支持数据的更新、删除等额外操作，如果更新频繁，会非常`消耗系统资源`
- 在`数据量小`的情况下，反范式不能体现性能的优势，可能还会让数据库的设计更加`复杂`

##### 反范式的适用场景

> 当冗余信息有价值或者能`大幅度提高查询效率`的时候，我们才会采取反范式的优化。
>

- **增加冗余字段的建议** 

  - 这个冗余字段`不需要经常进行修改`


  - 这个冗余字段`查询的时候不可或缺`


**2.** **历史快照、历史数据的需要**

在现实生活中，我们经常需要一些冗余信息，比如订单中的收货人信息，包括姓名、电话和地址等。每次发生的`订单收货信息`都属于`历史快照`，需要进行保存，但用户可以随时修改自己的信息，这时保存这些冗余信息是非常有必要的。

反范式优化也常用在`数据仓库`的设计中，因为数据仓库通常`存储历史数据`，对增删改的实时性要求不强，对历史数据的分析需求强。这时适当允许数据的冗余度，更方便进行数据分析。

![image_76](F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\07MySQL进阶.assets\image_76.png)

### `ER`模型

### 数据表的设计原则

> **注意**:这个原则**并不是绝对的**，有时候我们需要**牺牲数据的冗余度来换取数据处理的效率**。

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-145?token=AOAPFCOHPCGZPW2YIAU72M3C572NO" alt="image-20220719202610464" style="zoom:60%;" />

## `MySQL`数据库编写建议

##### **关于库**

1. 【强制】库的名称必须控制在32个字符以内，只能使用英文字母、数字和下划线，建议以英文字母开头。

2. 【强制】库名中英文`一律小写`，不同单词采用`下划线`分割。须见名知意。

3. 【强制】库的名称格式：业务系统名称_子系统名。

4. 【强制】库名禁止使用关键字（如type,order等）。

5. 【强制】创建数据库时必须`显式指定字符集`，并且字符集只能是utf8或者utf8mb4。创建数据库SQL举例：CREATE DATABASE crm_fund `DEFAULT CHARACTER SET 'utf8'`; 

6. 【建议】对于程序连接数据库账号，遵循`权限最小原则`。使用数据库账号只能在一个DB下使用，不准跨库。程序使用的账号`原则上不准有drop权限`。 

7. 【建议】临时库以`tmp_`为前缀，并以日期为后缀；备份库以`bak_`为前缀，并以日期为后缀。

#####  **关于表、列**

1. 【强制】表和列的名称必须控制在32个字符以内，表名只能使用英文字母、数字和下划线，建议以`英文字母开头`。 

2. 【强制】 `表名、列名一律小写`，不同单词采用下划线分割。须见名知意。

3. 【强制】表名要求有模块名强相关，同一模块的表名尽量使用`统一前缀`。比如：crm_fund_item 

4. 【强制】创建表时必须`显式指定字符集`为utf8或utf8mb4。 

5. 【强制】表名、列名禁止使用关键字（如type,order等）。

6. 【强制】创建表时必须`显式指定表存储引擎`类型。如无特殊需求，一律为InnoDB。 

7. 【强制】建表必须有comment。 

8. 【强制】字段命名应尽可能使用表达实际含义的英文单词或`缩写`。如：公司 ID，不要使用 corporation_id, 而用corp_id 即可。

9. 【强制】布尔值类型的字段命名为`is_描述`。如member表上表示是否为enabled的会员的字段命名为 is_enabled。 

10. 【强制】禁止在数据库中存储图片、文件等大的二进制数据。通常文件很大，短时间内造成数据量快速增长，数据库进行数据库读取时，通常会进行大量的随机IO操作，文件很大时，IO操作很耗时。通常存储于文件服务器，数据库只存储文件地址信息。

11. 【建议】建表时关于主键：`表必须有主键 `(1)强制要求主键为id，类型为int或bigint，且为auto_increment 建议使用unsigned无符号型。 (2)标识表里每一行主体的字段不要设为主键，建议设为其他字段如user_id，order_id等，并建立unique key索引。因为如果设为主键且主键值为随机插入，则会导致innodb内部页分裂和大量随机I/O，性能下降。

12. 【建议】核心表（如用户表）必须有行数据的`创建时间字段`（create_time）和`最后更新时间字段`（update_time），便于查问题。

13. 【建议】表中所有字段尽量都是`NOT NULL`属性，业务可以根据需要定义`DEFAULT值`。 因为使用NULL值会存在每一行都会占用额外存储空间、数据迁移容易出错、聚合函数计算结果偏差等问题。

14. 【建议】所有存储相同数据的`列名和列类型必须一致`（一般作为关联列，如果查询时关联列类型不一致会自动进行数据类型隐式转换，会造成列上的索引失效，导致查询效率降低）。

15. 【建议】中间表（或临时表）用于保留中间结果集，名称以`tmp_`开头。备份表用于备份或抓取源表快照，名称以`bak_`开头。中间表和备份表定期清理。

16. 【示范】一个较为规范的建表语句：

```mysql
CREATE TABLE user_info ( 
    `id` int unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键', 
    `user_id` bigint(11) NOT NULL COMMENT '用户id', 
    `username` varchar(45) NOT NULL COMMENT '真实姓名', 
    `email` varchar(30) NOT NULL COMMENT '用户邮箱', 
    `nickname` varchar(45) NOT NULL COMMENT '昵称', 
    `birthday` date NOT NULL COMMENT '生日', 
    `sex` tinyint(4) DEFAULT '0' COMMENT '性别', 
    `short_introduce` varchar(150) DEFAULT NULL COMMENT '一句话介绍自己，最多50个汉字', 
    `user_resume` varchar(300) NOT NULL COMMENT '用户提交的简历存放地址', 
    `user_register_ip` int NOT NULL COMMENT '用户注册时的源ip', 
    `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间', 
    `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间', 
    `user_review_status` tinyint NOT NULL COMMENT '用户资料审核状态，1为通过，2为审核中，3为未 通过，4为还未提交审核',
    PRIMARY KEY (`id`), 
    UNIQUE KEY `uniq_user_id` (`user_id`), 
    KEY `idx_username`(`username`), 
    KEY `idx_create_time_status`(`create_time`,`user_review_status`) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='网站用户基本信息'
```

17. 【建议】创建表时，可以使用可视化工具。这样可以确保表、字段相关的约定都能设置上。实际上，我们通常很少自己写 DDL 语句，可以使用一些可视化工具来创建和操作数据库和数据表。可视化工具除了方便，还能直接帮我们将数据库的结构定义转化成 SQL 语言，方便数据库和数据表结构的导出和导入。

#####  **关于索引**

1. 【强制】InnoDB表必须主键为id int/bigint auto_increment，且主键值`禁止被更新`。 

2. 【强制】InnoDB和MyISAM存储引擎表，索引类型必须为`BTREE`。 

3. 【建议】主键的名称以`pk_`开头，唯一键以`uni_`或`uk_`开头，普通索引以`idx_`开头，一律使用小写格式，以字段的名称或缩写作为后缀。

4. 【建议】多单词组成的columnname，取前几个单词首字母，加末单词组成column_name。如: sample 表 member_id 上的索引：idx_sample_mid。 

5. 【建议】单个表上的索引个数`不能超过6个`。 

6. 【建议】在建立索引时，多考虑建立`联合索引`，并把区分度最高的字段放在最前面。

7. 【建议】在多表 JOIN 的SQL里，保证被驱动表的连接列上有索引，这样JOIN 执行效率最高。

8. 【建议】建表或加索引时，保证表里互相不存在`冗余索引`。 比如：如果表里已经存在key(a,b)， 则key(a)为冗余索引，需要删除。

##### **SQL编写**

1. 【强制】程序端SELECT语句必须指定具体字段名称，禁止写成 *。 

2. 【建议】程序端insert语句指定具体字段名称，不要写成INSERT INTO t1 VALUES(…)。 

3. 【建议】除静态表或小表（100行以内），DML语句必须有WHERE条件，且使用索引查找。

4. 【建议】INSERT INTO…VALUES(XX),(XX),(XX).. 这里XX的值不要超过5000个。 值过多虽然上线很快，但会引起主从同步延迟。

5. 【建议】SELECT语句不要使用UNION，推荐使用UNION ALL，并且UNION子句个数限制在5个以内。

6. 【建议】线上环境，多表 JOIN 不要超过5个表。

7. 【建议】减少使用ORDER BY，和业务沟通能不排序就不排序，或将排序放到程序端去做。ORDER BY、GROUP BY、DISTINCT 这些语句较为耗费CPU，数据库的CPU资源是极其宝贵的。

8. 【建议】包含了ORDER BY、GROUP BY、DISTINCT 这些查询的语句，WHERE 条件过滤出来的结果集请保持在1000行以内，否则SQL会很慢。

9. 【建议】对单表的多次alter操作必须合并为一次。对于超过100W行的大表进行alter table，必须经过DBA审核，并在业务低峰期执行，多个alter需整合在一起。 因为alter table会产生`表锁`，期间阻塞对于该表的所有写入，对于业务可能会产生极大影响。

10. 【建议】批量操作数据时，需要控制事务处理间隔时间，进行必要的sleep。 

11. 【建议】事务里包含SQL不超过5个。因为过长的事务会导致锁数据较久，MySQL内部缓存、连接消耗过多等问题。

12. 【建议】事务里更新语句尽量基于主键或UNIQUE KEY，如UPDATE… WHERE id=XX;否则会产生间隙锁，内部扩大锁定范围，导致系统性能下降，产生死锁。

## `MySQL`调优拓展(对数据库参数进行调优)

## `MySQL`事务

### 事务的概念

- 事务是一些`SQL`语句的集合,事务将他们组合成一个整体.`MySQL`在对事务进行运行时,会保证事务的基础特性不被改变.
- 每一个事务为一个工作单位,如果某个事务的某个语句执行失败或者发生错误,那么`MySQL`会将我们数据库中的数据`ROLLBACK`到该事务执行之前的状态.事务中的语句只有全部正确执行完毕,事务才算执行完毕.只要存在一个语句执行错误都会导致我们的事务回滚

### 事务的基本特性(`ACID`)

- **`Atomicity`原子性**
  - 即事务中的所有语句要么全部顺利执行,要么事务执行失败,`MySQL`触发回滚
- **`Consistent`一致性**
  - 一致性是指事务执行前后，数据从一个`合法性状态`变换到另外一个`合法性状态`。这种状态是`语义上`的而不是语法上的，跟具体的业务有关。
- **`Isolation`隔离性**
  - 即并发执行的事务之间不能相互干扰,必须保证相对独立.也即事务与事务之间是隔离的
- **`durability`持久性**
  - 即只要事务执行完成,在不使用回滚的情况下,数据库中的数据不会以任何方式对执行完成后的数据产生影响.**如事务执行完成后,即便服务器立刻宕机,事务对于数据库的修改都不会丢失**
  - **注意**:持久性是通过`事务日志`来保证的。日志包括了`重做日志`和`回滚日志`。当我们通过事务对数据进行修改的时候，首先会将数据库的变化信息记录到重做日志中，然后再对数据库中对应的行进行修改。这样做的好处是，即使数据库系统崩溃，数据库重启后也能找到没有更新到数据库系统中的重做日志，重新执行，从而使事务具有持久性。 

### 事务的五大状态

- **`active`活动的**
  - 组成事务的**`SQL`语句正在依次执行但还没有全部执行完**
- **`partially committed`部分提交的**
  - 组成事务的**`SQL`语句已经全部执行完成,但是对于数据表的修改还没有刷新到磁盘上**
- **`failed`失败的**
  - 当事务处在`活动的`或者`部分提交的`状态时，可能遇到了某些错误（数据库自身的错误、操作系统错误或者直接断电等）而无法继续执行，或者人为的停止当前事务的执行，**但是数据库还没有恢复到事务执行前的状态,那么我们就说该事务处在`失败的`状态**。
- **`aborted`中止的**  
  - 如果事务执行了一部分而变为`失败的`状态，那么就需要**把已经修改的事务中的操作还原到事务执行前**的状态。换句话说，就是要撤销失败事务对当前数据库造成的影响。我们把这个撤销的过程称之为`回滚`。当`回滚`操作执行完毕时，也就是**数据库恢复到了执行事务之前的状态，我们就说该事务处在了`中止的`状态**。
- **`committed`提交的**
  - 当一个处在`部分提交的`状态的事务将**修改过的数据都`同步到磁盘`上之后**，我们就**可以说该事务处在了`提交的`状态**。

### 事务的使用

> **注意**:
>
> - 当开启一个事务之后事务中的所有`SQL`语句都会自动关闭自动提交功能,我们只有通过自己使用`COMMIT`关键字

#### 显式的使用

##### 方式1

- **`START TANSACTION`开启一个事务**

- **`READ ONLY|READ WRITE|WITH CONSISTENT SNAPSHOT`可选项,用于指定事务的类型**

  - `READ ONLY`:只读事务
  - `READ WRITE`:读写事务
  - `WITH CONSISTENT SNAPSHOT`:一致性读

- **一些`SQL`语句,一般以`DML`数据操作`SQL`语句为主**

- **提交事务或回滚事务**

  - **提交事务**

    ```mysql
    COMMIT
    ```

  - **回滚事务**

    ```mysql
    #回滚到事务执行之前
    ROLLBACK
    #回滚到指定的事务保存点
    ROLLBACK TO 保存点名称
    ```

##### 方式2

> - **使用`BEGIN`来开始一个事务**
>
> - **这样的开启事务的方式与`START TRANSACTION`的区别在于不能使用指定事务类型的可选项**

#### 保存点的创建与删除

> **在一个事务中的任意位置我们都可以创建保存点或删除保存点**

- **创建保存点**

  ```mysql
  SAVEPOINT 创建的保存点的名称
  ```

- **删除保存点**

  ```mysql
  RELEASE SAVEPOINT 保存点名称
  ```

#### 隐式的事务

> **当我们设置`autocommit`属性为`on`时有些`SQL`语句就会自动在执行完毕后执行`COMMIT`操作,若设置为`off`那么所有具有自动提交机制的`SQL`语句便不会再自动提交**

- **`DDL`数据定义语言是带有自动提交的**
- **操作`mysql`数据库中的表时我们用于操作的`SQL`语句会进行自动`COMMIT`**
- **当我们开启了一个事务,但该事务还没有使用`COMMIT`或`ROLLBACK`之前又开启另外一个事务,那么我们前一个事务会自动在下一个事务开启之前执行`COMMIT`操作**
- **当我们使用`SET`关键字来将`autocommit`的值设置为`on`时,会先自动执行一个`COMMIT`再执行我们的修改操作**
- **当我们使用`LOCK TABLE|UNLOCK TABLE`语句时会在该语句执行之前自动执行一个`COMMIT`**

### 常见并发问题

- **`Dirty Read`脏读**
  - 若存在这样的两个事务`A,B`,事务`A`对指定数据表的某一条记录的某一个字段进行了修改,而事务`B`在事务``A``修改了之后,但还未提交前读取了事务`A`修改了的字段,然后事务``A``发生了回滚,此时事务`B`就读取了一个错误的信息,此时说这种情况为**脏读**
- **`Dirty Write`脏写**
  - 若存在这样的两个事务`A,B`,事务`A`对指定数据表的某一条记录的某一个字段进行了修改,而事务`B`在事务``A``修改了之后,但还未提交前修改了事务`A`修改了的字段,那么说这种情况为**脏写**
- **`Non-repeatable Read`不可重读**
  - 若存在这样的两个事务`A,B`,事务``A``对数据表的某个记录的某个字段进行了读取(此时事务``A``并没有终止),然后事务`B`对事务`A`读取的数据进行了修改,然后当事务`A`再次读取原本的数据时就会发现两次读取到的数据不一致,这就是不可重读的问题
- **`Phantom`幻读**
  - 若存在这样的两个事务`A,B`,事务`A`读取了指定数据表中的一些记录(在事务`A`未提交的情况下),事务`B`向我们的数据表中添加了一些记录,然后当事务`A`再用同样的`SQL`语句进行数据表读取时,读取出的记录数相比于前一次增加了,这种情况被称之为**幻读**
- **其他**
  - 事务``B``执行时剔除了一些指定数据表中符合`studentno > 0`的记录而不是插入新记录，那么事务``A``之后再根据`studentno > 0`的条件读取的`记录变少了`,这是一种其他的情况

### 事务的隔离级别

#### 四大隔离级别

- `READ UNCOMMITTED`：**读未提交**，在该隔离级别，所有事务都可以读取到其他未提交事务的执行结果。不能避免脏读、不可重复读、幻读。
- `READ COMMITTED`：**读已提交**，它满足了隔离的简单定义：**一个事务只能读取到已经提交的事务对数据表所做的改变**。这**是大多数数据库系统的默认隔离级别（但不是MySQL默认的）**。可以避免脏读，但不可重复读、幻读问题仍然存在。
- `REPEATABLE READ`：**可重复读**，事务A在读到一条数据之后，此时**事务B对该数据进行了修改并提交**，那么事务A再读该数据，读到的还是原来的内容。可以避免脏读、不可重复读，但**幻读问题仍然存在**。`这是MySQL的默认隔离级别`。
- `SERIALIZABLE`：**可串行化**，确保事务可以从一个表中读取相同的行。在**这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作**。所有的并发问题都可以避免，但性能十分低下。能避免脏读、不可重复读和幻读。

#### 隔离级别的设置

```mysql
SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVEL 隔离级别; 
#其中，隔离级别格式： 
> READ UNCOMMITTED 
> READ COMMITTED 
> REPEATABLE READ 
> SERIALIZABLE
```

## `MySQL`事务日志

> **事务一般具有`ACID`的特性,而`isolation`隔离性一般通过`MySQL锁`实现,而`ACD`三大特性则需要借助`Redo Log`与`Undo Log`来保证**
>
> - `Redo Log`:**重做日志**.其会详细地保存所有的对我们的数据表进行修改的操作
> - `Undo Log`:**回滚日志**

****

### `Redo Log`

> **`Redo Log`设计的目的**:
>
> - 虽然我们的数据缓冲池能够加快我们`SQL`语句的执行速度,但是由于其修改操作也会先存储在内存上,并在一定时机刷盘到磁盘上,这种情况下,如果数据未刷入磁盘之前服务器宕机了,那么如果没有良好的保障措施就会导致我们的修改操作全部丢失.
> - 由于上面的原因也会导致事务的持久性无法生效
>
> **`Redo Log`的解决方案**
>
> - `Redo Log`会详细记录下我们对数据表进行的所有修改操作,因此当我们的数据表修改发生了丢失时,就可以查找我们的`Redo Log`来找回这些丢失的修改
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-146?token=AOAPFCJKGTOIWZGGKXQ64BLC572OK" alt="image-20220730165149076" style="zoom:58%;" />

#### `Redo Log`的优点与特性

- **优点**
  - **可以在保证数据表修改丢失的发生频率低的同时尽可能减少刷盘频率**
  - **`Redo Log`占用的空间小**
- **特点**
  - **`Redo Log`是在磁盘中顺序连续存储的**
  - **`MySQL`运行过程中`Redo Log`会自动运行并自动维护我们的操作**

#### `Redo Log`的组成

- **`Redo Log Buffer`**
  - **重做日志缓冲区**,其中会保存我们需要写入到我们的磁盘的日志记录.
  - 我们可以通过`innodb_log_buffer_size`系统变量来设置这个缓冲区的大小
- **`Redo Log File`**
  - **重做日志文件**,存储在我们的磁盘上的重做日志文件.

****

#### `Redo Log`的流程

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-147?token=AOAPFCPPRKRAAIZOUBDCS2DC572PE" alt="image-20220730165304980" style="zoom:55%;" />

- :one:在`Redo Log Buffer`中生成一条`Redo Log`记录,记录下我们修改的数据以及修改的目标

- :two:当我们需要对某个数据表进行修改时,我们的`MySQL`服务就会将要修改的记录读取到我们的内存中来,然后进行修改.
- :three:修改成功后我们的`MySQL`服务器就会在`Redo Log Buffer`中生成一条`Redo Log`记录,记录下我们修改的数据以及修改的目标
- :four:当我们的修改操作的事务调用`COMMIT`时,我们的`Redo Log Buffer`中的记录就会被刷盘到我们的磁盘中的`Redo Log File`
- :five:`MySQL`服务会定期地将我们内存中对数据进行的修改刷盘到我们的磁盘中

****

#### `Redo Log`的数据缓冲刷盘策略

> **我们可以通过`innodb_flush_log_at_trx_commit`来指定我们的`innodb`存储引擎的`Redo Log`重做日志来进行刷盘策略控制**
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-148?token=AOAPFCONJ52IBUIWIIUNBMTC572PS" alt="image-20220730165350027" style="zoom:58%;" />

- **`0`**
  - 当取值为该值时,我们的`Redo Log Buffer`不再会在每一次`COMMIT`指定被调用时刷入磁盘上的`Redo Log File`.而是由我们的`MySQL`主服务进程来控制刷盘时间
- **`1`**
  - 当取值为该值时,我们的`Redo Log Buffer`会在每一次`COMMIT`指令被调用的时候刷盘到磁盘上的`Redo Log File`
- **`2`**
  - 当取值为该值的时,我们的`Redo Log Buffer`会在每一次`COMMIT`指令被调用时,将我们的`Redo Log Buffer`中的内容转移到我们的操作系统的`文件缓冲区中`,然后由我们的操作系统来决定什么时候将这些内容进行刷盘

****

#### 补充知识

##### **`Mini-Transaction`**

> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-149?token=AOAPFCMLZ2TOFHP6WIWTIETC572QE" alt="image-20220730165712314" style="zoom:50%;" />

- **简称`mtr`每一个`mtr`记录了对于某一数据表的数据的一组修改操作**

##### **`Redo Log`写入`Redo Log Buffer`**

- ![image-20220730165924568](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-150?token=AOAPFCIGH2DIANV3VBDD42LC572Q4)

##### **事务并行时的`Redo Log`的`Redo Log Buffer`写入**

- ![image-20220730170113990](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-151?token=AOAPFCN47EGSV3ZXI7M2XVDC572RS)

##### **`Redo Log Buffer`中的`Redo Log Block`的结构分析**

- ![image-20220730170250175](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-152?token=AOAPFCMWHFZFX5XDOWVOLELC572SC)
- ![image-20220730170301254](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-153?token=AOAPFCLMJGMYHNA7G477TCDC572SW)

****

#### `Redo Log Buffer`和`Redo Log File`相关的系统变量

- **`innodb_log_buffer_size`:用于指定`Redo Log`的内存缓冲区的大小**
- **`innodb_flush_log_at_trx_commit`:用于指定我们的`Redo Log Buffer`中的数据的刷盘策略**
- **`innodb_log_group_home_dir`:指定`Redo Log File`文件的存储路径，默认值为`./`即我们`MySQL`数据库的数据目录**
- **`innodb_log_files_in_group`:明`Redo Log File`的个数，命名方式如:``ib_logfile0，ib_logfile1...ib_logfilen``.默认2个，最大100个。**
- **`innodb_log_file_size`:给单个`Redo Log File`文件设置大小,默认值为``48MB`` .最大值为``512GB``,注意最大值指的是整个`Redo Log File`系列文件之和，即（`innodb_log_files_in_group * innodb_log_file_size `）不能大于最大值`512GB`。**

****

#### 日志文件组与`Write Position`,`CheckPoint`

##### 日志文件组

> - 我们应该清楚,我们的`Redo Log File`并不仅仅只是由一个文件组成的,而是由多个子日志文件组成.
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-154?token=AOAPFCJKA62SE277WBYSCH3C572TG" alt="image-20220730175346748" style="zoom:50%;" />
>
> **问题**
>
> - 由于我们的日志文件组的大小是限定的,因此当我们写入的记录过多,以致于填满了我们的`Redo Log File`时就会导致我们需要考虑对`Redo Log File`中的一些内容做删除或覆盖操作.为了解决该问题,我们的`MySQL`就引入了`Write Positon`和`Check Point`机制

##### `Write Position`与`CheckPoint`

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-155?token=AOAPFCMICCLZQCJLAXP6RYLC572T2" alt="image-20220730174625536" style="zoom:50%;" /><img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/image-20220730174649692.png?token=AOAPFCIJU57ILHJMXXSW6PDC572U2" alt="image-20220730174649692" style="zoom:50%;" />

- :one:**`Write Position`与``CheckPoint``分别是一个指针,其指向我们的`Redo Log File`的某个位置的物理存储位置.**
- :two:**`CheckPoint`指向的位置之前的数据修改记录`Redo Log`日志都是已经被成功刷入磁盘,持久化了的,而之后的记录则是还没有被刷入我们的磁盘的数据修改记录的`Redo Log`日志**
- :three:**`Write Position`指向的位置就是当我们需要从`Redo Log Buffer`中向我们的`Redo Log File`写入数据修改记录的`Redo log`日志时的起始位置**
- **:four:由上面两条我们可以知道,`CheckPoint`之前的数据都是可以被覆盖的,而之后的都是不能被覆盖的,因此就会有如下情况**
  - 当我们的`Write Position`还没有追上我们的`CheckPoint`时那么就可以直接进行写入,因为`Write Position`指向的位置之后的一段空间是可以被覆写的
  - 当我们的`Write Position`与`CheckPoint`指向的位置相同时,此时我们的日志写入操作就不能在进行,因为此时进行会导致我们的暂时不能被覆盖的数据被覆盖.因此此时我们的写入操作必须暂时阻塞,等待内存中的数据修改刷盘到我们的磁盘上持久化,`CheckPoint`后移之后才能解除阻塞,继续插入

****

### `Undo Log`

> **`Undo Log`的设计目的**
>
> - `Undo Log`是用于**保证我们的事务的原子性**而存在的一个日志.
> - 我们的`Undo Log`**必须要实时地记载我们的`SQL`语句对于我们数据进行的修改操作**,当我们的事务进行失败时,我们需要可以通过`Undo Log`将我们的数据恢复到我们的事务执行之前的情况
>
> **`Undo Log`的作用**
>
> - 事务回滚的支持
> - `MVCC`

#### `Undo Log`的组成结构

> `InnoDB`对``Undo Log`的管理采用**段**的方式，也就是`回滚段（Rollback Segment）`。每个回滚段记录了`1024`个`Undo Log Segment`，而在每个`Undo Log Segment`段中进行`undo页`的申请。

##### 回滚段与事务的关系

- 同一时刻**一个`Undo Log Segment`只支持被一个事务使用**,但**一个事务可以同一时刻可以拥有多个`Undo Log Segment`**

- 同一时刻**每个事务**只会使用**一个回滚段**,而**一个回滚段**在同一时刻可能会**服务于多个事务**。
- 当一个**事务开始**的时候，会**制定一个回滚段**，在事务进行的过程中，当**数据被修改时**，**原始的数据会被复制到回滚段**。
- 在回滚段中，事务由于对数据进行修改,会不断填充`Undo Log Segment`盘区，直到事务结束或所有的空间被用完。如果当前的**盘区不够用**，事务会**在段中请求扩展下一个`Undo Log Segment`盘区**，如果**所有已分配的盘区都被用完**，事务会**覆盖最初的盘区**或者**在回滚段允许的情况下扩展新的盘区**来使用。
- 回滚段存在于`Undo`表空间中，在**数据库中可以存在多个undo表空间**，但**同一时刻只能使用一个undo表空间**。
- 当事务提交时，``InnoDB``存储引擎会做以下两件事情：
- - 将`Undo Log`放入列表中，以供之后的purge操作
  - 判断`Undo Log`所在的页是否可以重用，若可以分配给下个事务使用

##### 回滚段中存储的数据的类别

- `Uncommitted Undo Information`未提交的回滚数据
- `Committed Undo Information`:已经提交并且还没有过期的回滚数据
- `Expired Undo Information`已经提交并且过期了的回滚数据

****

### `Undo Log`与`Redo Log`的区别

- `Undo Log`的设计初衷是为了保证我们**事务的原子性**,而`Redo Log`是为了保证我们的**事务的持久性**
- `Redo Log`日志只有在我们的事务顺利地成功执行后才会被刷入磁盘保存,虽然在事务执行过程中我们的`Redo Log`也会实时记录,但是如果我们的事务执行失败,那么先前记载的记录是不会被刷入磁盘中而是会被丢弃的.而我们的`Undo Log`不同,`Undo Log`同样也必须实时地记载我们的事务对于数据的修改,但是即便我们的事务处理中断`Undo Log`中记录的数据也必须刷盘到磁盘中,而且必须保证任何情况下事务中断,数据都必须已经被刷入到磁盘中
- `Redo Log`记录的是我们数据进行修改操作时,修改后的数据情况.而我们的`Undo Log`记载的则是我们事务修改了的数据的原始数据.

#### `Undo Log`的分类

- `Insert Undo Log`
- `Update Undo Log`

****

### `Undo Log`与`Redo Log`存在时我们的`MySQL`数据``内存-磁盘``的同步情况

#### 没有使用这两个日志时

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-160?token=AOAPFCKHSO6TH444M7OUTDLC572X2" alt="image-20220730191730286" style="zoom:60%;" />

- 首先我们客户端进程发起数据修改相关的操作
- 然后我们需要修改的记录读取到我们的`Buffer Pool`中,并对其进行修改
- 然后我们便进行其他的操作,`MySQL`服务器进程会在合适的时候将我们存在于`Buffer Pool`中的数据刷入我们的磁盘

**问题**

- **持久性问题**:如果在`Buffer Pool`中的数据还没有刷入磁盘时,我们的服务器发生了严重的问题,我们的内存直接掉电,内存中的数据直接丢失了,我们的数据就永远不可能找回
- **原子性问题**:如果我们的部分对数据的修改操作需要撤回,在当前的这种情况下我们是完全无法得知哪些数据是我们修改了的,因此根本没有办法实现修改操作的撤回.

#### 使用了这两个日志时

![image-20220730191756243](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-161?token=AOAPFCJMCOECP4UY2MNEM6DC572YK)

- 首先我们的`MySQL`客户端向`MySQL`服务器发起对于数据进行修改的`SQL`语句申请
- 然后我们的`MySQL`服务器就会将我们需要修改的数据记录中没有事先保存在我们的`Buffer Pool`中的数据记录读取到我们的`Buffer Pool`中
- 然后我们的`Undo Log`会根据我们的数据修改申请将我们内存中的待会需要被修改的记录字段的原始数据内容以及基本信息保存起来,并刷入我们磁盘上的`Undo Log File`中以实现持久化
- 然后我们的`MySQL`服务器就会开始对我们`Buffer Pool`中的数据进行更新
- 然后我们的`Redo Log`就会根据我们的数据修改申请在我们的`Buffer Pool`中找到我们修改后的记录字段的新内容将其连同一些基本信息一起存入我们的`Redo Log Buffer`
- 然后我们的`MySQL`系统就会按照我们的`innodb_flush_log_at_trx_commit`指定的`Redo Log`刷盘策略对我们`Redo Log Buffer`中的数据进行刷盘以实现持久化

### `MySQL`出现问题时基于`Redo Log`的重建与基于`Undo Log`的回滚的基本流程

****

## `	MySQL`锁机制

### `MySQL`数据资源的使用情况分析

#### `读-读`并发

> **即多个事务同时对同一个数据表(甚至于同一条记录,同一个字段)进行读操作**

- **在这样的情况下我们稍微思考就能知道我们的并发读操作正常情况下是没有造成由于并发导致的绝大部分数据问题的可能性的**

#### `读-写`并发

> **即有两个事务同时进行,一个事务对我们的数据表(甚至是细分到某一条记录,某一个字段)进行读操作,而另一个事务则对同一个表(甚至是同一个记录,同一个字段)进行写操作**

- 在这样的情况下我们就会知道,由于`读-写`并行,有可能导致`不可重读,幻读,脏读`等问题

#### `写-写`并发

> **即两个事务同时进行,两个事务分别对同一个数据表(甚至于同一个记录,同一个字段)进行写操作**

- 在这样的情况下,由于`写-写`并行很可能会发生`脏写`等问题

****

### 并发问题的解决

#### 解决方案一

> - **读操作通过`MVCC`方案实现**
> - **写操作通过锁机制实现**

- **普通的``SELECT``语句在``READCOMMITTED``和``REPEATABLEREAD``隔离级别下会使用到``MVCC``读取记录**。
  - 在``READCOMMITTED``隔离级别下，一个事务在执行过程中每次执行``SELECT``操作时都会生成一个``ReadView``，`ReadView`的存在本身就保证了事务不可以读取到未提交的事务所做的更改，也就是避免了脏读现象；
  - 在``REPEATABLEREAD``隔离级别下，一个事务在执行过程中只有第一次执行``SELECT``操作才会生成一个``ReadView``，之后的``SELECT``操作都复用这个``ReadView``，这样也就避免了不可重复读和幻读的问题。

#### 解决方案二

> - **读写操作都通过锁机制实现**

#### 方案对比

- **解决方案一相对于解决方案二能够表现出在并发的场景下的更高的效率**
- **但是有些情况下我们还是有采用解决方案二的必要的**

****

### `MySQL`常见的**17**种锁

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-162?token=AOAPFCMGYUHNF5HCQ5VAMB3C572ZE" alt="第15章_锁_Page4_Image1" style="zoom: 67%;" />

#### 读写锁

> **读锁**：也称为**共享锁**、英文用`S`表示。针对同一份数据，多个事务的**读操作可以同时进行**而不会互相影响，相互不阻塞的。**但是涉及到写操作的事务就会被阻塞**
> **写锁**：也称为**排他锁**、英文用`X`表示。当前写操作没有完成前，它会阻断其他写锁和读锁。这样就能确保**在给定的时间里，只有一个事务能执行写入，并防止其他用户读取正在写入的同一资源**。
>
> **注意:在`Innodb`引擎下读写锁可以以数据表为粒度进行阻塞也可以以数据记录为粒度进行阻塞**

****

#### 表级锁

##### :one:`S`锁与`X`锁

> 在对某个表执行`SELECT、INSERT、DELETE、UPDATE`语句时，`InnoDB`存储引擎是不会为这个表添加表级别的`S锁`或者`X锁`的。在**对某个表执行一些诸如`ALTER TABLE、DROP TABLE`这类的`DDL`语句时，其他事务对这个表并发执行诸如`SELECT、INSERT、DELETE、UPDATE`的语句会发生阻塞**。同理，**某个事务中对某个表执行`SELECT、INSERT、DELETE、UPDATE`语句时，在其他会话中对这个表执行`DDL`语句也会发生阻塞**。这个过程其实是通过在`server层`使用一种称之为`元数据锁`（英文名：`MetadataLocks`，简称`MDL`）结构来实现的。
>
> 一般情况下，不会使用`InnoDB`存储引擎提供的表级别的`S锁`和`X锁`。只会在一些特殊情况下，比方说**崩溃恢复**过程中用到。比如，在系统变量`autocommit=0，innodb_table_locks=1`时，手动获取`InnoDB`存储引擎提供的`表t`的`S锁`或者`X锁`可以这么写：
>
> - `LOCK TABLES t READ`：`InnoDB`存储引擎会对`表t`加表级别的`S锁`。
>
> - `LOCK TABLES t WRITE`：`InnoDB`存储引擎会对`表t`加表级别的`X锁`。
>
> **不过尽量避免在使用`InnoDB`存储引擎的表上使用`LOCK TABLES`这样的手动锁表**语句，它们并不会提供什么额外的保护，只是会降低并发能力而已。**`InnoDB`的厉害之处还是实现了更细粒度的行锁**.

| 锁类型       | 自己可读 | 自己可写 | 自己可操作其他表 | 他人可读 | 他人可写 |
| ------------ | -------- | -------- | ---------------- | -------- | -------- |
| 读锁 ``S锁`` | 是       | 否       | 否               | 是       | 否，等   |
| 写锁 `X锁`   | 是       | 是       | 否               | 否，等   | 否，等   |

##### :two:意向锁

> - **意向锁是一种`表级锁`**
> - ==**如果一个事务想要获取某个数据表的表级别或行级别读写锁,那么其必须先获取到该表的表级别意向读写锁**==
> - **意向锁与意向锁之间无论是读写锁都不会相互排斥**
> - **除了``IS``与``S``兼容外， 意向锁会与表级共享锁(读锁) / 排他锁(写锁) 互斥**
> - ==**`IX`，`IS`是表级锁，不会和行级的X，S锁发生冲突。只会和表级的X，S发生冲突。**==
>
> **注意事项**:意向锁是由**存储引擎自己维护的** ，**用户无法手动操作意向锁**，在为数据行加共享 / 排他锁之前，
> `InooDB `会先获取该数据行,所在数据表的对应意向锁  

- **意向读锁(意向共享锁,IS锁)**:即告诉`MySQL`服务进程我们**当前事务有意向对某个数据表,某个数据记录添加`读锁(共享锁,S锁)`**
- **意向写锁(意向排他锁,IX锁)**:即告诉`MySQL`服务进程我们**当前事务有意向对某个数据表,某个数据记录添加`写锁(排他锁,X锁)`**

###### 意向锁的特点

-  `InnoDB`支持**多粒度锁**，**特定场景下，行级锁可以与表级锁共存**。
- ==**意向锁之间互不排斥**，但除了``IS``与``S``兼容外， **意向锁会与 共享锁 / 排他锁 互斥 。**==
  - 当某个表存在**读锁S**,那么此时显然不能允许其他事务获取该表的写锁**,从而也就不能允许获取意向写锁`IX`**
  - 当某个表存在**写锁X**,那么此时显然不能允许其他事务获取该表的读写锁,**从而也就不能允许获取意向读写锁**
  - 当某个表存在**读锁S**,那么此时显然能允许其他事务获取该表的读锁,**从而也就能允许获取意向读锁IS**
- ==**`IX`，`IS`是表级锁，不会和行级的X，S锁发生冲突。只会和表级的X，S发生冲突。**==
- 意向锁在保证并发性的前提下，实现了 行锁和表锁共存 且满足事务隔离性的要求。  

###### **意向锁的作用**

- 由于一个事务想要获取某个数据表的表级别或行级别读写锁,那么其必须先获取到该表的表级别意向读写锁,因此**当我们有事务想要对我们的当前表添加表级别的读写锁时,我们的`MySQL`服务进程就可以通过查看该表上是否有意向读写锁,来判断该表是否存在表级别或行级别读写锁.**
- 这样就可以让我们的`MySQL`服务进程**不用去依次查看我们数据表整体以及该数据表的每一行是否具有读写锁,提高了效率**

##### :three:自增锁

> **对于具备`AUTO_INCREMENT`自增列的数据表而言，有如下三种数据记录的插入模式**
>
> - **`Simple Inserts`**:即当语句被初始处理时可以**预先确定好插入的数据记录的条数**的`SQL`语句
> - **`Bulk Inserts`**:即在语句被初始处理时不可以**预先缺点好插入的数据记录的条数**的`SQL`语句,如涉及子查询作为插入数据记录的`SQL`语句
> - **`Mixed-Mode Inserts`**:即我们插入的数据记录中有些指定了自增列的值,而有些数据记录不指定自增列的值的插入方式
>
> **为什么要有自增锁?**
>
> **注意事项**
>
> - **同一时刻一个数据表只能有一个表级`AUTO-INC`锁.因此多个事务获取`AUTO_INC`锁时会发生争用**

###### 自增锁的三大模式

> **通过系统变量`innodb_autonic_lock_mode`来进行指定**

- **`0`**:**普通锁定模式**
  - **无论是`Simple Inserts`还是`Bulk Inserts`还是`Mixed-mode Inserts`都会在进行之前向`MySQL`服务进程申请表级`AUTO-INC`锁,并且会保持到语句结束**
  - 这种模式下可以保证在`BinLog`中重放的时候，``master``与`slave`中数据的`AUTO_INCREMENT`是相同的。
- **`1`**:**连续锁模式**
  - 在`MySQL 8.0`之前，连续锁定模式是**默认**的
  - **同一时刻只有一个事务的语句可以持有`AUTO-INC锁`**
  - **`Bulk Inserts`**仍然使用`AUTO-INC表级锁`，并**保持到语句结束**。
  - **`Simple Inserts`**则通过在``mutex(轻量锁)``的控制下获得所需数量的自动递增值来避免`表级AUTO-INC锁`的长时间占有， 其只会在`mutex`锁的分配过程下保持`AUTO-INC`锁，而**不是直到语句完成**
  - **无论是`Simple Inserts`还是`Bulk Inserts`都需要实现获取到表级`AUTO-INC`锁**
- **`2`**:**交错锁模式**
  - `MySQL 8.0`开始，交错锁模式是**默认**设置。  
  - 在此锁定模式下，`AUTO INCREMENT`保证在所有并发执行的所有类型的`insert`语句中**自增列的自动生成的值是唯一且单调递增的**。
  - 由于**存在并发时多个事务下的记录插入操作交替进行**,因此任何给定语句插入的行生成的值可能不是连续的  
    - 如两个事务都对一个表进行总共10条记录的`Simple Inserts`插入,那么由于上面的原因可能会导致虽然插入的数据记录的最大自增列值为10,最小为1,但是我们可能发现第一个事务的所有记录在数据表上的自增列的值为`1 3 5 7`而不是`1 2 3`的情况

##### :four:元数据锁

> `MySQL5.5`引入`Meta Data Lock`，简称`MDL`锁，属于**表锁**
>
> **`MDL`的作用是，保证读写的正确性**
>
> **注意事项**
>
> - 对一个表做**增删改查操作**的时候，**必须事先**给数据表加`MDL读锁`
> - 对表做**结构变更操作**的时候，**必须事先**给数据表加`MDL写锁` 
> - **`MDL读锁`与`MDL读锁`不相互争用,`MDL读锁`与`MDL写锁`相互争用,`MDL写锁`与`MDL写锁`相互争用**

- 如果一个查询语句正在**遍历一个表中的数据**，而执行期间另一个线程**对这个表结构做变更,增加了一列**，那么查询线程拿到的结果跟表结构对不上，肯定是不行的。

****

#### 行级锁

##### :one:记录锁

> - **记录锁的粒度为数据记录**
>
> - **记录锁分为`S锁,读锁`,`X锁,写锁`两种**
> - **``记录读锁与记录读锁``之间相互不争用,`记录读锁与记录写锁`相互争用,`记录写锁与记录写锁`相互争用**

- **记录级`S锁,读锁`**
- **记录级`X锁,写锁`**

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-163?token=AOAPFCOLKNDF3VZFKU24Z43C5722E" alt="image-20220731140455917" style="zoom:67%;" />

##### :two:间隙锁

> **一般用于使用了`AUTO INCREMENT`自增列的数据表.**
>
> **设计背景**
>
> - 我们知道`幻读`是由于一个事务对我们的数据表进行读取后,其他的事务又在我们的数据表中进行插入操作导致的,因此我们的`MySQL`就设计出了`间隙锁`来**一定程度上解决`幻读问题`**.
>
> **间隙锁的作用为使得当前数据表中的某两条记录之间在锁定期间不允许被添加新的记录**

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-164?token=AOAPFCJ3OTTGDMKHHPR2YMDC5722W" alt="image-20220731145445967" style="zoom: 67%;" />

图中`id`值为**`8`**的记录加了`gap`锁，**意味着不允许别的事务在`id`值为`8`的记录前边的间隙插入新记录** ，其实就是
`id`列的值$(3, 8)$这个区间的**新记录是不允许立即插入**的。比如，有另外一个事务再想插入一条`id`值为`4`的新记录，它定位到该条新记录的下一条记录的`id`值为`8`，而这条记录上又有一个`gap`锁，所以就会阻塞插入操作，直到有这个`gap`锁的事务提交了之后，`id`列的值在区间$(3, 8)$中的新记录才可以被插入。  

##### :three:临键锁

- **若我们既想`给某条记录加锁,`又想`阻止其他事务在该记录前面的与上一条记录之间的间隙添加新的记录`,那么我们就可以使用临键锁**

##### :four:插入意向锁

> - **`插入意向锁`类似于`意向锁`是一种表明我们事务意图的锁**
> - **在对数据表添加`间隙锁`或`临键锁`之前,我们必须事先获取到其对应的`插入意向锁`**
> - **插入意向锁之间不相互排斥,但`插入意向锁`与`间隙锁`或`临键锁`之间是相互排斥的**
>   - 如果某个位置具备`插入意向锁`那么就不能添加`间隙锁`或`临键锁`
>   - 如果一个位置有`间隙锁`或`临键锁`,那么就不能添加`插入意向锁`
>
> **插入意向锁的作用**
>
> - 当我们想要对数据表进行数据插入时,一般情况下我们会需要查看该数据表在我们要添加数据的位置是否存在`间隙锁`或`临键锁`.而这样直接查看的方式是非常低效的,因此`MySQL`设计出`插入意向锁`,**我们在插入之前只需要查看是否有对应的插入意向锁就能够知道我们要插入数据的位置是否有`间隙锁`或`临键锁`**

****

#### 页级锁

> **页锁**就是在``页的粒度 ``上进行锁定，**锁定的数据资源比行锁要多**，因为一个页中可以有多个行记录。当我
> 们使用页锁的时候，会**出现数据浪费的现象**，但这样的浪费最多也就是一个页上的数据行。**页锁的开销**
> **介于表锁和行锁之间，会出现死锁。锁定粒度介于表锁和行锁之间，并发度一般。**
>
> **每个层级的锁数量是有限制的**，因为锁会占用内存空间，`` 锁空间的大小是有限的 ``。当某个层级的锁数量
> 超过了这个层级的阈值时，就会进行**锁升级**。**锁升级就是用更大粒度的锁替代多个更小粒度的锁**，比如
> `InnoDB`中行锁升级为表锁，这样做的好处是占用的锁空间降低了，**但同时数据的并发度也下降了**。

****

#### 乐观锁与悲观锁

> **前提**:**乐观锁与悲观锁并不是显示存在的锁,它们只是一种锁的设计理念**

##### 乐观锁

> - 乐观锁认为**对同一数据的并发操作不会总发生，属于小概率事件，不用每次都对数据上锁**，但是在**更新的时候会判断一下在此期间别人有没有去更新这个数据** **，也就是**不采用数据库自身的锁机制**，而是**通过**程序来实现**。在程序上，我们可以采用`版本号机制`或者`CAS机制`实现。乐观锁适用于多读的应用类型，这样。在`Java`中`java.util.concurrent.atomic`包下的原子变量类就是使用了乐观锁的一种实现方式：`CAS`实现的。  
>
> - **乐观锁就是程序员自己控制数据并发操作的权限，基本是通过给数据行增加一个戳（版本号或
>   者时间戳），从而证明当前拿到的数据是否最新。 ** 
>
> **应用场景**

- **版本号机制**
  - 在表中设计一个 版本字段`version `第一次读的时候，会获取`version`字段的取值。然后对数据进行更
    新或删除操作时，会执行`UPDATE ... SET version=version+1 WHERE version=version`。此时
    如果已经有事务对这条数据进行了更改，修改就不会成功  
- **时间戳机制**
  - 时间戳和版本号机制一样，也是在更新提交的时候，将当前数据的时间戳和更新之前取得的时间戳进行
    比较，如果两者一致则更新成功，否则就是版本冲突。  

##### 悲观锁

> 悲观锁**总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上**
> **锁，这样别人想拿这个数据就会`阻塞`直到它拿到锁**（**共享资源每次只给一个线程使用，其它线程阻塞，**
> **用完后再把资源转让给其它线程**）。比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁，当
> 其他线程想要访问数据时，都需要阻塞挂起。`Java`中 `synchronized `和 `ReentrantLock `等独占锁就是
> 悲观锁思想的实现。  

****

#### 显式锁与隐式锁

##### 显示锁

> **通过特定的语句进行加锁，我们一般称之为显示加锁**  

##### 隐式锁

###### 情景1

> **特点**:**一个事务在操作数据时,并不会为自己创建锁,而是按照情况**
>
> - `情况1`:要操作的数据的`trx_id`字段指明的事务不在活动状态下,**此时可以直接操作数据,不用加锁**
> - `情况2`:要操作的数据的`trx_id`字段指明的事务在活动状态下,**此时不可以直接操作数据,并且要为该事务创建锁,为自己创建`waiting=true`的锁**

> -  `InnoDB`的**每条记录中都一个隐含的`trx_id`字段**，这个字段存在于聚簇索引的`B+`树中。
> - 在操作一条记录前，首先根据记录中的`trx_id`检查该事务**是否是活动的事务(未提交或回滚)**。**如果是活动的事务,且该活动的事务没有对其操作的记录进行加锁,那么我们就为该事务添加锁**
> - 然后我们检查如果为我们当前的记录加锁是否会发生**锁冲突**，**如果有冲突，创建锁，并设置锁为waiting状态,并进入下一步**。如果**没有冲突不加锁，跳到最后一步**。
> - 一直等待知道**加锁成功进入下一步**(**如果超时或被取消则`不再`进入下一步**)
> - **将自己的`trx_id`写入`trx_id`字段**然后**执行**我们需要的**数据修改操作**。  

###### 情景2

> - 对于**二级索引记录**来说，本身并没有`trx_id`隐藏列，但是在二级索引页面的`Page Header`部分有一个`PAGE_MAX_TRX_ID`属性，**该属性代表对该页面做改动的最大的事务id** ，**如果`PAGE_MAX_TRX_ID`属性值`小于当前最小的活跃事务id` ，那么说明对该页面做修改的事务都已经提交了，此时就说明没有其他的事务在操作我们的当前页面,因此我们就可以直接操作该页面**,**否则**就需要在页面中定位到对应的二级索引记录，然后回表**找到它对应的聚簇索引记录，然后再重复情景一的做法**。  

****

#### 全局锁

> **概念**
>
> - 全局锁就是**对整个数据库实例加锁**。当你需要**让整个库处于`只读状态`**的时候，可以使用这个命令，之后其他线程的以下语句会被阻塞：**数据更新语句（数据的增删改）、数据定义语句（包括建表、修改表结构等）和更新类事务的提交语句**。
>
> **应用场景**
>
> - 全库逻辑备份。

****

#### 死锁

> **概念**
>
> - 死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方占用的资源，从而导致恶性循环
>
> **解决死锁问题的两种途径**
>
> - **直接进入等待**，直到超时。这个超时时间可以通过参数`innodb_lock_wait_timeout`来设置
> - 发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务（将持有最少行级
>   排他锁的事务进行回滚），让其他事务得以继续执行。将参数`innodb_deadlock_detect`设置为`on`，表示开启这个逻辑

****

### 锁操作相关指令

****

### 锁的内存结构剖析

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-165?token=AOAPFCOMGKMKZW4Z7E3ZWELC5723S" alt="image-20220731164743212" style="zoom: 50%;" />

- `锁所在的事务信息`:**一个指针**,通过指针可以找到内存中关于拥有该锁的事务的更多信息

- `索引信息`:**一个指针**,记录加锁的记录属于哪个索引

- **表锁特有**

  - `表信息`:记载着是对哪个表加的锁，还有其他的一些信息

- **行锁特有**

  - `Space ID`:锁定的记录所在的**表空间**
  - `Page Number`:锁定的记录所在的**页号**
  - `n_bits`:对于**行锁**来说，**一条记录就对应着一个比特位**，一个页面中包含很多记录，**用不同的比特位来区分到底是哪一条记录加了锁**。为此在行锁结构的末尾放置了一堆比特位，这个`n_bits`属性**代表使用了多少比特位**。
    - `n_bits`的值一般都比页面中记录条数多一些。主要是**为了之后在页面中插入了新记录后也不至于重新分配锁结构**

- `type_mode`:一个**32位的数**，被分成了`lock_mode`、`lock_type`和`rec_lock_type`三个部分

  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-166?token=AOAPFCIFKZZL6OVVXD4BSCLC5724C" alt="image-20220731165521878" style="zoom:50%;" />
  - **`lock_mode`用于指明锁的模式**
    - `LOCK_IS`（十进制的`0`）：也就是当**该区域所有比特位置为`0`时生效**,表示**意向共享锁**，也就是`IS锁` 。
    - `LOCK_IX`（十进制的`1`）：也就是当**倒数第`1`个比特位置为`1`时生效**,表示**意向排他锁**，也就是`IX锁` 。
    - `LOCK_S`（十进制的`2`）：也就是当**倒数第`2`个比特位置为`1`时生效**,表示**共享锁**，也就是`S锁` 。
    - `LOCK_X`（十进制的``3``）：也就是当**倒数第`3`个比特位置为`1`时生效**,表示**排他锁**，也就是`X锁` 。
    - `LOCK_AUTO_INC`（十进制的`4`）：也就是当**倒数第`4`个比特位置为`1`时生效**,表示**`AUTO-INC锁`** 。  
  - **`lock_type`用于指明锁的类型**
    - `Lock_Table`(十进制的`16`),也就是当**倒数第`5`个比特位置为`1`时生效**,**表示表级锁**  
    - `Lock_Rec`(十进制的`32`),也就是当**倒数第`6`个比特位置为1时生效**,**表示行级锁**  
  - **`rec_lock_type`用于指明行锁的具体类型,只有在`lock_type`的值为`LOCK_REC`时生效**
    - `LOCK_ORDINARY`（十进制的`0`）：也就是当**当前区域所有比特位置为`0`时生效**表示**`next-key`临键锁** 。
    - `LOCK_GAP`（十进制的`512`）：也就是当**倒数第`10`个比特位置为`1`时生效**，**表示`gap`间隙锁** 。
    - `LOCK_REC_NOT_GAP`（十进制的`1024`）：也就是当**倒数第`11`个比特位置为`1`时生效**，**表示记录锁** 。
    - `LOCK_INSERT_INTENTION`（十进制的`2048`）：也就是当**倒数第`12`个比特位置为`1`时生效**，**表示插入意向锁**
    - `…`
  - **`is_waiting`指明该锁是否在等待加锁**
    - `LOCK_WAIT`（十进制的`256`） ：当**倒数第`9`个比特位置为`1`时，表示`is_waiting `为`true `，也就是当前**事务尚**未获取到锁**，处在等待状态；当这个比特位为`1`时，表示`is_waiting`为
      `false `，也就是当前事务**获取锁成功**。  

- `其他信息`

  - 为了更好的管理系统运行过程中生成的各种锁结构而设计了各种哈希表和链表  

- `一堆比特位`

  > 

  - 如果是` 行锁结构 `的话，在该结构**末尾**还放置了**一堆比特位**，**比特位的数量由上边提到的` n_bits `属性**
    **表示**。`InnoDB`数据页中的每条记录在` 记录头信息 `中都包含一个` heap_no `属性，伪记录` Infimum `的
    `heap_no `值为` 0 `，` Supremum `的` heap_no `值为` 1 `，之后每插入一条记录，` heap_no `值就`增1`。` 锁结
    构 `**最后的一堆比特位就对应着一个页面中的记录，一个比特位映射一个`heap_no` ，即一个比特位映射到页内的一条记录。  **

****

### 锁监控

#### 注意事项

- **每个层级的锁数量是有限制的**，因为锁会占用内存空间，`` 锁空间的大小是有限的 ``。当某个层级的锁数量
  超过了这个层级的阈值时，就会进行**锁升级**。**锁升级就是用更大粒度的锁替代多个更小粒度的锁**，比如
  `InnoDB`中行锁升级为表锁，这样做的好处是占用的锁空间降低了，**但同时数据的并发度也下降了**。

#### 借助`STATUS`参数监控锁

- `Innodb_row_lock_current_waits `:当前`MySQL`服务进程``is_waiting=true``的锁的个数
- `Innodb_row_lock_time`:从系统启动到现在锁定总时间长度  
- `Innodb_row_lock_time_avg`:从系统启动到现在,所有进入过等待状态的锁等待时间的平均值
- `Innodb_row_lock_time_max`:从系统启动到现在,所有进入过等待状态的锁中等待时长最长的锁的具体等待时长
- `Innodb_row_lock_waits`:从系统启动到现在进入过等待状态的锁的总个数  

#### 借助`information_schema`库监控锁

- **`INNODB_TRX`表**
- **`INNODB_LOCKS`表**
  - `MySQL 8.0`后被`Performance_schema`库下的``data_locks``表取代
- **`INNODB_LOCK_WAITS`表**
  - `MySQL 8.0`后被`Performance_schema`库下的``data_lock_waits``表取代
- **注意**:
  - `MySQL5.7及之前`可以通过`information_schema.INNODB_LOCKS`查看事务的锁情况，但**只能看到阻塞事务的锁**；如果**事务并未被阻塞**，则在该表中**看不到该事务的锁情况**。 
  - `MySQL8.0`删除了`information_schema.INNODB_LOCKS`，添加了` performance_schema.data_locks `，可以通过`performance_schema.data_locks`查看事务的锁情况，和`MySQL5.7`及之前不同，`performance_schema.data_locks`**不但可以看到阻塞该事务的锁，还可以看到该事务所持有的锁**。
  - 同时，`information_schema.INNODB_LOCK_WAITS`也被` performance_schema.data_lock_waits `所代替。 

****

## `MySQL`多版本并发控制

> 多版本并发控制(`Multiversion Concurrency Control`),简称为`MVCC`.`MVCC `是**通过数据行的多个版**
> **本管理来实现数据库的` 并发控制`** 。这项技术使得在`InnoDB`的事务隔离级别下执行` 一致性读 `操作有了保
> 证。换言之，就是**当我们查询一些正在被另一个事务更新的行的时候，在`MVCC`作用下我们可以看到它们被更新之前的值，这样在做查询的时候就不用等待另一个事务释放锁**。  

### 快照读与当前读

#### 快照读

> **快照读又叫一致性读**，读取的是**快照数据**。不加锁的简单的` SELECT `都属于快照读，即**不加锁的非阻塞读**

- **快照读在读取数据之前并不会对我们的数据添加锁,因此快照读期间,其他的事务可以同时对被快照读的数据表,数据记录做修改**
- 快照读使得我们可以**在某个字段在被其他事务修改且事务并未提交的情况下**,让我们的当前事务**读取该字段的值时读取到的是其未被修改前的值**
- **快照读的前提是`隔离级别不是串行级别`，串行级别下的快照读会`退化成当前读`**  

#### 当前读

> 当前读就是一种正常的数据读取方式,在**读取数据之前必须对读取操作涉及到的数据表或数据行加上`读锁,共享锁`使得在我们的当前事务读取期间,读取的内容不能被其他事务所修改**

### `MVCC`的实现原理剖析

> **`MVCC`的实现主要依赖于`隐藏字段,Undo Log,Read View`三个设计**

#### 隐藏字段与`Undo Log`

##### **`trx_id`**

> **即我们在`隐式锁`中谈到的那个`trx_id`**

##### **`roll_pointer`**

> **一个指针**,我们知道每当我们对数据表的某条记录进行修改时,就会事先将该记录的原始数据存储到我们的`Undo Log`中.而我们的**`roll pointer`就是一个指针,其指向我们当前数据记录的原始数据在`Undo Log`中的位置**
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-167?token=AOAPFCIX3MUQVTEZ7LJCM5DC57246" alt="image-20220731221144901" style="zoom: 67%;" />

- **注意**
  - 每次对记录进行改动，都会记录一条`undo日志`，**每条`undo日志`也都有一个` roll_pointer属性`**  
  - `INSERT `操作对应的`undo日志`**没有该属性，因为该记录并没有更早的版本**  
  - **`Undo Log`中每一条日志的`roll pointer`的作用在于充当链表的`next pointer`的作用的指针,将我们一条一条的`Undo Log`日志组织成一个链表**
    - ==**值得注意的是,这个组织成的链表中的每一条`Undo Log`日志都是记录同一个数据表下同一个数据记录的变化的**==
    - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-168?token=AOAPFCPA7RHJG5R6L3WM44LC5725M" alt="image-20220731221119777" style="zoom: 30%;" />
  - 对该记录每次更新后,都会将旧值放到一条` undo日志 `中，就算**是该记录的一个旧版本**，随着更新次数的增多，**该记录的所有的版本都会被` roll_pointer `属性连接成一个链表**，我们把**这个链表称之为 `版本链`** ，版本链的**头节点就是当前记录最新的值**  

#### ==`Read View`==

##### 不同隔离级别下数据读取的要求

- **``READ UNCOMMITTED  ``**:**读未提交**,在这种情况下直接读取即可,无需借助读锁或`MVCC`辅助
- **`SERIALIZABLE`**:**串行读**,`InnoDB`规定使用加锁的方式来访问记录。  
- **`READ COMMITTED`与`REPEATABLE READ`**:只能读**已经提交了**的事务修改过的记录。假如另一个事务已经修改了记录但是尚未提交，那么不能直接读取最新版本的记录,而**应该根据情况判断`Undo Log`中该记录的版本链上哪一个记录是我们当前事务可以读取的.**  

##### `Read View`的组成

> **由上面我们知道,`Read View`的存在就是为了保证我们的事务读取到的记录是该事务进行时已经提交了的数据.**

- **`creator_trx_id`**
  - 指明创建这个`Read View`的事务的`ID`
    - 只有设计对数据修改的事务才会具有实际的`ID`,而对于只读事务而言,其事务`ID`恒为`0`
- **`trx_ids`**
  - 存储着创建该`Read View`时`MySQL`系统中所有处于活跃状态的事务的`ID`
- **`up_limit_id`**
  - `trx_ids`中存储的所有事务`ID`中最小的`ID`的值
- **`low_limit_id`**
  - 生成`ReadView`时系统中应该分配给下一个事务的` ID `值。`low_limit_id `是系统**最大的事务`ID`值**，这里要注意是系统中的事务`ID`，需要**区别于正在活跃的事务`ID  `**
  - 其并不是`trx_ids`中存储的所有事务`ID`中最大的`ID`的值,实际上我们的`MySQL`服务有事务`ID`计数器,每给一个设计数据修改的事务分配了事务`ID`我们的事务`ID`计数器就会加`1`,**因此这个`low_limit_id`指的就是创建`ReadView`时我们事务`ID`计数器的值**

##### `Read View`的规则

> **在访问某条记录时，只需要按照下边的步骤判断记录的某个版本是否可见。** 

- 如果被访问版本的`trx_id`属性值与`ReadView`中的` creator_trx_id `值**相同**，意味着**当前事务在访问它自己修改过的记录，所以该版本`可以`被当前事务访问**。
- 如果被访问版本的`trx_id`属性值**小于**`ReadView`中的` up_limit_id `值，表明**生成该版本的事务在当前事务生成`ReadView`前已经提交，所以该版本`可以`被当前事务访问**。
- 如果被访问版本的`trx_id`属性值**大于或等于**`ReadView`中的` low_limit_id `值，表明**生成该版本的事务在当前事务生成`ReadView`后才开启，所以该版本`不可以`被当前事务访问**。
- 如果被访问版本的`trx_id`属性值在`ReadView`的` up_limit_id `和` low_limit_id `**之间**，那就**需要判断一下`trx_id`属性值是不是在` trx_ids `列表中**。
  - 如果**在**，说明创建`ReadView`时**生成该版本的事务还是活跃的，该版本`不可以`被访问**。
  - 如果**不在**，说明创建`ReadView`时**生成该版本的事务已经被提交，该版本`可以`被访问**。   

### `MVCC`实现的整体流程

- :one:`MySQL`客户端向我们的`MySQL`服务端发起事务`SQL`请求
- :two:`MySQL`服务端进程为该事务分配**事务`ID`**,也即版本号
- :three:`MySQL`服务端进程为我们的事务建立起其对于的`Read View`
- :four:查询我们当前数据表中的数据,然后用数据的`trx_id`隐藏字段与我们的`Read View`进行对比
  - 如果该`trx_id`的取值符合我们上述`Read View`中可以直接读取的方式的规则,那么我们的该事务就可以**顺利使用我们的数据**.
  - 如果不符合规则,那么就需要进入下一步
- :five:此时我们就需要根据我们记录的`roll_pointer`指针获取到记录在`Undo Log`中的对应位置
- :six:然后对该记录的版本链上的数据记录的`trx_id`隐藏字段与我们的当前事务的`Read View`进行对比.直到找到一条符合`Read view`规则的数据记录
- :seven:然后就可以将我们找到的符合了`Read View`规则的数据读取出来使用了

#### **注意事项**

- 在隔离级别为**读已提交**（`Read Committed`）时，**一个事务中**的**每一次` SELECT `查询**都会**重新获取**一次
  `Read View` ,**即便前后两次查询语句完全相同** 
- 当隔离级别为**可重复读**(`Repeatable Read `)的时候,一个事务**只在第一次` SELECT `的时候**会获取一次` Read View`,而**后面所有的` SELECT `都会复用这个` Read View`**  

**问题**:由于重新获取`Read View`因此每一次获取到的`Read View`是不一定相同的,因此就可能导致**不可重读或幻读**现象的产生

##### 举例

## `MySQL`常用日志

### 六大常用日志

> **除`二进制日志`外，其他日志都是` 文本文件 `。默认情况下，所有日志创建于` MySQL数据目录 `中**  

- **慢查询日志**：记录所有执行时间超过`long_query_time`的**`SQL`查询语句**，方便我们对查询进行优化。  
- **通用查询日志**：记录**所有连接的起始时间和终止时间**，以及**连接发送给数据库服务器的所有指令**，
  对我们复原操作的实际场景、发现问题，甚至是对数据库操作的审计都有很大的帮助。  
- **错误日志**：记录`MySQL`服务的**启动、运行或停止`MySQL`服务时出现的问题**，方便我们了解服务器的
  状态，从而对服务器进行维护。  
- **二进制日志**：记录**所有更改数据的语句**，可以用于主从服务器之间的数据同步，以及服务器遇到故
  障时数据的无损失恢复  
- **中继日志**：**用于主从服务器架构**中，**从服务器**用来**存放主服务器二进制日志内容的一个中间文件**。
  从服务器通过读取中继日志的内容，来同步主服务器上的操作  
- **数据定义语句日志**：记录**数据定义语句执行的元数据操作**  

### 通用查询日志(`General Query Log`)

> 记录**所有连接的起始时间和终止时间**，以及**连接发送给数据库服务器的所有指令**，对我们复原操作的实际场景、发现问题，甚至是对数据库操作的审计都有很大的帮助。  

#### 日志状态管理

> 相关系统变量:`general_log`,`general_log_file`

- **开启与关闭**

  ```mysql
  #临时开启
  set @@global.general_log = on;
  #临时关闭
  set @@global.general_log = off;
  ```

  ```shell
  #永久开启
  [mysqld]
  general_log = on
  #永久关闭
  [mysqld]
  general_log = off
  ```

- **日志文件位置获取**

  ```mysql
  SHOW VARIABLES LIKE "general_log_file"
  ```

#### 日志查看

> **通用查询日志的存储格式为文本文件,因此我们可以直接按照文本文件的读取方式查看通用查询日志**

#### 日志刷新与删除

> 如果数据的使用非常频繁，那么**通用查询日志会占用服务器非常大的磁盘空间**。数据管理员**可以删除很**
> **长时间之前的查询日志**，以**保证`MySQL`服务器上的硬盘空间**。  

- **`方式1`**

  - **到通用查询日志文件目录下手动删除我们的通用查询日志文件**

- **`方式2`**

  - **在操作系统终端中运行以下指令重新生成通用查询日志文件**

    ```shell
    mysqladmin -uroot -p flush-logs
    ```

### 错误日志(`Error Log`)

> 记录`MySQL`服务的**启动、运行或停止`MySQL`服务时出现的问题**，方便我们了解服务器的状态，从而对服务器进行维护。

#### 日志状态管理

- **`Error Log`是默认开启且无法被关闭的**

- **日志文件存储位置**

  ```mysql
  SHOW VARIABLES LIKE "log_err"
  ```

- **文件存储位置修改**

  ```shell
  [mysqld]
  log_err= 路径\日志文件名.err
  ```

#### 日志查看

> **错误日志文件在`MySQL 8.0`下以`.err`格式存储,但是我们可以`通过直接使用记事本打开`来查看其内容**

#### 日志刷新与删除

- **首先直接手动删除其在文件系统中的文件**

- **然后按照如下指令重新生成一个空的错误日志文件**

  ```shell
  install -omysql -gmysql -m0644 /dev/null /var/log/mysqld.log
  ```

### 二进制日志(`BinLog`)

> `binlog`即`binary log`，二进制日志文件，也叫作**变更日志（`update log`）**。它记录了数据库所有执行的`DDL `和` DML `等**数据库更新事件的语句**，但是**不包含没有修改任何数据的语句**（如数据查询语句`select、
> show`等）。  
>
> **应用场景**
>
> - **数据恢复**  
> - **数据复制**
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-169?token=AOAPFCJOQSJSJRMESD33CWDC5726E" alt="image-20220801151356081" style="zoom: 60%;" />

#### 日志状态管理

- **日志的开启与关闭**

  ```mysql
  #临时设置
  	#开启
  SET @@global.log_bin = on;
  	#关闭
  SET @@global.log_bin = off;
  
  #永久设置
  	#开启
  [mysqld]
  log_bin = on
  	#关闭
  [mysqld]
  log_bin = off
  ```

- **日志文件存储位置管理**

  ```mysql
  [mysqld]
  log_bin_basename= "路径/文件名前缀"
  log_bin_index= "路径/文件名前缀.index"
  ```

#### 日志查看

> **`MySQL`创建二进制日志文件时有如下流程**
>
> - 按照`log_bin_index`指定的路径与文件名创建一个`.index`文件
> - 然后再按照`log_bin_index`指定的路径与文件名建立第一个数据文件(**以`.000001`为后缀**)
>   - `MySQL`服务**每重新启动一次**，以`.000001`为后缀的文件就会**增加一个**，并且**其后缀名按`1`递增**  

- **查看当前所有现存的`二进制日志数据文件`以及其大小**

  ```mysql
  SHOW BINARY LOGS;
  ```

- **操作系统终端查看指定`二进制日志数据文件`的内容**

  ```shell
  #复杂的呈现方式
  mysqlbinlog -v "<数据文件路径>"
  #更简洁的呈现方式
  mysqlbinlog -v --base64-output=DECODE-ROWS "<数据文件路径>"
  
  
  # 可查看参数帮助
  mysqlbinlog --no-defaults --help
  # 查看最后100行
  mysqlbinlog --no-defaults --base64-output=decode-rows -vv atguigu-bin.000002 |tail
  -100
  # 根据position查找
  mysqlbinlog --no-defaults --base64-output=decode-rows -vv atguigu-bin.000002 |grep -A
  20 '4939002'
  ```

- **`MySQL`下使用`SQL`语句查看二进制日志数据文件的内容**

  ```mysql
  show binlog events [IN <'log_name'>] [FROM <pos>] [LIMIT [offset,] row_count];
  ```

  - **`log_name`**
    - 要查看的**二进制数据文件的名称与后缀(**==**不用给出其存储路径**==**)**,不指定就默认为**第一个数据文件**
  - **`pos`**
    - 指定查看的`SQL`语句的**起始`Position`**,,不指定就默认数据文件的**第一个`Position`**
  - **`offset`**
    - **偏移量**,,不指定就**默认为`0`**
  - **`row_count`**
    - **查询的总条数**,不指定就默认为**全部**

#### 日志刷新与删除

- **删除指定二进制日志数据文件**

  ```mysql
  # 删除指定文件名的
  PURGE {MASTER | BINARY} LOGS TO "指定日志文件名";
  
  # 删除在指定日期之前创建的
  PURGE {MASTER | BINARY} LOGS BEFORE ‘指定日期’;
  
  #删除所有文件
  RESET MASTER;
  ```

#### 借助二进制日志文件进行数据恢复

- **基本语法**

  > **注意**:使用`mysqlbinlog`命令进行恢复操作时，**必须是编号小的先恢复**，例如`atguigu-bin.000001`**必**
  > **须**在`atguigu-bin.000002`**之前恢复**  

  ```shell
  mysqlbinlog [option] <用于恢复的二进制日志数据文件名>|mysql –u <用户名> -p <密码>;
  ```

  - **`option`**
    - **`--start-date 和 --stop-date`**:指定**恢复数据库**的**起始时间点**和**结束时间点**  
      - **日期的格式为`YYYY-MM-DD hh:mm:ss`**
    - **`--start-position和--stop-position`**:指定**恢复数据**的**开始位置**和**结束位置**  
      - `Position`的值为`int`

### 二进制日志`Bin Log`工作流程

- **事务执行完成且`COMMIT`成功后我们的`MySQL`服务器会对该事务涉及到的每一条数据修改的语句生成`Bin Log`记录**
- **然后`MySQL`服务器会将该`Bin Log`记录保存到我们内存上的`Bin Log Cache`中**
  - 因为一个事务的`Bin Log`不能被拆开，无论这个事务多大，也要确保一次性写入，所以系统会给每个事务所属的线程分配一个块内存作为`Bin log Cache`。  
- **然后`Bin Log Cache`中关于该事务的`Bin Log`记录就会按照我们指定的规则被保存到我们的磁盘上**
- <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-170?token=AOAPFCPKG2RXAADRP5CEQ6LC57264" alt="image-20220801223025948" style="zoom:60%;" />
- <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-171?token=AOAPFCPDEY26QT2QM6HPBI3C5727K" alt="image-20220801211554713" style="zoom: 60%;" />

### `Bin Log`,`Redo Log`,`Undo Log`

> **三者的不同**
>
> - **`Bin  Log`与`Undo Log`为逻辑日志**,其记录的内容**不是在**磁盘上的某个数据页上对某一条记录进行修改这样的**物理位置上的数据修改**,而是记录类似于**给某个表的`ID=2`的这一行的` c `字段加` 1`**这样的逻辑内容
> - **`Redo Log`**则为物理日志,其记录的内容**直接就是在**磁盘上的**某个数据页上对某一条记录进行修改**这样的**物理位置上的数据修改**
> - **`Undo Log`与`Redo Log`都是实时写入**的,`Undo Log`是**每一次数据修改都会要写入一次**磁盘的`Undo Log`文件,而`Redo Log`是**每一次数据修改都会要写入一次**`Redo log Buffer`.而我们的**`Bin Log`则会要等到我们的整个事务进行完成并`COMMIT`后**才生成该事务的`Bin Log`记录**然后写入**`Bin Log Buffer`

### 基于`两阶段设计模式`的`Redo Log`加`Bin Log`的数据恢复

> - 由于`Bin Log`要在我们的事务提交完成后**进行`COMMIT`后才会生成该事务的`Bin Log`记录并写入`Bin Log Cache`**.因此如果在我们`MySQL`服务器运行过程中我们的事务刚刚进行完``COMMIT``操作我们的`MySQL`服务端进程**发生错误中断了**,那么显然此时我们的这个事务的**`Bin Log`记录肯定无法完成,但是很显然此时的`Redo Log`是完整的写完了的,只不过处在`prepare`阶段**.
>   - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-172?token=AOAPFCOPSXQLWI7ZAXBYODLC573AA" alt="image-20220801223448222" style="zoom:60%;" />
> - **此时就会导致一个问题,我们的`Redo Log`中有相应的记录,而本来应该保存了所有的数据修改语句的`Bin Log`中却少了这一些记录.而无论是我们的`主从复制`,`数据恢复`,`数据复制备份`都是利用`Bin Log`来进行的.显然在这样的情况下,我们备份得到的数据是相对于我们原数据少了一些修改操作的,因此我们必须借助`Redo Log`才能完成`主从复制`,`数据恢复`,`数据复制备份`**
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-173?token=AOAPFCJCN4LEHVHPUDY34U3C573BM" alt="image-20220801223056531" style="zoom:67%;" />MySQL进阶-4

##### 注意

> **在`COMMIT`阶段是先调用`COMMIT`指令,然后写入`Bin Log`,然后才将`Redo Log`设置为`COMMITTED`状态**

- 当在事务的**`COMMIT`阶段之前服务进程就出错**了,那么此时认为是**事务的失败**,因此会**回滚数据**
- 当在事务的`COMMIT`阶段的**`Bin Log`写入过程中出错**,那么此时**不认为是事务的失败**,因此我们**可以直接通过`Prepare`状态的`Redo Log`来进行数据的恢复**
- 当在事务的`COMMIT`阶段的**`Redo Log`的`COMMITTED`状态设置写入过程中出错**,那么此时**不认为是事务的失败**,因此我们**可以直接通过`Bin Log`来进行数据的恢复**

- ![image-20220801223506641](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-175?token=AOAPFCLMSQ775GJK77MLFZ3C573CO)
- ![image-20220801223520040](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-176?token=AOAPFCOQNX7WZ23A7YY6PBLC573DA)

### 中继日志(`Relay Log`)

> - **中继日志只在主从服务器架构的从服务器上存在**  
> - **从服务器为了与主服务器保持一致**，要从主服务器**读取二进制日志`Bin Log`的内容**，并且把读取到的信息写入` 本地的日志文件 `中，**而这个从服务器本地的日志文件就叫中继日志`Relay Log`**
> - 从服务器**通过读取中继日志**的内容，来**同步主服务器上的操作**,这一个过程又被称之为**数据同步**
>
> **注意事项**
>
> - 搭建好主从服务器之后，**中继日志默认会保存在从服务器的数据目录**下  
> - 文件名的格式是
>   - **数据文件**:`从服务器名-relay-bin.序号` 
>   - **索引文件**:`从服务器名 -relay-bin.index`  

#### 日志状态管理

> **基本同`Bin Log`**

#### 日志查看

> **基本同`Bin Log`**

#### 日志刷新与删除

> **基本同`Bin Log`**

### 注意事项

> 如果从服务器宕机，有的时候**为了系统恢复，要重装操作系统**，这样就可能会**导致你的` 服务器名称 `与之前不同** 。而**中继日志文件名**里是**` 包含从服务器名 `**的。在这种情况下，就**可能导致你恢复从服务器的时候，无法**
> **从宕机前的中继日志文件里读取数据**，以为是日志文件损坏了，其实**是名称不对**了。**解决的方法也很简单，把从服务器的名称改回之前的名称**  

## 主从复制

> **设计背景**
>
> - 一般应用对数据库而言都是`读多写少`，也就说**对数据库读取数据的压力比较大**.那么一个解决这一问题的思路就是采用`数据库集群`的方案，做` 主从架构 `、进行` 读写分离`.**使得我们的主服务进程的压力减少**
>
> **优化方式优先级(从高到低)**
>
> - `优化SQL语句和数据表索引`
> - 采用` 缓存的策略`
> - 采用` 主从架构 `，进行`读写分离`  
>
> <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-177?token=AOAPFCPHVQ57HAMDID2HLJLC573DW" alt="image-20220802133744352" style="zoom:60%;" />

### 主从复制的作用解读

- ** 	**

  - 即通过**设置多个子服务进程**的方式(子服务进程的数据库与主服务进程的数据库是通过`Bin Log`进行同步的),**使得原本大量提交给主服务进程的`SQL`请求**中的一**大部分由子服务进程进行处理**,从而**减轻我们主服务进程的压力**.
  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-178?token=AOAPFCPWKOGUT4QWUN6ELXDC573EE" alt="image-20220802134036650" style="zoom: 60%;" />

- **`数据备份`**

- **`提高可用性`**

### 从服务进程数据同步的流程

> **整个流程涉及`两个进程`,`三个线程`**
>
> - **主服务进程**与**从服务进程**
> - **一个主服务线程**与**两个从服务线程**

- :one:首先假设当前时刻我们的主服务进程与从服务进程的数据**是同步的**
- :two:然后我们的主服务进程由于**有客户端发起数据修改**事务,对**主服务数据进行了修改**.并且事务`COMMIT`阶段完成了,其对应的**`Bin Log`也生成了**.
- :three:然后我们的主服务进程就会通过其下属的`Log Dump`线程向我们从服务进程的`IO`线程发出数据发送请求,当`IO`线程接收了该发送请求后,我们的**`Log Dump`线程就会将我们的`Bin Log`发送给我们的`IO`线程**
- :four:`IO`线程接收到我们的`Bin Log`后就会**利用其更新自己的中继日志文件`Relay Log`**.
- :five:然后我们从服务进程的**`SQL`线程就会读取我们的`Relay Log`文件内容**,然后根据该文件中的内容对从服务进程的**数据库中的数据进行更新**.

![image-20220802134208719](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-179?token=AOAPFCLHQRBTMFZX23HTEQDC573EW)

### 主从架构的问题与原则

> 主从架构由于需要涉及到基于`Bin Log`的服务器数据同步,因此对于一个修改十分频繁的服务而言.主从服务器之间的数据是不会完全同步的,**会存在`延时`问题**

#### 基本原则

- 每个`Slave`从服务进程**只能从属于一个**`Master`主服务进程
- 每个`Slave`从服务进程都**必须**有其**唯一可标识的服务进程`ID`**
- 一个`Master`主服务进程可以**管控多个**`Slave`从服务进程

### 主从架构的搭建

#### 一主一从

> 在这样的架构下**`主服务进程`**一般被用于**处理所有客户端发起的`写数据请求`**,而**`从服务进程`**一般被用于**处理所有客户端发起的`读数据请求`**<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-180?token=AOAPFCPNI6OKIABUHIB3CCDC573FI" alt="image-20220802182404352" style="zoom:67%;" />

##### 基本流程

- :one:**准备`两台`主机**(可以是两台虚拟机)

- :two:**每台主机上都必须安装`MySQL`,并且最好能保证同版本**

  - 当然我们可以**先配置好一台虚拟机**,然后**通过虚拟机克隆的方式配置出另一台主机**.只不过此时我们**要注意以下几点**
    - **修改克隆的主机的网络的`MAC`地址**
    - **修改克隆的主机的`hostname`**
      - `Linux`下可以在`/etc/hostname`文件中修改
    - **修改克隆的主机的`IP`地址**
      - `Linux`下可在`/etc/netplan`的`.yaml`文件中修改
    - **修改克隆的主机的`UUID`**
      - `Linux`下可在`/var/lib/mysql/auto.cnf`文件中进行修改

- :three:**为从机创建账户**

  - ```mysql
    #5.5 5.7下为从主机创建账户,并给该账户授予主从复制的权限
    GRANT REPLICATION SLAVE ON *.* TO '账户名'@'从主机IP' IDENTIFIED BY '密码';
    #8.0下为从主机创建账户,并给该账户授予主从复制的权限
    CREATE USER '账户名'@'从主机IP' IDENTIFIED BY '密码';
    GRANT REPLICATION SLAVE ON *.* TO '账户名'@'从主机IP';
    ALTER USER '账户名'@'从主机IP' IDENTIFIED WITH mysql_native_password BY '密码';
    flush privileges;
    ```

- :four:**对主机的`MySQL`配置文件进行修改**

  - **必须项**

  ```shell
  [mysqld]
  #设置为1表示其为主服务器
  server_id=1
  #启用二进制日志BinLog
  log_bin=on
  #设置二进制日志文件的数据文件存储路径与文件前缀名
  log_bin_basename=路径/文件前缀名
  ##设置二进制日志文件的索引文件存储路径与文件名
  log_bin_index=路径/文件名.index
  ```

  - **可选设置**

  ```shell
  #指定是否只可读,0表示可读可写,1表示只可读
  innodddread_only=0
  #指定二进制dd文件的自动过期时间
  binlog_expire_logs_seconds=时长
  #指定单个二进制数据文件的大小
  max_binlog_size=字节数
  #设置不复制的数据库
  binlog-ignore-db=数据库名1[,数据库名2...]
  #设置需要复制的数据库
  binlog-do-db=需要复制的数据库名1[,需要复制的数据库名2]
  #指定二进制日志文件的数据文件中数据的保存格式STATEMENT|ROW|MIXED
  binlog_format=
  ```

  - **不同格式的区别**
    - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-181?token=AOAPFCI6HXXLPIASZSYMEJ3C573GC" alt="image-20220802203249951" style="zoom:67%;" />
    - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-182?token=AOAPFCMGDSV3OZTCHG4B44LC573G2" alt="image-20220802203310426" style="zoom:67%;" />
    - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-183?token=AOAPFCIQXJHSDWSURVNBO43C573HI" alt="image-20220802203324427" style="zoom:67%;" />\

- :five:**对从机的`MySQL`配置文件进行修改**

  - **必选项**

    ```shell
    [mysqld]
    #设置从机的ID
    server_id=2
    ```

  - **可选项**

    ```shell
    #开启从机的中继日志并指定中继日志的文件名
    relay_log=中继日志文件名
    #指定中继日志数据文件的存储位置
    relay_log_basenaem="存储路径/中继日志文件名"
    #指定中继日志索引文件的存储位置
    relay_log_index="存储路径/中继日志文件名.index"
    ```

- :six:**为从机创建连接主机的基本信息**

  ```mysql
  CHANGE MASTER TO
  MASTER_HOST='主机的IP地址',
  MASTER_USER='主机上设置的用户名',
  MASTER_PASSWORD='主机上设置的用户名的密码',
  MASTER_LOG_FILE='主机进行SHOW master status查询结果的File字段的值',
  MASTER_LOG_POS=主机进行SHOW master status查询结果的Position字段的值;
  ```

- :seven:**启动从服务器的主从复制**

  ```mysql
  START SLAVE;
  ```

- :eight:**其他拓展操作**

  - **重置从服务器的`Relay Log`**

    ```mysql
    RESET SLAVE;
    ```

  - **停止从服务器的主从复制**

    > **停止后要在从服务器上先运行`RESET SLAVE`再运行`START SLAVE`才能重新开启主从同步**

    ```mysql
    STOP SLAVE
    ```

  - **查看从服务器的主从同步状态**

    > **若`Slave_IO_Running`与`Slave_SQL_Running`都为`Yes`则主从复制配置成功**

    ```
    SHOW SLAVE STATUS
    ```

#### 双主双从

> **即建立多个主服务器处理`写请求`,以及多个从服务器用于处理`读请求`.配置过程与一主一从类似.只不过要按照下图的关系来配置我们的主从关系**

![image-20220802214343857](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-184?token=AOAPFCMIZOTN7RELJD6LW73C573HY)

![image-20220802214348777](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-185?token=AOAPFCPJ3A2TTD4Z7JTTOHLC573II)

### 主从架构的数据同步问题

> 进行主从同步的内容是**二进制日志**，它是一个文件，在进`行 网络传输 `的过程中就**一定会` 存在主从延迟
> `**（比如` 500ms`），这样就**可能造成用户在从库上读取的数据不是最新的数据**，也**就是主从同步中的` 数据
> 不一致性 `问题**  

#### 问题的产生原因剖析

> - 在网络正常的时候，日志从主库传给从库所需的时间是很短的，即`T2-T1`的值是非常小的。即，**网络正常情况下，主从延迟的主要来源是从服务器接收完`binlog`和执行完这个事务之间的时间差** 
>
> **网络正常情况下**的原因
>
> - 从库**机器的性能与主库相差大**
> - 从库**处于大量的事务并发处理状态下**,压力过大
> - 从库**正被大型查询事务占用**,在这种情况下往往锁表,锁记录,无法正常使用`Redo Log`进行数据同步 

#### 如何尽量减轻主从延迟问题

- :one:降低从服务器上的事务并发概率,避免从服务器压力过大
- :two:优化`SQL`语句,尽量避免从服务器上的慢查询
- :three:提高从库机器的配置  
- :four:尽量采用` 短的链路 `，也就是**主库和从库服务器的距离尽量要短**，提升端口带宽，减少`binlog`传输
  的网络延时。  
- :five:实时性要求的业务读强制走主库，从库只做灾备，备份  

#### 如何最终解决一致性问题

> **按数据一致性`从低到高`**

##### 异步复制

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-186?token=AOAPFCPDWVLBXIABPQ437W3C573I6" alt="image-20220802231416946" style="zoom: 67%;" />

##### 半同步复制

<img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-187?token=AOAPFCMO2GQRGAYQ4IA43VTC573JO" alt="image-20220802231429588" style="zoom:67%;" />

##### 组复制`MGR`

> 组复制技术，简称` MGR（MySQL Group Replication）`。是` MySQL `在` 5.7.17 `版本中推出的一种**新的数据复制技术，这种复制技术是基于` Paxos `协议的状态机复制**。  

- **工作原理**
  - 首先我们将多个节点(**主服务器,从服务器**)共同**组成一个复制组**
  - 根据上面的复制组构建起一个**一致性协议层`Consensus层`**
  - 当复制组内**某一个节点**要`执行读写(RW)事务`时,如果想要执行`COMMIT`,也就是读写事务想要进行提交,必须**在`Consensus层`经过复制组内一半以上的节点同意**才能执行,**否则必须阻塞**
  - 当复制组内**某个节点**要执行`只读(RO)事务`时,则**不加阻碍**,`Consensus层`或直接予以同意
  - ![image-20220802232543902](https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-188?token=AOAPFCMZJFKJGPG2CWPIWKTC573J4)

### 常用数据库中间件介绍

> **数据库中间件**
>
> - 数据库中间件即由其他人,或企业编写的一套帮助我们程序员**实现数据库读写分离策略的框架**
> - **数据库中间件**相比于我们**自己手动编码实现**读写分离的**优点在于**
>   - `功能强大`
>   - `使用简单`
>   - `快速开发`

#### 常见数据库中间件

- **`Cobar`**(**停止维护**)
  - 属于**阿里`B2B`事业群**，始于`2008`年，在阿里服役`3`年多，接管`3000+`个`MySQL`数据库的`schema`,集群日处理在线`SQL`请求`50亿次`以上。由于`Cobar`发起人的离职，**`Cobar`目前已经停止维护**  
- **`Mycat`**
  - 开源社区**在阿里`cobar`基础上进行二次开发**，解决了`cobar`存在的问题，并且加入了许多新的功能在其中。**青出于蓝而胜于蓝**。  
- **`OneProxy`**(**商业,收费,高性能高稳定**)
  - 基于`MySQL`官方的`proxy`思想利用`c语言`进行开发的，`OneProxy`是一款商业**` 收费 `的中间件**。舍弃了一些功能，**专注在` 性能和稳定性`上** 。  
- **`KingShard`**
  - **小团队**用`go`语言开发，还**需要发展，需要不断完善**  
- **`Vitess`**(**Youtube**)
  - `Youtube`生产在使用，**架构很复杂**。**不支持`MySQL`原生协议，使用` 需要大量改造成本`**  
- **`Atlas`**(**360,待完善**)
  - `360`团队基于`mysql proxy`改写，功能还需完善，高并发下不稳定  
- **`MaxScala`**(**`MySQL`原作者维护**)
  - `mariadb`（`MySQL`**原作者**维护的一个版本） 研发的中间件  
- **`MySQLRoute`**(**`MySQL`官方维护**)
  - `MySQL`官方`Oracle`公司发布的中间件  

## 数据库的备份与恢复

### 备份(`mysqldump`)

> **物理备份**:**直接备份数据文件，转储数据库物理文件到某一目录**
>
> - `恢复速度快`
> - `占用空间大`
> - **`MySQL`中可以用` xtrabackup `工具来进行物理备份**  
>
> **逻辑备份**:**对数据库对象利用工具进行导出工作，汇总入备份文件内**.
>
> - `恢复速度慢`
> - `占用空间小`
> - ==**逻辑备份就是备份`SQL`语句**==
> - **`MySQL `中常用的逻辑备份工具为` mysqldump `** 

#### 指定单数据库备份(**带有数据库创建语句**)

```shell
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 待备份的数据库名称 > 备份文件存储路径/备份文件名称.sql
```

#### 指定多数据库备份(**带有数据库创建语句,方式3不带有**)

```shell
#方式1
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> --databases 待备份的数据库名称1[] 待备份的数据库名称2...] > 备份文件存储路径/备份文件名称.sql

#方式2
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> -B 待备份的数据库名称1[] 待备份的数据库名称2...] > 备份文件存储路径/备份文件名称.sql

#方式3
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 待备份的数据库名称1[] 待备份的数据库名称2...] > 备份文件存储路径/备份文件名称.sql
```

#### 全数据库备份(**带有数据库创建语句**)

```shell
#方式1
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> --all-databases > 备份文件存储路径/备份文件名称.sql

#方式2
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> -A > 备份文件存储路径/备份文件名称.sql
```

#### 备份单个数据库下的部分表(**不带有数据库创建语句**)

```shell
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 数据库名称 表名1[] 表名2...] > 备份文件存储路径/备份文件名称.sql
```

#### 备份单个数据库下指定表的指定一些数据记录(**不带有数据库创建语句**)

```shell
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 数据库名称 表名 --where="<过滤条件>" > 备份文件存储路径/备份文件名称.sql
```

#### 备份据库下除某些数据表外的其他数据表(**不带有数据库创建语句**)

```shell
#单个数据库
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 数据库名称 --ignore-table = 数据库名称.表名1[ 数据库名称.表名2...] > 备份文件存储路径/备份文件名称.sql

#多个数据库
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> --databases 数据库名称1[ 数据库名称2...] --ignore-table = 数据库名称.表名1[ 数据库名称.表名2...] > 备份文件存储路径/备份文件名称.sql
```

#### 只备份结构或数据

```shell
#只备份结构
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 数据库名称> --no-data > 备份文件存储路径/备份文件名称.sql
#只备份数据
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 数据库名称> --no-create-info > 备份文件存储路径/备份文件名称.sql
```

#### 备份中包含存储过程、函数、事件

> `mysqldump`备份默认是**不包含存储过程，自定义函数及事件**的。可以使用` --routines `或` -R `选项来备
> 份**存储过程及函数**，使用` --events` 或` -E `参数来**备份事件**。  

```shell
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码>  -R -E待备份的数据库名称 > 备份文件存储路径/备份文件名称.sql
```

### `MySQLDump`常用选项

```shell
--password[=password]，-p[password]：当连接服务器时使用的密码。

--user=user_name，-u user_name：当连接服务器时MySQL使用的用户名。

-port=port_num，-P port_num：用于连接的TCP/IP端口号。--add-drop-database：在每个CREATE DATABASE语句前添加DROP DATABASE语句。

--add-drop-tables：在每个CREATE TABLE语句前添加DROP TABLE语句。

--add-locking：用LOCK TABLES和UNLOCK TABLES语句引用每个表转储。重载转储文件时插入得更快。

--all-database, -A：转储所有数据库中的所有表。与使用--database选项相同，在命令行中命名所有数据库。

--delete，-D：导入文本文件前清空表。

--default-character-set=charset：使用charsets默认字符集。如果没有指定，就使用utf8

--lock-all-tables，-x：对所有数据库中的所有表加锁。在整体转储过程中通过全局锁定来实现。该选项自动关
闭--single-transaction和--lock-tables。

--lock-tables，-l：开始转储前锁定所有表。用READ LOCAL锁定表以允许并行插入MyISAM表。对于事务表（例
如InnoDB和BDB），--single-transaction是一个更好的选项，因为它根本不需要锁定表。

--no-create-db，-n：该选项禁用CREATE DATABASE /*!32312 IF NOT EXIST*/db_name语句，如果给出-
-database或--all-database选项，就包含到输出中。

--no-create-info，-t：只导出数据，而不添加CREATE TABLE语句。

--no-data，-d：不写表的任何行信息，只转储表的结构。

-T :

--flush-logs，-F：开始转储前刷新MySQL服务器日志文件。该选项要求RELOAD权限。

#*****************************************************************************************#


--comment[=0|1]：如果设置为0，禁止转储文件中的其他信息，例如程序版本、服务器版本和主机。

--skipcomments与--comments=0的结果相同。默认值为1，即包括额外信息。

--compact：产生少量输出。该选项禁用注释并启用--skip-add-drop-tables、--no-set-names、--skipdisable-keys和--skip-add-locking选项。

--compatible=name：产生与其他数据库系统或旧的MySQL服务器更兼容的输出，值可以为ansi、MySQL323、
MySQL40、postgresql、oracle、mssql、db2、maxdb、no_key_options、no_table_options或者
no_field_options。

--complete_insert, -c：使用包括列名的完整的INSERT语句。

--debug[=debug_options], -#[debug_options]：写调试日志

。
--delete--master-logs：在主复制服务器上，完成转储操作后删除二进制日志。该选项自动启用-masterdata。

--extended-insert，-e：使用包括几个VALUES列表的多行INSERT语法。这样使得转储文件更小，重载文件时可
以加速插入。

--force，-f：在表转储过程中，即使出现SQL错误也继续。

--opt：该选项是速记，它可以快速进行转储操作并产生一个能很快装入MySQL服务器的转储文件。该选项默认开启，
但可以用--skip-opt禁用。


--protocol={TCP|SOCKET|PIPE|MEMORY}：使用的连接协议。

--replace，-r –replace和--ignore：控制替换或复制唯一键值已有记录的输入记录的处理。如果指定--
replace，新行替换有相同的唯一键值的已有行；如果指定--ignore，复制已有的唯一键值的输入行被跳过。如果不
指定这两个选项，当发现一个复制键值时会出现一个错误，并且忽视文本文件的剩余部分。

--silent，-s：沉默模式。只有出现错误时才输出。

--socket=path，-S path：当连接localhost时使用的套接字文件（为默认主机）。

--verbose，-v：冗长模式，打印出程序操作的详细信息。

--xml，-X：产生XML输出。
```

### 恢复(`mysql`)

> **我们上面通过`mysqldump`做的数据备份其实就是备份了一个由众多`SQL`语句组成的`.sql`文件,因此当我们需要对`mysqldump`备份的数据进行恢复时,只需要想办法批量运行`.sql`文件中的`SQL`文件即可**

#### 单库备份中恢复单库

- **当`.sql`文件中包含了数据库创建的语句时**

  ```shell
  mysql –u <用户名称> –h <主机名称> -P <端口号> –p <密码> < 备份文件存储路径/备份文件名称.sql
  ```

- **当`.sql`文件中不包含数据库创建的语句时**

  ```shell
  mysql –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 数据库名称 < 备份文件存储路径/备份文件名称.sql
  ```

#### 单库备份中恢复单表

> **这样的情况下我们必须要多一道`.sql`文件分离操作**,将我们需要的**数据表对应的语句**从我们的全量备份的`.sql`文件中**分离出来**并存入另一个`.sql`文件

```shell
mysql –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 数据库名 < 备份文件存储路径/备份文件名称.sql
```

#### 全量备份中恢复全部库

```shell
mysql –u <用户名称> –h <主机名称> -P <端口号> –p <密码> < 备份文件存储路径/备份文件名称.sql
```

#### 全量备份中恢复单库

> **这样的情况下我们必须要多一道`.sql`文件分离操作**,将我们需要的**数据库对应的语句**从我们的全量备份的`.sql`文件中**分离出来**并存入另一个`.sql`文件

- **分离**

- **恢复**

  - **包含数据库创建语句时**

  ```shell
  mysql –u <用户名称> –h <主机名称> -P <端口号> –p <密码> < 备份文件存储路径/备份文件名称.sql
  ```

  - **不包含数据库创建语句时**

  ```shell
  mysql –u <用户名称> –h <主机名称> -P <端口号> –p <密码> 数据库名 < 备份文件存储路径/备份文件名称.sql
  ```

### 物理备份与物理恢复

## ``MySQL``数据表的导入与导出

### 导出

> **即将`SELECT`查询的结果写入到我们指定的文本文件中**

#### 基于**`SELECT INTO OUTFILE`**语句的导出

- **前提**

  - `MySQL`默认情况下对导出的文本文件的存储目录有限制,即只能保存在`MySQL`服务进程允许的目录下

  -   **查看`MySQL`指定的存储目录**

    ```mysql
    SHOW VARIABLES LIKE "%secure"
    ```

- **基本语法**

  ```mysql
  SELECT语句 INTO OUTFILE "存储路径/文件名.txt"
  
  SELECT * FROM atguigu.account INTO OUTFILE '/var/lib/mysql-files/account_1.txt' FIELDS
  TERMINATED BY ',' ENCLOSED BY '\"';
  ```

  - **`FIELDS TERMINATED BY`**:用于**指定**文本文件中存储时**字段之间的分隔符**
  - **`ENCLOSED BY`**:用于**指定**文本文件中存储时**各个记录下各个字段的值的包括符号**

#### 基于`mysqldump`的导出

> 这种方式会生成两个文件,**一个后缀为`.sql`,一个后缀为`.txt`**,且**文件名均为我们的数据表名**

```shell
mysqldump –u <用户名称> –h <主机名称> -P <端口号> –p <密码> -T "存储路径/" 数据库名 数据表名
```

#### 基于`mysql`的导出

- **方式1**

  ```shell
  mysql -uroot -p --execute = "SELECT * FROM account;" atguigu > "/var/lib/mysqlfiles/account.txt"
  ```

  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-189?token=AOAPFCJHTXSLXC3B7NKEFR3C573LA" alt="image-20220802164022499" style="zoom: 67%;" />

- **方式2**

  ```shell
  mysql -uroot -p --vertical --execute="SELECT * FROM account;" atguigu >
  "/var/lib/mysql-files/account_1.txt"
  ```

  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-190?token=AOAPFCJKIH47QP6GDUYOKNDC573LO" alt="image-20220802164105113" style="zoom: 50%;" />

- **导出到`XML`文件**

  ```shell
  mysql -uroot -p --xml --execute="SELECT * FROM account;" atguigu>"/var/lib/mysqlfiles/account_3.xml"
  ```

  - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-191?token=AOAPFCK52X5M64Q2XZYHKCTC573MC" alt="image-20220802164213538" style="zoom:60%;" />

- **导出到`HTML`文件**

  ```shell
  mysql -uroot -p --HTML --execute="SELECT * FROM account;" atguigu>"/var/lib/mysqlfiles/account_3.xml"
  ```

### 导入

#### 基于`LOAD DATA INFILE`的导入

- ```mysql
  LOAD DATA INFILE '/var/lib/mysql-files/account_0.txt' INTO TABLE atguigu.account;
  
  LOAD DATA INFILE '/var/lib/mysql-files/account_1.txt' INTO TABLE atguigu.account
  FIELDS TERMINATED BY ',' ENCLOSED BY '\"';
  ```

  - **`FIELDS TERMINATED BY`**:用于**指定**文本文件中存储时**字段之间的分隔符**
  - **`ENCLOSED BY`**:用于**指定**文本文件中存储时**各个记录下各个字段的值的包括符号**

#### 基于`mysqlimport`的导入

```shell
mysqlimport -uroot -p atguigu '/var/lib/mysql-files/account.txt' --fields-terminatedby=',' --fields-optionally-enclosed-by='\"'
```

- `--fields-terminatedby`:用于**指定**文本文件中存储时**字段之间的分隔符**
- `--fields-optionally-enclosed-by`:用于**指定**文本文件中存储时**各个记录下各个字段的值的包括符号**

## `MySQL`数据库迁移

## `MySQL`误操作修复

## `MySQL`常用命令行工具拓展

### `mysql`

### `mysqlbinlog`

### `mysqladmin`

### `mysqldump`

### `mysqlimport`

### `mysqlshow  `

> **客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引。**  

- **基本语法**

  ```shell
  mysqlshow [options] [数据库名 [表名 [列名]]]
  ```

- **基本参数**

  ```shell
  #连接参数
  -u, --user=name 		指定用户名
  -p, --password[=name] 	指定密码
  -h, --host=name 		指定服务器IP或域名
  -P, --port=				指定连接端口
  #内容参数
  --count 显示数据库及表的统计信息（数据库，表 均可以不指定）
  -i 		显示指定数据库或者指定表的状态信息
  ```

- **注意事项**

  - **当不指定数据库名与表名这些选项时**
    - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-192?token=AOAPFCPNNQQ7LCEWSRQNFULC573M6" alt="image-20220802152217291" style="zoom:55%;" />
  - **当只指定数据库名不指定表名时**
    - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-193?token=AOAPFCLNZSJDLVD2SCEYPRLC573NM" alt="image-20220802152321494" style="zoom:55%;" />
  - **当指定数据库名数据表名,不指定列名时**
    - <img src="https://raw.githubusercontent.com/tangling0112/MyPictures/master/img/MySQL%E8%BF%9B%E9%98%B6-194?token=AOAPFCM43PAZGA3YYJ55J2DC573OA" alt="image-20220802152439911" style="zoom:55%;" />
