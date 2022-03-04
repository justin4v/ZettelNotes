
# 设置隔离级别

1. 查看当前隔离级别
```sql
# 查看事务隔离级别 5.7.20 之前
show variables like 'transaction_isolation';
SELECT @@transaction_isolation

# 5.7.20 之后
SELECT @@tx_isolation
show variables like 'tx_isolation'

# 结果
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+
```

2. 修改隔离级别
```sql
set [作用域] transaction isolation level [事务隔离级别]

SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}。
```
- 作用域是 SESSION 或者 GLOBAL：GLOBAL 是全局的，而 SESSION 只针对当前回话窗口。
- 如设置全局隔离级别为读提交级别。
```sql
mysql> set global transaction isolation level read committed; 
```



# MySQL 中执行事务

事务的执行过程：
1. 以 begin 或者 start transaction 开始事务语句；
2. 最后要执行 commit 或 rollback 操作，事务结束；
3. 事务*真正开始于 begin 命令之后的第一条语句*。

# 隔离级别的实现
1. MySQL 事务隔离**依靠锁实现**；

用一张表验证事务，表结构如下：

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(30) DEFAULT NULL,
  `age` tinyint(4) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8
```

初始只有一条记录

```shell
mysql> SELECT * FROM user;
+----+-----------------+------+
| id | name            | age  |
+----+-----------------+------+
|  1 | 古时的风筝       |  1   |
+----+-----------------+------+
```

## 读未提交
- 任何事务*对数据的修改都会暴露*给其他事务，即便事务还没有提交。
- 会出现*脏读*问题；
- 读未提交过程示意
![img](读未提交.png)

## 读提交

1. 一个事务*只能读到已经提交过的数据*，可以解决脏读的问题。
2. 读提交事务隔离级别是大多数流行数据库的默认事务隔离界别，比如 Oracle，但是不是 MySQL 的默认隔离界别。
3. 读提交示意
![img](读提交.png)


## 可重复读
- 事务*不会读到其他事务对原有数据*的修改，即使其他事务已提交；
- 事务*开始时读到的数据和在事务提交前的任意时刻读到数据是一样的*。
- 但对于其他事务**新插入**的数据还是可以读到的，这就是**幻读**问题。
- *MySQL 的可重复读解决了幻读问题*。

![[可重复读.png]]


## 幻读


![[幻读.png]]


## 串行化

- 串行化解决了脏读、可重复读、幻读的问题，但是效率最差；
- 它将事务的执行变为顺序执行，相当于**单线程**，后一个事务的执行必须等待前一个事务结束。

# MySQL事务隔离实现

1. *读未提交*，不加锁，可以理解为没有隔离。
2. *串行化*：
	1. 读的时候加**共享锁**，其他事务可以并发读，但是*不能加写锁*。
	2. 写的时候加**独占锁**，其他事务*不能写也不能读*。
3. 读提交和可重复读。这两种隔离级允许一定的并发，又要解决问题。

## 实现可重复读
MySQL 采用了 **MVCC (多版本并发控制，Multi-Version Concurrency Control)** 的方式实现可重复读。

### MVCC
1. 数据库表中一行记录实际上有**多个版本**；
2. 每个版本的记录除了有数据本身外，还要有一个**表示版本的字段row trx_id**，是对应的**事务 id**；
3. 事务 ID 记为 transaction id，它在事务开始的时候向事务系统申请，按时间先后顺序递增。

![img](MVCC示意.png)
上图中一行记录现有 3 个版本，每一个版本都记录使其产生的事务 ID，比如事务A的transaction id 是100，那么版本1的row trx_id 就是 100，同理版本2和版本3。


# 总结
- MySQL 的 InnoDB 引擎才支持事务，其中**可重复读**是默认的隔离级别；
- 读未提交和串行化基本上是不需要考虑的隔离级别，前者不加锁限制，后者相当于单线程执行；
- 读提交解决了脏读问题；
- **行锁**解决了并发更新的问题。
-  MySQL 在可重复读级别解决了幻读问题，是通过行锁和间隙锁的组合 **Next-Key 锁**实现的。