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

1. 可以解决脏读的问题。
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



# 总结




# 参考
1. [[事务与隔离级别]]
2. [[MVCC实现]]