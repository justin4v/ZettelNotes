#Mysql #Optimize 

# explain 
## id列
- select 的序列号；、
- id 按 select 出现的顺序增长。
- select 分为简单查询和复杂查询。
	- 复杂查询分为：简单子查询、派生表（from语句中的子查询）、union 查询。

1）简单子查询
```sql
mysql> explain select (select 1 from actor limit 1) from film;
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
| id | select_type | table | type  | possible_keys | key      | key_len | ref  | rows | Extra       |
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
|  1 | PRIMARY     | film  | index | NULL          | idx_name | 32      | NULL |    1 | Using index |
|  2 | SUBQUERY    | actor | index | NULL          | PRIMARY  | 4       | NULL |    2 | Using index |
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+ 
```

2）from子句中的子查询
```sql
mysql> explain select id from (select id from film) as der;
+----+-------------+------------+-------+---------------+----------+---------+------+------+-------------+
| id | select_type | table      | type  | possible_keys | key      | key_len | ref  | rows | Extra       |
+----+-------------+------------+-------+---------------+----------+---------+------+------+-------------+
|  1 | PRIMARY     | <derived2> | ALL   | NULL          | NULL     | NULL    | NULL |    2 | NULL        |
|  2 | DERIVED     | film       | index | NULL          | idx_name | 32      | NULL |    1 | Using index |
+----+-------------+------------+-------+---------------+----------+---------+------+------+-------------+
```

3）union查询
```sql
mysql> explain select 1 union all select 1;
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
| id | select_type  | table      | type | possible_keys | key  | key_len | ref  | rows | Extra           |
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
|  1 | PRIMARY      | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | No tables used  |
|  2 | UNION        | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | No tables used  |
| NULL | UNION RESULT | <union1,2> | ALL  | NULL          | NULL | NULL    | NULL | NULL | Using temporary |
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
```
- union 结果放在一个*匿名临时表*中，不在SQL中出现，id 是NULL。

### select_type列
- *查询种类*：简单还是复杂查询

1）**simple**：简单查询。查询不包含子查询和union
```sql
mysql> explain select * from film where id = 2;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | film  | const | PRIMARY       | PRIMARY | 4       | const |    1 | NULL  |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
```
2）**primary**：复杂查询中最外层的 select

3）**subquery**：包含在 select 中的子查询（不在 from 子句中）

4）**derived**：包含在 from 子句中的子查询。MySQL会将结果存放在一个临时表中，也称为派生表（derived）

```sql
mysql> explain select (select 1 from actor where id = 1) from (select * from film where id = 1) der;
+----+-------------+------------+--------+---------------+---------+---------+-------+------+-------------+
| id | select_type | table      | type   | possible_keys | key     | key_len | ref   | rows | Extra       |
+----+-------------+------------+--------+---------------+---------+---------+-------+------+-------------+
|  1 | PRIMARY     | <derived3> | system | NULL          | NULL    | NULL    | NULL  |    1 | NULL        |
|  3 | DERIVED     | film       | const  | PRIMARY       | PRIMARY | 4       | const |    1 | NULL        |
|  2 | SUBQUERY    | actor      | const  | PRIMARY       | PRIMARY | 4       | const |    1 | Using index |
+----+-------------+------------+--------+---------------+---------+---------+-------+------+------------+ 
```
5）**union**：在 union 中的第二个和随后的 select

6）**union result**：从 union 临时表检索结果的 select

 union 和 union result 示例：
```sql
mysql> explain select 1 union all select 1;
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
| id | select_type  | table      | type | possible_keys | key  | key_len | ref  | rows | Extra           |
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
|  1 | PRIMARY      | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | No tables used  |
|  2 | UNION        | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL | No tables used  |
| NULL | UNION RESULT | <union1,2> | ALL  | NULL        | NULL | NULL    | NULL | NULL | Using temporary |
+----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
```

## table列
- 查询使用的表。

## type 列
- 表示关联类型或访问类型，即 MySQL 决定*如何查找表中的行*。
- 性能顺序：`system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL`

1. **NULL**：能在优化阶段分解查询语句，在执行阶段不用访问表或索引。例如：在索引列中选取最小值，可以单独查找索引来完成。
```sql
mysql> explain select min(id) from film;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                        |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Select tables optimized away |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------+
```

2. **`const, system`**：能优化并将其转化成一个常量（可以看 show warnings 的结果）。
	1. primary key 或 unique key 与常数比较时，表最多有一个匹配行，读取1次，速度比较快。
```sql
mysql> explain extended select * from (select * from film where id = 1) tmp;
+----+-------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table      | type   | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
|  1 | PRIMARY     | <derived2> | system | NULL          | NULL    | NULL    | NULL  |    1 |   100.00 | NULL  |
|  2 | DERIVED     | film       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+

mysql> show warnings;
+-------+------+---------------------------------------------------------------+
| Level | Code | Message                                                       |
+-------+------+---------------------------------------------------------------+
| Note  | 1003 | /* select#1 */ select '1' AS `id`,'film1' AS `name` from dual |
+-------+------+---------------------------------------------------------------+
```
3. **eq_ref**：primary key 或 unique key 索引的所有部分被连接使用 ，最多只会返回一条符合条件的记录。这可能是在 const 之外最好的联接类型了。

```sql
mysql> explain select * from film_actor left join film on film_actor.film_id = film.id;
+----+-------------+------------+--------+---------------+--------+---------+----+------+-------+
| id | select_type | table  | type   | possible_keys | key   | key_len | ref   | rows | Extra  |
+----+-------------+------------+--------+---------------+---------+---------+-----+------+--------+
|  1 | SIMPLE   | film_actor | index  | NULL | idx_film_actor_id | 8   | NULL |  3 | Using index |
|  1 | SIMPLE  | film   | eq_ref | PRIMARY   | PRIMARY  | 4  | test.film_actor.film_id |  1 | NULL  |
+----+-------------+------------+--------+---------------+-------------------+---------+---------
```

4. **`ref`**
	1. 相比 `eq_ref`，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀；
	2. 索引要和某个值相比较，可能会找到多个符合条件的行。
```sql
1. 简单 select 查询，name是普通索引（非唯一索引）
mysql> explain select * from film where name = "film1";
+----+-------------+-------+------+---------------+----------+---------+-------+------+-----------------+
| id | select_type | table | type | possible_keys | key      | key_len | ref   | rows | Extra                    |
+----+-------------+-------+------+---------------+----------+---------+-------+------+---------------+
|  1 | SIMPLE      | film  | ref  | idx_name      | idx_name | 33      | const |    1 | Using where; Using index |
+----+-------------+-------+------+---------------+----------+---------+-------+------+-------------+

2.关联表查询，idx_film_actor_id是film_id和actor_id的联合索引，这里使用到了film_actor的左边前缀film_id部分。
```sql
mysql> explain select * from film left join film_actor on film.id = film_actor.film_id;
+----+-------------+------------+-------+------------+------------+---------+--------+----+-------------+
| id | select_type | table      | type  | possible_keys     | key               | key_len | ref          | rows | Extra       |
+----+-------------+------------+-------+----------+------------+---------+--------+------+-------------+
|  1 | SIMPLE      | film       | index | NULL              | idx_name          | 33      | NULL         |    3 | Using index |
|  1 | SIMPLE      | film_actor | ref   | idx_film_actor_id | idx_film_actor_id | 4       | test.film.id |    1 | Using index |
+----+-------------+------------+-------+-----------+-----------+---------+--------+--+-------------+
```

5. **`ref_or_null`**：类似`ref`，但是可以搜索值为 NULL 的行。

```sql
mysql> explain select * from film where name = "film1" or name is null;
+----+-------------+-------+-------------+---------------+----------+---------+-------+------+--------------------------+
| id | select_type | table | type        | possible_keys | key      | key_len | ref   | rows | Extra                    |
+----+-------------+-------+-------------+---------------+----------+---------+-------+------+--------------------------+
|  1 | SIMPLE      | film  | ref_or_null | idx_name      | idx_name | 33      | const |    2 | Using where; Using index |
+----+-------------+-------+-------------+---------------+----------+---------+-------+------+--------------------------+
```

6. **`index_merge`**：表示使用了*索引合并的优化*方法。

例如下表：id是主键，tenant_id是普通索引。or 的时候没有用 primary key，而是使用了 primary key(id) 和 tenant_id 索引。

```sql
mysql> explain select * from role where id = 11011 or tenant_id = 8888;
+----+-------------+-------+-------------+-----------------------+-----------------------+---------+------+------+-------------------------------------------------+
| id | select_type | table | type        | possible_keys         | key                   | key_len | ref  | rows | Extra                                           |
+----+-------------+-------+-------------+-----------------------+-----------------------+---------+------+------+-------------------------------------------------+
|  1 | SIMPLE      | role  | index_merge | PRIMARY,idx_tenant_id | PRIMARY,idx_tenant_id | 4,4     | NULL |  134 | Using union(PRIMARY,idx_tenant_id); Using where |
+----+-------------+-------+-------------+-----------------------+-----------------------+---------+------+------+-------------------------------------------------+
```

7. **`range`**：范围扫描通常出现在 in(), between ,> ,<, >= 等操作中。

```sql
mysql> explain select * from actor where id > 1;
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
|  1 | SIMPLE      | actor | range | PRIMARY       | PRIMARY | 4       | NULL |    2 | Using where |
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
```

8. **`index`**：和ALL一样，不同就是*只需扫描索引树*，这通常比ALL快一些。

```sql
mysql> explain select count(*) from film;
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
| id | select_type | table | type  | possible_keys | key      | key_len | ref  | rows | Extra       |
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
|  1 | SIMPLE      | film  | index | NULL          | idx_name | 33      | NULL |    3 | Using index |
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
```

9. `ALL：` *全表扫描*，意味着mysql需要从头到尾去查找所需要的行。通常情况下这需要增加索引来进行优化了

```sql
mysql> explain select * from actor;
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | actor | ALL  | NULL          | NULL | NULL    | NULL |    2 | NULL  |
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
```

### 5. possible_keys列

- *可能使用哪些索引*查找。
- 如果 possible_keys 有值，而 key 显示 NULL 的情况：表中数据不多，mysql 认为索引对查询帮助不大，选择了全表查询。
- 如果该列是 NULL，*表示没有相关的索引*。可以通过检查 where 子句看是否可以*创造一个适当的索引*提高查询性能。

### 6. key列
- 实际采用哪个索引。
- 如果没有使用索引，则该列是 NULL。
- 如果想强制mysql使用或忽视 possible_keys 列中的索引，使用 *force index、ignore index*。

### 7. key_len列

- 显示索引里使用的字节数，可以*算出具体使用了索引中的哪些列*。

举例来说：
- film_actor 的联合索引 idx_film_actor_id 由 film_id 和 actor_id 两个int列组成，并且每个是 int 4字节。
- 通过结果中的 *key_len=4 可推断出查询使用了第一个列：film_id* 列来执行索引查找。

```sql
mysql> explain select * from film_actor where film_id = 2;
+----+-------------+------------+------+-------------------+-------------------+---------+-------+------+-------------+
| id | select_type | table      | type | possible_keys     | key               | key_len | ref   | rows | Extra       |
+----+-------------+------------+------+-------------------+-------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | film_actor | ref  | idx_film_actor_id | idx_film_actor_id | 4       | const |    1 | Using index |
+----+-------------+------------+------+-------------------+-------------------+---------+-------+------+-------------+
```

key_len计算规则如下：
-   字符串
    -   char(n)：n字节长度
    -   varchar(n)：2字节存储字符串长度，如果是utf-8，则长度 3n + 2
-   数值类型
    -   tinyint：1字节
    -   smallint：2字节
    -   int：4字节
    -   bigint：8字节　　
-   时间类型　
    -   date：3字节
    -   timestamp：4字节
    -   datetime：8字节
-   如果字段允许为 NULL，需要1字节记录是否为 NULL

索引*最大长度是768字节*，当字符串过长时，mysql 会做一个类似左前缀索引的处理，将前半部分的字符提取出来做索引。

### 8. ref列

- 显示了在key列记录的索引中，表查找值所用到的列或常量，常见的有：const（常量），func，NULL，字段名（例：film.id）

### 9. rows列

- mysql估计要读取并检测的行数，注意这个不是结果集里的行数。

### 10. Extra列

这一列展示的是额外信息。常见的重要值如下：

**`distinct`**: 一旦 mysql 找到了与行相联合匹配的行，就不再搜索了

```sql
mysql> explain select distinct name from film left join film_actor on film.id = film_actor.film_id;
+----+-------------+------------+-------+-------------------+-------------------+---------+--------------+------+------------------------------+
| id | select_type | table      | type  | possible_keys     | key               | key_len | ref          | rows | Extra                        |
+----+-------------+------------+-------+-------------------+-------------------+---------+--------------+------+------------------------------+
|  1 | SIMPLE      | film       | index | idx_name          | idx_name          | 33      | NULL         |    3 | Using index; Using temporary |
|  1 | SIMPLE      | film_actor | ref   | idx_film_actor_id | idx_film_actor_id | 4       | test.film.id |    1 | Using index; Distinct        |
+----+-------------+------------+-------+-------------------+-------------------+---------+--------------+------+------------------------------+
```

**`Using index`**：返回的列数据只使用了索引中的信息，而没有再去访问表中的行记录。
- 性能高的表现。

```sql
mysql> explain select id from film order by id;
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
|  1 | SIMPLE      | film  | index | NULL          | PRIMARY | 4       | NULL |    3 | Using index |
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+ 
```

**`Using where`**：mysql 服务器将在*存储引擎检索行后再进行过滤*。
- 先读取整行数据，再按 where 条件进行检查，符合就留下，不符合就丢弃。

```sql
mysql> explain select * from film where id > 1;
+----+-------------+-------+-------+---------------+----------+---------+------+------+--------------------------+
| id | select_type | table | type  | possible_keys | key      | key_len | ref  | rows | Extra                    |
+----+-------------+-------+-------+---------------+----------+---------+------+------+--------------------------+
|  1 | SIMPLE      | film  | index | PRIMARY       | idx_name | 33      | NULL |    3 | Using where; Using index |
+----+-------------+-------+-------+---------------+----------+---------+------+------+--------------------------+
```

**`Using temporary`**：mysql *需要创建一张临时表来处理查询*。
- 一般是要进行优化的，首先是想到用索引来优化。

```sql
1. actor.name没有索引，此时创建了张临时表来distinct
mysql> explain select distinct name from actor;
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra           |
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------+
|  1 | SIMPLE      | actor | ALL  | NULL          | NULL | NULL    | NULL |    2 | Using temporary |
+----+-------------+-------+------+---------------+------+---------+------+------+-----------------+

2. film.name建立了idx_name索引，此时查询时extra是using index,没有用临时表
```sql
mysql> explain select distinct name from film;
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
| id | select_type | table | type  | possible_keys | key      | key_len | ref  | rows | Extra       |
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
|  1 | SIMPLE      | film  | index | idx_name      | idx_name | 33      | NULL |    3 | Using index |
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
```

**`Using filesort`**：mysql 会对结果*使用一个外部索引排序*，而不是按索引次序从表里读取行。
- mysql 会根据联接类型浏览所有符合条件的记录，并保存排序关键字和行指针，然后排序关键字并按顺序检索行信息。
- 一般也是要考虑使用索引来优化的。

```sql
1. actor.name未创建索引，会浏览actor整个表，保存排序关键字name和对应的id，然后排序name并检索行记录
mysql> explain select * from actor order by name;
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra          |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
|  1 | SIMPLE      | actor | ALL  | NULL          | NULL | NULL    | NULL |    2 | Using filesort |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+

2. film.name建立了idx_name索引,此时查询时extra是using index

```sql
mysql> explain select * from film order by name;
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
| id | select_type | table | type  | possible_keys | key      | key_len | ref  | rows | Extra       |
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
|  1 | SIMPLE      | film  | index | NULL          | idx_name | 33      | NULL |    3 | Using index |
+----+-------------+-------+-------+---------------+----------+---------+------+------+-------------+
```



# 参考
1. [MySQL进阶篇SQL优化（show status、explain分析）](https://www.cnblogs.com/wzk153/p/14536323.html)
2. [mysql explain详解](https://cloud.tencent.com/developer/article/1093229)