#Mysql #Problem-and-Solutions 

# 死锁探测
- innodb 主动探知到死锁，会回滚了某一等待的事务。
- 探知死锁的方法：
	1. 两个事务相互等待时，当等待超过某一阀值时，对其中一个事务回滚，另一个事务若能继续执行，此时发生了死锁。
		- 在innodb中，参数innodb_lock_wait_timeout用来设置超时时间。
	2. `wait-for graph` 算法主动进行死锁检测，每当**加锁请求无法立即被满足需要并进入等待**时，wait-for graph 算法都会被触发。

## wait-for graph
 - innodb 将各个事务看为一个节点，资源就是各个事务占用的锁；
 - 当事务 1 需要等待事务 2 的锁时，就生成一条**有向边**从1指向2；
 - 最后行成一个**有向图**。
 - 只要检测**有向图是否出现环路**，出现环路就是死锁。
 
![[wait for graph检测死锁示意.png]]

# innodb隔离级别、索引与锁
## 索引
- 假设有一张*消息表（msg）*，里面有3个字段。
- 假设id是主键，token 是*非唯一索引*，message没有索引。

| id: bigint | token: varchar(30) | message: varchar(4096) |
| ---------- | ------------------ | ---------------------- |

- innodb 对主键使用了**聚簇索引**；
	- 一种数据存储方式；
	- 表数据是和主键一起存储；
	- 主键索引的叶结点存储行数据。
- 对于[[|二级索引]]，其叶子节点存储的是主键值

![[聚簇索引和普通索引示意.png]]

## 索引与锁的关系

1. delete from msg where id=2；
	- id 是主键，因此直接锁住整行记录即可。

![[主键索引与锁示意.png]]

2. delete from msg where token=’ cvs’;
	- 由于 token 是二级索引，因此首先锁住二级索引（两行）；
	- 接着会锁住相应主键所对应的记录；
![[二级索引与锁关系示意.png]]

3. delete from msg where message=订单号是多少’；
	- message 没有索引，所以走的是全表扫描过滤。
	- 表上的各个记录都将添加上X锁。
![[无索引与锁关系示意.png]]



# Mysql 锁错误
Mysql造成锁的情况有很多：

1.  执行DML操作没有commit，再执行删除操作就会锁表。
2.  在同一事务内先后对同一条数据进行插入和更新操作。
3.  表索引设计不当，导致数据库出现死锁。
4.  长事物，阻塞DDL，继而阻塞所有同表的后续操作。

`Lock wait timeout exceeded`与`Dead Lock`不一样。

-   `Lock wait timeout exceeded`：后提交的事务等待前面处理的事务释放锁，但是在等待的时候超过了mysql的锁等待时间，就会引发这个异常。
-   `Dead Lock`：两个事务互相等待对方释放相同资源的锁，从而造成的死循环，就会引发这个异常。


**`innodb_lock_wait_timeout`与`lock_wait_timeout`也是不一样的。

-   `innodb_lock_wait_timeout`：innodb的dml操作的行级锁的等待时间
-   `lock_wait_timeout`：数据结构ddl操作的锁的等待时间


# 锁超时
```java
java.sql.SQLTransaientConnectionException: Lock wait timeout exceeded;
...
SQL： DELETE FROM xxx WHERE 1=1 AND BOOK_ID IN (?,?,?,?)
```

## 原因
- **`BOOK_ID` 没有建立索引**；
- sql 执行时**锁了整张表**；
- 多线程执行时，其他线程等待时间超时。

查看超时时间设置：
-  `innodb_lock_wait_timeout`：innodb的dml操作的行级锁的等待时间
-  `SHOW VARIABLES LIKE 'innodb_lock_wait_timeout'

## 解决
- 在 `BOOK_ID` 字段建立索引，避免全表扫描。


# 死锁

```java
java.sql.SQLTransaientConnectionException: deadlock found when trying to get lock;
...
SQL： DELETE FROM xxx WHERE 1=1 AND BOOK_ID IN (?,?,?,?)
```

## 原因
- in 查询值过多

## 解决
- 使用短事务，分批次提交，类似操作系统时间片形式，提高并发性。



# 参考
1. [MySql Lock wait timeout exceeded该如何处理？](https://ningyu1.github.io/site/post/75-mysql-lock-wait-timeout-exceeded/)
2. [MySQL delete limit AND 逻辑删除优劣比较](https://blog.csdn.net/qq_21454973/article/details/109162528)
3. [mysql死锁（锁与事务）](https://www.cnblogs.com/111testing/p/11371236.html)
4. [[索引分类]]