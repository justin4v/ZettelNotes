# 简介

- MVCC 使得*数据库读不会对数据加锁*，*普通的 SELECT 请求不会加锁*，提高了数据库的并发处理能力。
- MVCC只在 *REPEATABLE READ* 和 *READ COMMITIED* 两个隔离级别下工作。
- READ UNCOMMITIED *总是读取最新的数据行*，而不是符合当前事务版本的数据行。
- 而 SERIALIZABLE 则*会对所有读取的行都加锁*。


# InnoDB MVCC 实现
Mysql 实现依赖的是 
- *undo log* 
- *read view* 
- 每行数据的两个隐藏列：*事务 ID（DB_TRX_ID，递增）*，*回滚指针（DB_ROLL_PT）*
- 如果没有设置主键还会多添加： *主键（ROW_ID）*
- 查询时需要用*当前事务id和行记录的事务id进行比较*。

保存这两个额外事务编号，使大多数读操作都可以不用加锁。
使得读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行。不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作，以及一些额外的维护工作。

## undo log
undo log分为：**insert undo log** 和 **update undo log**
### insert undo log
- insert 操作中产生的 undo log；
- insert 操作记录只对当前事务本身可见，对于其他事务不可见；
- insert undo log 在事务提交后直接删除而不需要进行 purge 操作。

> purge的主要任务是将数据库中已经 mark del 的数据删除，另外也会批量回收undo pages

数据库 Insert时的数据初始状态：
![[InnoDB Insert的MVCC状态.png]]

### update undo log

- update 或 delete 操作中产生的 undo log。
- 会对已经存在的记录产生影响，因此 update undo log 不能在事务提交时就进行删除；
- 将事务提交时放到入 history list 上，等待 purge 线程进行最后的删除操作。

数据第一次被修改时
![[Update log第一次修改.png]]

## ReadView
 - MVCC 中**RC(READ COMMITTED)** 和 **RR(REPEATABLE READ)** 两种隔离级别的*核心是判断所有版本中哪个版本是当前事务可见*。
 - InnoDB 设计了 **ReadView** 的概念：
	 -  **ReadView** 主要包含*当前系统中有哪些活跃的读写事务*；
	 - 将它们的事务 id 放到一个列表**m_ids**。

### 版本可见判断逻辑
查询时的版本链数据是否看见的判断逻辑：
- *被访问版本数据的 trx_id 属性值小于 m_ids 列表中最小的事务 id*（最早的事务）：
	- 生成该版本数据的事务在 ReadView 生成前已经提交，所以该版本可以被当前事务访问。
- *被访问版本数据的 trx_id 属性值大于 m_ids 列表中最大的事务id*（最晚的事务）：
	- 生成该版本的事务在 ReadView 生成后才生成，所以该版本不可以被当前事务访问。
- *被访问版本数据的 trx_id 属性值在 m_ids 列表中最大的事务id和最小事务id之间*：
	- *trx_id 在 m_ids 列表中*，创建 ReadView 时生成该版本数据的事务还是*活跃*的，该版本不可以被访问；
	- *trx_id 不在 m_ids 列表中*，说明创建 ReadView 时生成该版本数据的事务已经*被提交*，该版本可以被访问

# RC和RR
- RC 级别的事务在每次查询开始时都会生成一个独立的 ReadView
- RR 级别的事务只在在事务开始后第一次读取数据时生成一个 ReadView