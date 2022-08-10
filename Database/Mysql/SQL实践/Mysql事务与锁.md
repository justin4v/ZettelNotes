#Mysql #Problem-and-Solutions 

# 死锁探测
- innodb 主动探知到死锁，会回滚了某一等待的事务。
- 探知死锁的方法：
	1. 两个事务相互等待时，当一个等待时间超过设置的某一阀值时，对其中一个事务进行回滚，另一个事务就能继续执行。这种方法简单有效，在innodb中，参数innodb_lock_wait_timeout用来设置超时时间。

仅用上述方法来检测死锁太过被动，innodb还提供了wait-for graph算法来主动进行死锁检测，每当加锁请求无法立即满足需要并进入等待时，wait-for graph算法都会被触发。

### 1.2 wait-for graph原理

  我们怎么知道上图中四辆车是死锁的？他们相互等待对方的资源，而且形成环路！我们将每辆车看为一个节点，当节点1需要等待节点2的资源时，就生成一条有向边指向节点2，最后形成一个有向图。我们只要检测这个有向图是否出现环路即可，出现环路就是死锁！这就是wait-for graph算法。


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