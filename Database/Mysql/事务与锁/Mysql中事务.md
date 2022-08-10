#Mysql #Transaction 


# 隔离级别图示说明

有一张表结构如下
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

# 总结


# 执行事务

事务的执行过程：
1. 以 begin 或者 start transaction 开始事务语句；
2. 最后要执行 commit 或 rollback 操作，事务结束；
3. 事务*真正开始于 begin 命令之后的第一条语句*。

# 参考
1. [[事务与隔离级别]]
2. [[MVCC实现]]