# Mysql-关系型数据库

## 1.什么是关系型数据库

### 1.1 概念

```
以关系代数理论提出的关系模型，以元祖为操作单元！（相当于数据库的一行）
以及可以用ER图表示的关系模型图
所有数据库的操作都可以转化成关系代数的理论操作，例如集合的交差补并，以及元祖单个元素获取
```

### 1.2 数据库设计范式

+ 第一范式（1NF）：每个属性列原子不可拆分，但学号-姓名做一列，那就不是第一范式，如果存在非主属性针对码的部分依赖，也破坏了第一范式

+ 第二范式（2NF)：满足第一范式的基础上，每一列都要有主键，但如果存在传递依赖，那就破坏了第二范式，例如 学号 姓名 班级 人数

+ 第三范式（3NF）：在满足第二范式的基础上，每一列都要有所属主键 但如果存在多主属性 例如 仓库  管理员 物品名，数量，仓库，管理员 都是主属性，那么就会冗余

  ![1615387790316](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615387790316.png)

+ BCNF(巴斯科德范式)：在满足第三范式的基础上，消除多主属性

![1615387799105](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615387799105.png)

```
超键：一般是候选键中最小数量的那个 如学生编号
侯选键：能标志元组唯一的各种组合 如 学生编号 、学生编号姓名等等
主键：候选键选择一个作为主键
码：就是侯选键
主属性：侯选键中的属性
非主属性：与主属性相对
消除非主属性对码的部分依赖 例如码值：学生编号 班级编号 非主属性姓名对码有部分依赖，所有要拆分
消除非主属性对码的传递依赖 码值学生编号-》班级-》班主任
非主属性班主任通过传递依赖到学生编号 

```



### 1.3.常见的数据库

+ 开源：MySQL、PostgreSQL（对商业比较友好）
+ 商业：Oracle，DB2，SQL Server
+ 内存数据库：Redis，VoltDB
+ 图数据库：Neo4j，Nebula
+ 时序数据库：InfluxDB、openTSDB
+ 其他关系数据库：Access、Sqlite、H2、Derby、Sybase、Infomix等
+ NoSQL数据库：MongoDB、Hbase、Cassandra、CouchDBNewSQL/
+ 分布式数据库：TiDB、CockroachDB、NuoDB、OpenGauss、OB、TDSQL

## 2.SQL语言

最早针对关系数据库实现的一种结构化查询语言，简单易学！1974年提出，在IBM某系统上实现！以及后面制定各种标准
SQL99 SQL97等

主要包含以下几部分

+ 数据查询语言（DQL）：select * from  
+ 数据操作语言DML : insert update delete
+ 事务控制语言TCL commit reollback savepoint
+ 数据控制语言DCL grant revoke赋权限
+ 数据定义语言DDL create alter drop
+ 指针控制语言CCL  DECLARE CUROSR

## 3.mysql数据库

oracle->sun->ab

收购完 分裂成俩个版本 mariaDB和mysql

4.0-支持事务-》5.7稳定使用最多的-》8.0最新和功能完善的版本

5.7支持：-多主-MGR高可用-分区表-json-性能-修复XA等

8.0 -通用表达式-窗口函数-持久化参数-自增列持久化-默认编码utf8mb4-DDL原子性-JSON增强-不再对group by进行隐式排序

```
通用表表达式：with cte as (select 1) select * from cte; 递归友好类似于 oracle pivot
```

```
MySQL5.7及其以前的版本，MySQL服务器重启，会重新扫描表的主键最大值，如果之前已经删除过id=100的数据，但是表中当前记录的最大值如果是99，那么经过扫描，下一条记录的id是100，而不是101。　　　　MySQL8.0则是每次在变化的时候，都会将自增计数器的最大值写入redo log,同时在每次检查点将其写入引擎私有的系统表。则不会出现自增主键重复的问题。
```

```
各种JSON函数增强
```

## 4.深入数据库原理

### 4.1MySQL架构图

![1615468464605](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615468464605.png)

1.连接器 各种连接方式 JDBC,ODBC.NET PHP

2.server层：连接池 -SQL接口层-》SQL解析-词法分析语法分析-》分析器（分析各种SQL执行操作）-》没命中则进入引擎层

3.存储引擎：插件式的方式

```
myIsam:不支持事务，但性能较高，对一致性要求不高。小网站论坛
innodb:支持事务
archive:归档数据库 数据进行压缩，不怎么查
memory:内存数据库
```

4.文件和日志：不同的存储引擎实现不同，例如myisam数据和索引分开放，innodb则放在一起

### 4.2Mysql存储(innodb)

+  日志组文件：ib_logfile0和ib_logfile1默认为5m
+ 表结构文件：*.frm
+ 表空间文件：*.ibd
+ 字符集和排序规则文件：  db.opt  
+  binlog 二进制日志文件：记录主数据库服务器的 DDL 和 DML 操作  
+   二进制日志索引文件：master-bin.index  
+   数据都在 ibdata


```
show global variables like "%datadir%" 查询数据文件地址
```

  共享模式 innodb_file_per_table=1  
      

```
共享表空间和独立表空间
共享表空间用于压缩，对insert操作比较友好，但不易迁移
独立表空间为每张表都新建一个表空间，容易迁移维护
```

### 4.3 mysql执行流程

![1615473561002](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615473561002.png)

SERVER层

1.连接器 连接池进行连接维护 2.进行缓存查询、用分析器进行词法分析语法分析3.优化器生成最优的执行计划4.交给执行器进行执行

INNODB引擎层

+ 写事前undolog用于事前数据恢复
+ 找到是否有所在记录存在于内存中，存在
  + 唯一索引：判断所在数据是否冲突，不冲突更新内存
  + 普通索引：直接更新
+ 不存在
  + 唯一索引：讲数据从磁盘读入内存中判断冲突与否，更新
  + 普通索引：在changer buffer更新记录（异步更新到磁盘中）
+ 进行redolog恢复 事后恢复
+ 写入binlog用于主从同步
+ 提交redolog到磁盘，提交binlog到磁盘（二阶段提交）

```
changer buffer: 进行非主键索引数据更新或插入时，不需要进行唯一性判断，因此直接在此更新，后台异步批量同步到磁盘，减少数据IO操作。如果经常进行数据写入，读比较少的列，可以考虑将此列设为普通索引,但如果写之后立马读，会触发merge过程，反而降低了性能
buffer pool:对数据和索引的缓存区，降低磁盘IO操作，采用LRU数据淘汰算法
二阶段提交：必须等redolog和binlog写入磁盘后才算事务完成，否则会出现日志不一致情况，例如先写入redolog 宕机进行主从同步会有问题，先写入binlog宕机，则进行数据恢复会和原来不一样
```

### 4.4 mysql 执行引擎和状态

![1615477354242](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615477354242.png)

innodb执行事务和行锁表锁，外键, mysisam存储空间大，且支持压缩

### 4.5mysql 执行顺序

![1615477586978](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615477586978.png)

### 4.6 mysql索引原理（innodb）

索引是为了提高数据插叙速度而产生的类似于书的目录

#### 4.6.1常见的索引模型

+ 哈希表：适合等值查询例如memcache NoSQL引擎，范围查询复杂度高。
+ 有序数组:适合等值和范围查询，但如果增删改数组，则移动成本太高。适合静态存储引擎。（就是数组插入增删的劣势）
+ B+树：N叉树，查询速度快，且增删改，易以维护

+ 数据按页分块 每页16k