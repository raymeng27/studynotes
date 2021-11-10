# 数据库

# 一、数据库简介

# 二、关系模型数据库

# 三、SQL

> SQL最早的版本是由IBM开发的，它最初被叫做Sequel，Sequel语言一致发展至今，其名称已变为SQL(结构化查询语言)。1986年ANSI和ISO发布了SQL标准：SQL-86，1989年ANSI发布了一个SQL的扩充标准：SQL-89。该标准的下一个版本是SQL-92标准，接着是SQL:1999，SQL:2003，SQL:2006，SQL:2011。目前（2021年）最新的版本是SQL:2016。

## 1.SQL基本概述

SQL语言有以下几部分：

* 数据定义语言（Data-Definition Language：DDL）：SQL DDL提供定义关系模式，删除关系以及修改关系模式的命令。
* 数据操纵语言（Data-Manipulation Language：DML）：SQL DML提供从数据库中查询信息，以及在数据库中插入元组，删除元组，修改元组的能力。
* 完整性（integrity）：SQL DDL包括定义完整性约束的命令，保存在数据库中的数据必须满足所定义的完整性约束。破坏完整性约束的更新是不允许的。
* 视图定义（view definition）：SQL DDL包括定义视图的命令。
* 事务控制（transaction control）：SQL包括定义事务的开始和事务的结束的命令。
* 嵌入式SQL和动态式SQL（embedded SQL and dynamic SQL）：嵌入式和动态SQL定义SQL语句如何嵌入到通用编程语言，如C、C++和Java中。
* 授权（authorzation）：SQL DDL包括定义对关系和视图的访问权限的命令。

## 2.SQL数据定义

### a.基本类型

SQL标准支持多种固有类型，如下：

- char(n)：固定长度的字符串，用户指定长度n，也可以使用全称character。

- varcahr(n)：可变长度的字符串，用户指定最大长度n。

- int：整数类型，等价于全称Integer。

- smallint：小整数类型。

- numeric(p,d)：定点数，精度由用户指定。这个数有p位数字（加上一个符号位），其中d位数字在小数点的右边。

- real，double precision：浮点数与双精度浮点数。

- float(n)：精度至少为n为的浮点数。

- SQL中的日期和时间类型

  - date：日历日期，包括年（四位），月，日。
  - time：一天中的时间，包括小时，分，秒。可以用变量time(p)来表示秒的小数点后的数字位数（默认是0）。
  - timestamp：date和time的组合。可以使用timestamp(p)来表示秒的小数点后的数字位数（默认值是6）。

- default：默认值，如下

  ```sql
  CREATE TABLE default_student(
  	id INT AUTO_INCREMENT PRIMARY KEY,
  	NAME VARCHAR(20) NOT NULL,
  	tot_cred NUMERIC(3,1) DEFAULT 10.0
  );
  ```

- 大对象类型：SQL提供字符数据的大对象数据类型clob，以及二进制数据的大对象数据类型blob。例如：

  ```sql
  CREATE TABLE person(
  	id INT AUTO_INCREMENT PRIMARY KEY,
  	NAME VARCHAR(20) NOT NULL,
      describe clob,
      photo blob
  );
  ```

  ***Tips：MySQL中不支持clob，在MySQL中使用的text。***

  MySQL支持的数据类型：

  |     类型     |                   大小                   |                        范围（有符号）                        |                        范围（无符号）                        |      用途       |
  | :----------: | :--------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :-------------: |
  |   TINYINT    |                  1 byte                  |                         (-128，127)                          |                           (0，255)                           |    小整数值     |
  |   SMALLINT   |                 2 bytes                  |                      (-32 768，32 767)                       |                         (0，65 535)                          |    大整数值     |
  |  MEDIUMINT   |                 3 bytes                  |                   (-8 388 608，8 388 607)                    |                       (0，16 777 215)                        |    大整数值     |
  | INT或INTEGER |                 4 bytes                  |               (-2 147 483 648，2 147 483 647)                |                      (0，4 294 967 295)                      |    大整数值     |
  |    BIGINT    |                 8 bytes                  |   (-9,223,372,036,854,775,808，9 223 372 036 854 775 807)    |               (0，18 446 744 073 709 551 615)                |   极大整数值    |
  |    FLOAT     |                 4 bytes                  | (-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) |         0，(1.175 494 351 E-38，3.402 823 466 E+38)          | 单精度 浮点数值 |
  |    DOUBLE    |                 8 bytes                  | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度 浮点数值 |
  |   DECIMAL    | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 |                        依赖于M和D的值                        |                        依赖于M和D的值                        |     小数值      |

  |   类型    | 大小 ( bytes) |                             范围                             | 格式                |           用途           |
  | :-------: | :-----------: | :----------------------------------------------------------: | :------------------ | :----------------------: |
  |   DATE    |       3       |                    1000-01-01/9999-12-31                     | YYYY-MM-DD          |          日期值          |
  |   TIME    |       3       |                   '-838:59:59'/'838:59:59'                   | HH:MM:SS            |     时间值或持续时间     |
  |   YEAR    |       1       |                          1901/2155                           | YYYY                |          年份值          |
  | DATETIME  |       8       |           1000-01-01 00:00:00/9999-12-31 23:59:59            | YYYY-MM-DD HH:MM:SS |     混合日期和时间值     |
  | TIMESTAMP |       4       | 1970-01-01 00:00:00/2038结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |

  |    类型    |         大小          |              用途               |
  | :--------: | :-------------------: | :-----------------------------: |
  |    CHAR    |      0-255 bytes      |           定长字符串            |
  |  VARCHAR   |     0-65535 bytes     |           变长字符串            |
  |  TINYBLOB  |      0-255 bytes      | 不超过 255 个字符的二进制字符串 |
  |  TINYTEXT  |      0-255 bytes      |          短文本字符串           |
  |    BLOB    |    0-65 535 bytes     |     二进制形式的长文本数据      |
  |    TEXT    |    0-65 535 bytes     |           长文本数据            |
  | MEDIUMBLOB |  0-16 777 215 bytes   |  二进制形式的中等长度文本数据   |
  | MEDIUMTEXT |  0-16 777 215 bytes   |        中等长度文本数据         |
  |  LONGBLOB  | 0-4 294 967 295 bytes |    二进制形式的极大文本数据     |
  |  LONGTEXT  | 0-4 294 967 295 bytes |          极大文本数据           |

### b.基本模式定义

使用create table定义关系，通用命令 create table r (A1 D1,...,An Dn,<完整性约束1>,...,<完整性约束k>);其中r是关系（表名），每个Ai是列名，Di是属性Ai的域（列的类型）。SQL支持许多不同的完整性约束：

* primary key(列名)：primary key声明列构成关系的主码。主码属性必须非空且唯一，主码的声明是可选的。
* foreign key(列名) references：foreign key声明表示关系中任意元组在属性上的取值必须对应于关系s中某元素在主码属性上的取值。
* not null：一个属性上的not null约束表明在该属性上不允许空值。

### c.SQL查询的基本结构

```sql
select A1,...An
from r1,...rn
where P;
```

每一个Ai代表一个列，每一个ri代表一个表。P是条件语句，如果省略了where子句，则P为true。

### d.附加的基本运算

- SQL提供了一个重命名结果关系中属性的方法。形式如：old-name as new-name，as子句既可以出现在select子句中，也可以出现在from子句中。例子如下：

  ```sql
  select name as instructor_name, course_id
  from instructor,teaches
  where instructor.id = teaches.id;
  ```

- 字符串运算

  SQL使用一对单引号来标示字符串，例如'Computer'。如果单引号是字符串的组成部分，那就用两个单引号字符来标示，如字符串"it's OK"。SQL在字符串上可以使用like操作来实现模式匹配。有以下两个特殊的字符来描述模式。

  - 百分号(%)：匹配任意子串。
  - 下划线(_)：匹配任意一个字符。

  例如：

  ```sql
  select dept_name
  from department
  where building like '%Watson%';
  ```

  表示找出所在建筑名称中包含子串'Watson'的所有系名。SQL允许定义转义字符，转义字符直接放在特殊字符的前面，表示该特殊字符被当成普通字符。在like比较运算中使用escape关键词来定义转义字符。如：like 'ad\%cd%' escape '\\' 匹配所有以“ab%cd”开头的字符串。

- "*"号可以用在select子句中表示“所有的属性”。

- 排序元组的显示次序使用 order by子句，order by默认使用升序（asc），如果想要使用降序，需要用desc表示，如下：

  ```sql
  select *
  from instructor
  order by salary desc,name asc
  ```

- where子句谓语

  SQL提供between比较运算符来说明一个值是小于或等于某个值，还可以使用not between，如下：

  ```sql
  select name
  from instructor
  where salary between 9000 and 10000;
  ```

  SQL允许用记号(v1,...vn)来表示一个分量值分别为v1,...vn的n维元组。在元组上可以运用比较运算符，按字典顺序进行比较运算。例如(a1,a2) <= (b1,b2)在a1 <= b1且a2 <= b2时为真。如下：

  ```sql
  select name,course_id
  from instructor,teaches
  where (instructor.id,dept_name) = (teaches.id,'Biology');
  ```


### e.集合运算

SQL作用在关系上的union、intersect和except运算对应于数学集合论中的∪、∩和﹣运算。

- 并运算

  ```sql
  (select course_id from section where semester = 'Fall' and year = 2009)
  union
  (select course_id from section where semester = 'Spring' and year = 2010);
  ```

  与select子句不同，union运算自动去除重复。如果想保留重复，就必须用union all代替union。

- 交运算

  ```sql
  (select course_id from section where semester = 'Fall' and year = 2009)
  intersect
  (select course_id from section where semester = 'Spring' and year =2010);
  ```

  intersect运算自动去除重复，如果想保留重复，就必须用intersect all代替intersect。

- 差运算

  ```sql
  (select course_id from section where semester = 'Fall' and year = 2009)
  except
  (select course_id from section where semester = 'Spring' and year =2010);
  ```

  except运算从其第一个输入中输出所有不出现在第二个输入中的元组。次运算自动去除重复的元组，如果想保留所有重复，就必须用except all 代替except。

### f.空值

如果算术表达式的任一输入为空，则该**算术表达式**（涉及诸如+、-、*或/）结果为空。SQL将涉及空值的任何**比较运算**的结果视为unknown。由于where子句的谓词中可以对比结果使用诸如and，or和not的布尔运算，所以这些布尔运算的定义也被扩展到可以处理unknown值。

- and：true and unknown的结果为unknown，false and unknown结果是false，unknown and unknown的结果是unknown。
- or：true or unknown的结果是true，false and unknown的结果是unknown，unknown and unknown的结果是unknown。
- not：not unknown的结果是unknown。

### g.聚集函数

聚集函数是以值的一个集合为输入、返回单个值的函数。SQL提供了五个固有聚集函数：

平均值：avg	最小值：min	最大值：max	总和：sum	计数：count

例：

```sql
select svg(salary) from instructor where dept_name = 'Copm.Sci';
```

* 分组聚集

  group by子句可以对给出的一个或多个属性是用来构造分组的。如：

  ```sql
  select depet_name,avg(salary) as salary_avg from instructor group by depet_name;
  ```

  当SQL查询使用分组时，任何没有出现在group by子句中的那些属性如果出现在select子句中，它只能出现在聚集函数内部，否则这样的查询是错误的。

* having子句

  有时候，对分组限定条件比对元组限定条件更有用。having子句中的谓词在形成分组后才起作用。如下：

  ```sql
  select dept_name,avg(salary) as avg_salary
  from instructor
  group by dept_name
  having avg(salary) > 42000;
  ```

  与select子句的情况类似，任何出现在having子句中，但没有被聚集的属性必须出现在group by子句中，否则查询是错误的。

* 对空值和布尔值的聚集

  SQL任务sum运算符应忽略输入中的null值，除了count(*)外所有的聚集函数都忽略输入集合中的空值。规定空集的count运算值为0。

### h.数据库的修改

* 删除delete

  删除语句:

  ```sql
  delete from r where P;
  ```

  其中P是一个谓词，r代表一个关系。delete语句首先从r中找出所有是P为真的元组，然后把它们从r中国删除。如果省略where子句，则r中所有元组都将被删除。例：

  ```sql
  delete from instructor where salary between 1000 and 2990;
  ```

* 插入insert

  要往关系中插入数据，可以指定待插入的元组，或者写一条查询语句来生成待插入的元组集合。带插入元组的属性值必须在相应属性的域中。例：

  ```sql
  insert into course values ('meng',99);
  ```

  此例中，元组属性值的排列顺序和表中属性排列的顺序一致。如果想对指定属性插入数据，需要在insert语句中指定属性。例：

  ```sql
  insert into course(name,score) values ('lay',90);
  ```

* 更新update

  update语句和insert、delete语句类似，待更新的元组可以用查询语句找到，例：

  ```sql
  update instructor set salary = salary*1.5;
  ```

  update语句的where子句可以包含seletc语句的where子句中的任何合法结构。

  SQL提供case结构，可以利用它在一条update语句中执行多种更新。格式一般如下：

  ```sql
  update instructor
  set salary = case
  				when pred1 then result1
  				...
  				else result0
  			end;
  ```

  当i是第一个满足的pred1...predn时，次操作就会返回resulti，如果没有，则返回result0。

# 四、中级SQL