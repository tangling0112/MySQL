## 1 计算列

- ```SQL
  DROP TABLE IF EXISTS TEST;
  CREATE TABLE TEST(
  a INT,
  b INT,
  c INT GENERATED ALWAYS AS (a + b) VIRTUAL
  );
  INSERT INTO TEST(a,b)
  VALUES(50,15);
  
  SELECT *
  FROM TEST; 
  /*
  此时C就是我们的计算列其效果为,我们向表中插入数据时,可以认为表中只有a,b两个字段.而至于字段c,系统会自动根据我们的字段a,b来生产
  */
   
  ```

## 2 `NATUAL JOIN`与`USING`

## 3 窗口函数

> **单行函数,聚合函数也可以搭配着我们的窗口使用,至于其输出我们可以用窗口函数的计算流程推理获得**

### 标准语法结构

- **形式1**

  ```mysql
  函数 OVER([PARTITION BY 字段名 ORDER BY 字段名 ASC|DESC])
  FROM子句
  [WHERE子句
  GROUP BY子句
  HAVING子句
  ORDER BY子句
  LIMIT子句]
  #[PARTITION BY 字段名 ORDER BY 字段名 ASC|DESC]被称为一个窗口
  ```

- **形式2**

  ```mysql
  函数 OVER 窗口名
  FROM子句
  [WHERE子句
  GROUP BY子句
  HAVING子句]
  WINDOW 窗口名 AS ([PARTITION BY 字段名 ORDER BY 字段名 ASC|DESC])
  [ORDER BY子句
  LIMIT子句]
  ```

### 案例

### 常用的窗口函数

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706181405043.png" alt="image-20220706181405043" style="zoom:85%;" />

- `ROW_NUMBER()`:**分组隔离**,对数据中的序号进行顺序显示,并且不同的`PARTITION BY`分组之间不共享序号

  - 也就是若``A分组``后紧跟``B分组``,当``A分组``的**最后一条记录**的序号达到``6``时,第``B分组``的**第一个记录**的序号会从``1``开始而**不是**``7``

  ```mysql
      SELECT
  
      ROW_NUMBER() OVER(PARTITION BY category_id ORDER BY price DESC) AS row_num,
      id, category_id, category, NAME, price, stock
  
      FROM goods;
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706181540173.png" alt="image-20220706181540173" style="zoom:85%;" />

- `RANK()`:**分组隔离**,该函数作用是排序函数,其会对同一个分组中的所有记录按照`ORDER BY`子句的排序结果对**每个记录给出一个排序值**.并且**当两个记录的排序字段的值相同的时候,这两个字段会获得同一个排序值**

  - **注意**:我们从下面表中可以发现在第一个分组中,由于第二三条记录使用了相同的排序值,因此排序值`3`便没有在该分组中出现,而是直接在下一个记录将排序值跳到了`4`.**我们把这称为忽略重复排序值**

  ```mysql
  SELECT
  
  RANK() OVER(PARTITION BY category_id ORDER BY price DESC) AS row_num,
  id, category_id, category, NAME, price, stock
  
  FROM goods
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706182438809.png" alt="image-20220706182438809" style="zoom:85%;" />

- `DENSE_RANK()`:**分组隔离**,与`RANK()`基本相同只不过其是**不忽略重复值**的

  ```mysql
  SELECT
  
  DENSE_RANK() OVER(PARTITION BY category_id ORDER BY price DESC) AS row_num,
  id, category_id, category, NAME, price, stock
  
  FROM goods;
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706183008427.png" alt="image-20220706183008427" style="zoom:85%;" />

- `PERCENT_RANK()`:**分组隔离**,其值与`RANK()`计算出的值`rank`有关,我们不妨令`PERCENT_RANK()`的值为`P_rank`,令当前分组的总记录数为`rows`

  - 我们有
    $$
    P_{rank}=\frac{rank-1}{rows-1}
    $$
    

  ``` mysql
  #写法一：
  SELECT RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS r,
  PERCENT_RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS pr,
  id, category_id, category, NAME, price, stock
  FROM goods
  WHERE category_id = 1;
  
  #写法二：
  SELECT RANK() OVER w AS r,
  PERCENT_RANK() OVER w AS pr,
  id, category_id, category, NAME, price, stock
  FROM goods
  WHERE category_id = 1 WINDOW w AS (PARTITION BY category_id ORDER BY price DESC);
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706183254690.png" alt="image-20220706183254690" style="zoom:85%;" />

- `CUME_DIST()`:**分组隔离**,我们令该函数的输出值为`C_Percent`,其所在分组的用于排序的字段的所有记录中最大取值为`Max`,当前处理的记录的用于排序的字段的取值为`Cur`,那么其取值符合以下公式
  $$
  C_{Percent}=\frac{Cur}{Max}
  $$
  

  ```mysql
  SELECT
  CUME_DIST() OVER(PARTITION BY category_id ORDER BY price ASC) AS cd,
  id, category, NAME, price
  FROM goods;
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706183817962.png" alt="image-20220706183817962" style="zoom:85%;" />

- `LAG(expr,n)`:**分组隔离**,其返回值为当前处理的记录的前`n`条记录作为`expr`表达式的输入得到的输出值

  - **注意**
    - 这个`n`的取值是有限制的,假如第一分组中只有`6`条记录,而我们取`n=6`,那么对于第一分组的最后一条条记录而言,其前面第`6`条数据显然不属于第一分组,因此`LAG(expr,n)`此时的返回值就会是`NULL`,同样的,第一分组的倒数第二条记录的`LAG(expr,n)`的返回值也会是`NULL`,事实上第一分组的任何一条记录调用该函数的返回值都会是`NULL`
    - 同样的对于第二分组而言,假如第二分组中只有`6`条记录,而我们取`n=6`,那么对于第一分组的最后一条条记录而言,其前面第`6`条数据显然不属于第二分组,而属于第一分组,因此`LAG(expr,n)`此时的返回值就会是`NULL`,同样的,第二分组的倒数第二条记录的`LAG(expr,n)`的返回值也会是`NULL`,事实上第二分组的任何一条记录调用该函数的返回值都会是`NULL`

  ```mysql
  SELECT id, category, NAME, price, pre_price, price - pre_price AS diff_price
  FROM (
  		SELECT id, category, NAME, price,LAG(price,1) OVER w AS pre_price
  		FROM goods
  		WINDOW w AS (PARTITION BY category_id ORDER BY price)
  	 ) t;
  #显然第一条记录由于前一条记录不属于第一分组因此其LAG(price,1)的返回值为NULL,因此29.9-NULL=NULL
  #第二条记录的前一条记录的price=29.9,因此LAG(price,1)的返回值为29.9,因此39.9-29.9=10.0
  ...
  #显然第七条记录由于前一条记录不属于第二分组,而属于第一分组因此其LAG(price,1)的返回值为NULL,因此59.9-NULL=NULL
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706220231540.png" alt="image-20220706220231540" style="zoom:85%;" />

- `LEAD(expr,n)`:**分组隔离**,其作用与`LAG(expr,n)`几乎相同,只不过是取得后`n`条记录

  - **注意**:这个`n`的取值是有限制的,加入第一分组中只有`6`条记录,而我们取`n=6`,那么对于第一分组的第一条记录而言,其后面第`6`条数据显然不属于第一分组,因此`LEAD(expr,n)`此时的返回值就会是`NULL`

  ```mysql
  SELECT id, category, NAME, behind_price, price,behind_price - price AS
  diff_price
  FROM(
  		SELECT id, category, NAME, price,LEAD(price, 1) OVER w AS behind_price
  		FROM goods WINDOW w AS (PARTITION BY category_id ORDER BY price)
  	) t;
  	
  #显然第一条记录的后一条记录的price=39.9,因此LEAD(price,1)的返回值为39.9,因此39.9-29.9=10.0
  #显然第二条记录的后一条记录的price=79.9,因此LEAD(price,1)的返回值为79.9,因此79.9-39.9=40.0
  #由于第六条记录的当前分组
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706223521881.png" alt="image-20220706223521881" style="zoom:85%;" />

- `FIRST_VALUE(expr)`:**分组隔离**,**当任意一个第一分组**的记录调用`FIRST_VALUE(expr)`时,都**会将第一分组的排序在第一条的记录作为函数的输入**.同样的当**任意一个第二分组**的记录调用`FIRST_VALUE(expr)`时,都**会将第二分组的排序在第一条的记录作为函数的输入**.

  ```mysql
  SELECT id, category, NAME, price, stock,FIRST_VALUE(price) OVER w AS first_price
  FROM goods 
  WINDOW w AS (PARTITION BY category_id ORDER BY price);
  #显然第一分组的第一条记录id=5,第二分组的第一条记录id=9
  #显然第一条记录处于第一分组,因此其将id=5的记录作为FIRST_VALUE(price)的输入,其返回值显然为29.9
  #第二条记录处于第一分组,因此其将id=5的记录作为FIRST_VALUE(price)的输入,其返回值显然为29.9
  #第三条记录处于第一分组,因此其将id=5的记录作为FIRST_VALUE(price)的输入,其返回值显然为29.9
  ...
  #第七条记录处于第二分组,因此其将id=9的记录作为FIRST_VALUE(price)的输入,其返回值显然为59.9
  #第八条记录处于第一分组,因此其将id=9的记录作为FIRST_VALUE(price)的输入,其返回值显然为59.9
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706221341665.png" alt="image-20220706221341665" style="zoom:85%;" />

- `LAST_VALUE(expr)`:**分组隔离**,与`FIRST_VALUE(expr)`基本相同,只不过当它被调用时是以当前记录所在分组的最后一条记录作为输入.

  ```mysql
  SELECT id, category, NAME, price, stock,LAST_VALUE(price) OVER w AS last_price
  FROM goods WINDOW w AS (PARTITION BY category_id ORDER BY price);
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706224349265.png" alt="image-20220706224349265" style="zoom:85%;" />

- `NTH_VALUE(expr,n)`:**分组隔离**,与`FIRST_VALUE(expr)`基本相同,只不过当它被调用时是以当前记录所在分组的从上往下数第`n`条记录作为输入.

  ```mysql
  SELECT id, category, NAME, price,NTH_VALUE(price,2) OVER w AS second_price,
  NTH_VALUE(price,3) OVER w AS third_price
  FROM goods
  WINDOW w AS (PARTITION BY category_id ORDER BY price);
  
  #显然第一分组的第2条记录id=1,第二分组的第2条记录id=7
  #显然第一条记录处于第一分组,但是该的记录作为FIRST_VALUE(price)的输入,其返回值显然为39.9
  #第二条记录处于第一分组,因此其将id=5的记录作为FIRST_VALUE(price)的输入,其返回值显然为29.9
  #第三条记录处于第一分组,因此其将id=5的记录作为FIRST_VALUE(price)的输入,其返回值显然为29.9
  ...
  #第七条记录处于第二分组,因此其将id=9的记录作为FIRST_VALUE(price)的输入,其返回值显然为59.9
  #第八条记录处于第一分组,因此其将id=9的记录作为FIRST_VALUE(price)的输入,其返回值显然为59.9
  ```

  <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706222243743.png" alt="image-20220706222243743" style="zoom:85%;" />

- `NTILE(n)`:**分组隔离**,其会将每个分组中的数据按顺序分为`n`个小组.

  - 若有`6`条记录,要分`4`组,那么分组情况为.若有`5`条记录要分`4`组,那么分组情况为
    - <img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706224850355.png" alt="image-20220706224850355" style="zoom:67%;" />

```mysql
SELECT NTILE(3) OVER w AS nt,id, category, NAME, price
FROM goods WINDOW w AS (PARTITION BY category_id ORDER BY price);
```

<img src="F:\A_Java_DataBase_Study_FIle\DataBase\MySQL\06MySQL 8.0新特性.assets\image-20220706224738148.png" alt="image-20220706224738148" style="zoom:85%;" />

## 4 公用表表达式

### 普通公用表表达式

> 我们可以这样理解:公用表表达式就相当于`C语言`中的``宏定义#define``,其**给我们的子查询语句起了一个别名**,当我们查询语句中**使用到**公用表表达式起的**别名**时,`MySQL`服务器就**会将该别名自动文本替换**成我们的对应**子查询语句**

- **创建公用表表达式**

  ```mysql
  WITH <别名>
  AS
  (<子查询语句>);
  ```

- **案例**

  ```mysql
  #创建公用表表达式
  WITH emp_dept_id
  AS (SELECT DISTINCT department_id FROM employees)
  
  #使用公用表表达式
  SELECT *
  FROM departments d JOIN emp_dept_id e
  ON d.department_id = e.department_id;
  ```

### 递归公用表表达式

> 递归公用表表达式

- **创建递归公用表表达式**

  ```mysql
  WITH RECURSIVE <名称>
  AS
  (种子查询
  UNION [ALL]
  递归子查询
  );
  ```

- **案例**

  ```mysql
  #定义递归公用表表达式
  WITH RECURSIVE cte
  AS
  ( 
  	SELECT employee_id,last_name,manager_id,1 AS n FROM employees WHERE employee_id = 100 -- 种子查询，找到第一代领导
  
      UNION ALL
  	
      SELECT a.employee_id,a.last_name,a.manager_id,n+1
      FROM employees AS a
      JOIN cte ON (a.manager_id = cte.employee_id) -- 递归查询，找出以递归公用表表达式的人为领导的人
  )
  
  #使用递归公用表表达式
  SELECT employee_id,last_name FROM cte WHERE n >= 3;
  ```
