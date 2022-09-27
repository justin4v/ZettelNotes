#Todo  #Generated-column #Mysql #JSON 

- 有时候需要对字段上加函数然后进行GROUP BY。使用执行分析，发现出现 Using temporary， 分组条件并没有走索引。
- mysql 5.7 的函数会导致索引失效。
- 可以通过添加一个冗余字段来保存函数的计算结果，然后添加索引，这时候的 GROUP BY 就会走索引了，但是操作不便且会增加存储空间。
- mysql 5.7 提供了一个新特性：虚拟列 **Generated columns**，可以使用虚拟列来方便的达到这个目的。

# 虚拟列

- 又叫做虚拟列或计算列。
- 生成列的值是在列定义时包含了一个计算表达式计算得到的，有两种类型的生成列：
	- VIRTUAL，即`虚拟`类型，字段值不实际存储，当读取行时再计算，虚拟列类型不占存储  
	- STORED，即`存储`类型，字段值会实际存储起来，当插入或更新时，字段值会计算出来并存储起来
	- Virtual生成列一般比 Stored 生成列更有用，因为它**不会占用存储空间**，也是 Mysql 默认类型。

1. 衍生列的定义可以修改，但virtual和stored之间不能相互转换，必要时需要删除重建  
2. 虚拟列字段只读，不支持 INSRET 和 UPDATE。  
3. 只能引用本表的非 generated column 字段，不可以引用其它表的字段。  
4. 使用的表达式和操作符必须是 Immutable 属性。  
5. 支持创建索引。  
6. 可以将已存在的普通列转化为stored类型的衍生列，但virtual类型不行；同样的，可以将stored类型的衍生列转化为普通列，但virtual类型的不行。  
7. MySQL可以在衍生列上面创建索引。对于stored类型的衍生列，跟普通列创建索引无区别。  
8. 对于virtual类型的衍生列，创建索引时，会将衍生列值物化到索引键里，即把衍生列的值计算出来，然后存放在索引里。如果衍生列上的索引起到了覆盖索引的作用，那么衍生列的值将直接从覆盖索引里读取，而不再依据衍生定义去计算。
9. 针对virtual类型的衍生列索引，在insert和update操作时会消耗额外的写负载，因为更新衍生列索引时需要将衍生列值计算出来，并物化到索引里。`但即使这样，virtual类型也比stored类型的衍生列好`，有索引就避免了每次读取数据行时都需要进行一次衍生计算，同时stored类型衍生列实际存储数据，使得聚簇索引更大更占空间。
10. virtual类型的衍生列索引使用 MVCC日志，避免在事务rollback或者purge操作时重新进行不必要的衍生计算。

## 用法
定义
```mysql
col_name data_type [GENERATED ALWAYS] AS (expr)
  [VIRTUAL | STORED] [NOT NULL | NULL]
  [UNIQUE [KEY]] [[PRIMARY] KEY]
  [COMMENT 'string']
```

如：
```sql
CREATE TABLE person (
  first_name VARCHAR(10) NOT NULL COMMENT '名',
  last_name VARCHAR(10) NOT NULL COMMENT '姓',
  full_name VARCHAR(21) GENERATED ALWAYS AS (CONCAT(first_name,' ',last_name)) STORED NOT NULL COMMENT '全名'
);
```

```sql
ALTER TABLE person ADD full_name_gc VARCHAR(21) 
GENERATED ALWAYS AS (CONCAT(first_name,'_',last_name)) VIRTUAL NOT NULL COMMENT '全名(虚拟列)'
```

# Index JSON
- Mysql 没有提供直接索引 JSON 类型数据的方法，但是 *Generate column* 可以做到类似的事情。
- 在 JSON 类型上新增虚拟列并建立索引，可以快速查询 JSON field。

## 示例
- 假设有一个 player 列表，JSON 格式，里面含有一些游戏信息
- 原始 JSON:
```json
{
    "id": 1,  
    "name": "Sally",  
    "games_played":{    
       "Battlefield": {
          "weapon": "sniper rifle",
          "rank": "Sergeant V",
          "level": 20
        },                                                                                                                          
       "Crazy Tennis": {
          "won": 4,
          "lost": 1
        },  
       "Puzzler": {
          "time": 7
        }
     }
 }
```

### 新建一个 players table
```sql
CREATE TABLE `players` (  
    `id` INT UNSIGNED NOT NULL,
    `player_and_games` JSON NOT NULL,
    PRIMARY KEY (`id`)
);
```

### 新增 Generate column 
- names_virtual 值从 JSON 中提取，采用 [[JSON path]] 语法提取值；
- 使用 Mysql 操作符 `->>`，等价于 `JSON_UNQUOTE(JSON_EXTRACT(...))`，得到无括号包围的实际值。

```sql
ALTER TABLE 'players' add column `names_virtual` VARCHAR(20) GENERATED ALWAYS AS (`player_and_games` ->> '$.name') not null；
```

### 新建索引
- Generate column 支持建立索引；

```sql
CREATE INDEX `names_idx` ON `players`(`names_virtual`);
```


# 使用场景
1. 虚拟生成列可以用作简化和统一查询的方法。可以将复杂条件定义为生成列，并从表上的多个查询中引用该条件，以确保所有条件都使用完全相同的条件。
2. 存储的生成列可用作复杂条件的物化缓存，这些条件需要快速计算。
3. 生成列可以模拟功能索引：使用生成列定义功能表达式并为其编制索引。这对于处理无法直接索引的类型的列（例如，JSON列）很有用 。





# 参考
1. [MySQL for JSON: Generated Columns and Indexing](https://www.compose.com/articles/mysql-for-json-generated-columns-and-indexing/#:~:text=MySQL%20for%20JSON%3A%20Generated%20Columns%20and%20Indexing%201,...%203%20Storing%20values%20in%20generated%20columns%20)