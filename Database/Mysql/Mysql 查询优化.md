#SQL #Mysql #Optimize

# 架构和查询过程
![[Mysql架构示意.png]]

- 最上一层为客户端，处理：*连接处理、授权认证、安全*等功能。
- 中间层为 MySQL 核心服务：
	- 包括查询*解析、分析、优化、缓存、内置函数*。
	- *跨存储引擎*的功能也在这一层实现：存储过程、触发器、视图等。
- 最下一层为*存储引擎*，负责MySQL中的*数据存储和提取*

## 查询过程
![[mysql查询过程示意.png]]

## 客户端/服务端通信
- 类似“半双工”通信方式；
- 同一时刻，要么客户端向服务端发送数据，要么反过来。不能同时进行。
- 实际开发中，尽量保持查询简单且只返回必需的数据，查询中尽量 *避免使用 SELECT \* 以及加上 LIMIT 限制*。


## 查询缓存
- MySQL将结果缓存在一个引用表（类似于 HashMap）中，通过哈希值索引；
- 哈希值和查询本身、当前要查询的数据库、客户端协议版本号等一些可能影响结果的信息有关；
- 非确定性功能将导致查询不被缓存（包括临时表，用户变量，RAND（），NOW（）和UDF）。
- MySQL的查询缓存跟踪缓存中涉及的每个表，如果表的数据或结构发生变化，和表相关的所有缓存数据都将失效。
- 任何的写操作，对应表的所有缓存都失效。
- 如果查询缓存非常大或者碎片很多，缓存就可能带来很大的系统消耗。
- 不要轻易打开查询缓存，特别是写密集型应用。
- [Mysql 8.0 版本查询缓存将被移除](https://dev.mysql.com/blog-archive/mysql-8-0-retiring-support-for-the-query-cache/) ：严重的可伸缩性问题（scalability issues），且很容易成为瓶颈（bottleneck）。
- 如果需要使用查询缓存，可参考 [proxysql: High-performance MySQL proxy with a GPL license](https://github.com/sysown/proxysql)
- *越靠近用户*时，缓存会带来最大的好处。

## 语法解析和预处理
- 解析 SQL 语句，并生成解析树。
- 验证语法。如是否使用错误的关键字等等。
- 预处理进一步检查解析树是否合法。如要查询的数据表和数据列是否存在等等。


## 查询优化
- 优化器将语法树转化成查询计划。
- 多数情况下，一条查询有多种执行方式；
- MySQ L使用基于成本的优化器，查询当前会话的 `last_query_cost` 得到当前查询的成本。

```sql
show status like 'last_query_cost';
```


# 查询优化
## 分析SQL执行频率
例如：分析读为主，还是写为主
```sql
show [global|session] status;
```

## 定位效率低的SQL

```sql
-- 慢查询日志定位
-log-slow-queries = xxx;（指定文件名）

-- 查看当前正在进行的线程，包括线程状态、是否锁表
SHOW PROCESSLIST;
```

## 分析SQL执行计划
```sql
explain "your sql"
desc "your sql"
```

### explain 结果分析
```sql
mysql> explain extended select * from film where id = 1;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | film  | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+----------+-------+
```
- id：select 查询的序列号
- select_type
	-  SIMPLE 简单表，不使用表连接或子查询
	- PRIMARY 主查询，复杂查询中最外层的 select
	- **subquery**：包含在 select 中的子查询（不在 from 子句中）
	- **derived**：包含在 from 子句中的子查询。MySQL会将结果存放在一个临时表中，也称为派生表（derived）
	- **union**：在 union 中的第二个和随后的 select
	- **union result**：从 union 临时表检索结果的 select

- type:
	- ALL 全表扫描
	- index 索引全扫描
	- range 索引范围扫描
	- ref 使用非唯一索引或唯一索引的前缀扫描
	- eq_ref 类似 ref，使用的索引是唯一索引
	- const/system 单表中最多有一个匹配行
	- NULL 不用访问表或者索引，直接得到结果
  
## show profile 分析 SQL

```sql
-- 是否支持
select @@have_profiling 

--是否开启
select @@profiling 

-- 执行 sql

show profiles; 

show profile block io for QUERY 17;
```

## 索引优化
- 频繁作为查询条件的字段应该建立索引；
- 唯一性叫差的字段不适合作为唯一索引，因为能过滤的部分占比较小；
- 更新频繁的字段不适合建立索引，因为更新数据时也需要更新索引；
- 不作为查询条件的字段不要建立索引。
- Mysql 查询经过优化器优化后，*一般每次都只使用一个索引*：
	- 经常联合查询的条件，应该建立联合索引；
	- 有多个单列索引而只使用一个，是因为多数情况下，*多次扫描多个索引的成本*比只扫描一个，然后全表扫描成本高。

## 一般建议
- 从Explain和Profile出发；
- 用小结果集驱动大的结果集（如join中左表为*驱动表*）；
- 在索引中完成排序；
- select 最小的 Columns；
- 尽早过滤，并使用最有效的过滤条件；
- 避免复杂的JOIN和子查询。

## 常用查询
- 尽量避免where使用 != 或 <>  
- 尽量避免 where 子句用 or 连接：
	- or 连接的所有字段都有索引，索引才会生效，否则*索引失效*。
- 乱用%导致全表扫描  
- 尽量避免where子句对字段进行表达式操作  
- 尽量避免where子句对字段进行函数操作  
- 覆盖查询，返回需要的字段  
- 优化嵌套查询，关联查询优于子查询  
- 用exist代替in  
- 避免使用 '\*' , 只返回需要的字段



# 参考
1. [MySQL进阶篇SQL优化（show status、explain分析）](https://www.cnblogs.com/wzk153/p/14536323.html)
2. [[Index]]
3. [[索引的最左前缀原则]]