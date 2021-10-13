# 概念
SQL 注入（SQL Injection）是一种数据库层的安全漏洞。
主要原因是程序对用户输入数据的**合法性没有判断和处理**，导致攻击者可以在事先定义好的 SQL 语句中添加额外的 SQL 语句，并实现非法操作。

# 注入类型
### 1. 恶意拼接查询

我们知道，SQL 语句可以查询、插入、更新和删除数据，且使用分号来分隔不同的命令。例如：
```sql
SELECT * FROM users WHERE user_id = $user_id
```

其中，user_id 是传入的参数，如果传入的参数值为“1234; DELETE FROM users”，那么最终的查询语句会变为：
```sql
SELECT * FROM users WHERE user_id = 1234; DELETE FROM users
```

如果以上语句执行，则会删除 users 表中的所有数据。  

### 2. 利用注释执行非法命令。

SQL 语句中可以插入注释。例如：  
```sql
SELECT COUNT(*) AS 'num' FROM game_score WHERE game_id=24411 AND version=$version
```

如果 version 包含了恶意的字符串`'-1' OR 3 AND SLEEP(500)--`，那么最终查询语句会变为：

```sql
SELECT COUNT(*) AS 'num' FROM game_score WHERE game_id=24411 AND version='-1' OR 3 AND SLEEP(500)--
```

以上恶意查询只是想耗尽系统资源，SLEEP(500) 将导致 SQL 语句一直运行。如果其中添加了修改、删除数据的恶意指令，那么将会造成更大的破坏。

### 3. 传入非法参数

SQL 语句中传入的字符串参数是用单引号引起来的，如果字符串本身包含单引号而没有被处理，那么可能会篡改原本 SQL 语句的作用。 例如：  
```sql
SELECT * FROM user_name WHERE user_name = $user_name
```

如果 user_name 传入参数值为 G'chen，那么最终的查询语句会变为：  
```sql
SELECT * FROM user_name WHERE user_name ='G'chen'
```

一般情况下，以上语句会执行出错，这样的语句风险比较小。虽然没有语法错误，但可能会恶意产生 SQL 语句，并且以一种你不期望的方式运行。

### 4. 添加额外条件

在 SQL 语句中添加一些额外条件，以此来改变执行行为。条件一般为真值表达式。例如：
```sql
UPDATE users SET userpass='$userpass' WHERE user_id=$user_id;
```

如果 user_id 被传入恶意的字符串“1234 OR TRUE”，那么最终的 SQL 语句会变为：
```sql
UPDATE users SET userpass= '123456' WHERE user_id=1234 OR TRUE;
```
这将更改所有用户的密码。

# 避免SQl注入
可以采取适当的预防措施来避免 SQL 注入

## 1. 过滤输入内容，校验字符串

过滤输入内容就是在数据提交到数据库之前，就把用户输入中的不合法字符剔除掉。
  
如果值属于特定的类型或有具体的格式，那么在拼接 SQL 语句之前就要进行校验，验证其有效性。

## 2. 参数化查询

参数化查询目前被视作是预防 SQL 注入攻击最有效的方法。
 
  
MySQL 以“?” 符：
```sql
UPDATE myTable SET c1 = ?, c2 = ?, c3 = ? WHERE c4 = ?
```

在使用参数化查询的情况下，数据库服务器不会将参数的内容视为 SQL 语句的一部分来进行处理，而是在数据库完成 SQL 语句的编译之后，才套用参数运行。因此就算参数中含有破坏性的指令，也不会被数据库所运行。  

## 3. 安全测试、安全审计

除了开发规范，还需要合适的工具来确保代码的安全。我们应该在开发过程中应对代码进行审查，在测试环节使用工具进行扫描，上线后定期扫描安全漏洞。通过多个环节的检查，一般是可以避免 SQL 注入的。  
  
有些人认为存储过程可以避免 SQL 注入，存储过程在传统行业里用得比较多，对于权限的控制是有一定用处的，但如果存储过程用到了动态查询，拼接 SQL，一样会存在安全隐患。