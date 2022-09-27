#Todo  #Generated-column #Mysql #JSON 

- 有时候需要对字段上加函数然后进行GROUP BY。使用执行分析，发现出现 Using temporary， 分组条件并没有走索引。
- mysql 5.7 的函数会导致索引失效。
- 可以通过添加一个冗余字段来保存函数的计算结果，然后添加索引，这时候的 GROUP BY 就会走索引了，但是操作不便且会增加存储空间。
- mysql 5.7 提供了一个新特性：虚拟列 **Generated columns**，可以使用虚拟列来方便的达到这个目的。

# 虚拟列

- 又叫做虚拟列或计算列。
- 生成列的值是在列定义时包含了一个计算表达式计算得到的，有两种类型的生成列：
	- Virtual（虚拟）：这个类型的列会在读取表记录时自动计算此列的结果并返回。  
	- Stored（存储）：这个类型的列会在表中插入一条数据时自动计算对应的值，并插入到这个列中，列会作为一个常规列存在表中。
	- Virtual生成列一般比存储生成列更有用，因为它不会占用存储空间。

1、衍生列的定义可以修改，但virtual和stored之间不能相互转换，必要时需要删除重建  
2、虚拟列字段只读，不支持 INSRET 和 UPDATE。  
3、只能引用本表的非 generated column 字段，不可以引用其它表的字段。  
4、使用的表达式和操作符必须是 Immutable 属性。  
5、支持创建索引。  
6、可以将已存在的普通列转化为stored类型的衍生列，但virtual类型不行；同样的，可以将stored类型的衍生列转化为普通列，但virtual类型的不行。  
7、MySQL可以在衍生列上面创建索引。对于stored类型的衍生列，跟普通列创建索引无区别。  
8、对于virtual类型的衍生列，创建索引时，会将衍生列值物化到索引键里，即把衍生列的值计算出来，然后存放在索引里。如果衍生列上的索引起到了覆盖索引的作用，那么衍生列的值将直接从覆盖索引里读取，而不再依据衍生定义去计算。

9、针对virtual类型的衍生列索引，在insert和update操作时会消耗额外的写负载，因为更新衍生列索引时需要将衍生列值计算出来，并物化到索引里。`但即使这样，virtual类型也比stored类型的衍生列好`，有索引就避免了每次读取数据行时都需要进行一次衍生计算，同时stored类型衍生列实际存储数据，使得聚簇索引更大更占空间。

10、virtual类型的衍生列索引使用 MVCC日志，避免在事务rollback或者purge操作时重新进行不必要的衍生计算。

> 注意，出于性能的考虑，选择Virtual 而不是 Stored

# 参考
1. [MySQL for JSON: Generated Columns and Indexing](https://www.compose.com/articles/mysql-for-json-generated-columns-and-indexing/#:~:text=MySQL%20for%20JSON%3A%20Generated%20Columns%20and%20Indexing%201,...%203%20Storing%20values%20in%20generated%20columns%20)