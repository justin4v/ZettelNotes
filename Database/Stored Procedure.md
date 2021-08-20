## 存储过程
**KEYS**
- SQL语句集合
- 存储在Metadata Repository/Data Dictionary
- [[RDBMS]]
- 可带有parameter

**DESCRIPTIONS**
Stored Procedure(**storp,StoPro,sproc**)是**一组SQL语句集合**，存储在[[#Data Dictionary]]/Metadata Repository，以整体为单位在关系型数据库（[[RDBMS]]）中执行。类似于Java中Method(方法)，或者C语言中的Function(函数)，以一个Procedure Name标识。
参考[[#Example]]。

### 优缺点
优点
1. 运行速度更快。执行一次后会保存，下次直接调用
2. 可重复使用。
缺点：
1. 面向过程。难以处理复杂逻辑。
2. 


### Data Dictionary
data dictionary 或者 metadata repository 是指数据库的元数据(metadata)存储库，存储如relationships、origin、format等数据。这个定义于 *IBM Dictionary of Computing* 。

### Example

```SQl
---------------  
-- 增加字段的一般存储过程  
--------------  
-- 修改分割符为 //
DELIMITER //  
-- 创建存储过程前先检查是否存在，存在就删除  
DROP PROCEDURE IF EXISTS add_column //  
-- 存储过程  
CREATE PROCEDURE add_column(IN dbName CHAR(50),IN tabName CHAR(50),IN colName CHAR(50),IN altStr CHAR(200))  
BEGIN  
	 DECLARE flag INT DEFAULT 0;  
		-- 创建一个变量用来存储遍历过程中的表名  
	 DECLARE tablename varchar(255);  
		-- 查询出需要遍历的表名列表  
	 DECLARE table_list CURSOR FOR select table_name from information_schema.tables where table_schema=dbName;  
		-- 查询是否有下一个数据，没有将标识设为1，相当于hasNext  
	 DECLARE CONTINUE HANDLER FOR NOT FOUND SET flag = 1;  
	   CASE  
	 -- 如果传入的表名为空，则在库中所有表上增加字段  
	 WHEN ISNULL(tabName) THEN  
	 -- 打开游标  
	 OPEN table_list ;  
			-- 取值设置到临时变量中，我这里就一个字段  
	 -- 注意：多个字段取值，变量名不要和返回的列名同名，变量顺序要和sql结果列的顺序一致  
	 FETCH table_list INTO tablename ;  
            REPEAT  
                 IF NOT EXISTS (select * from information_schema.columns  
                 where table_schema=dbName  
                 and table_name=tablename  
                 and column_name=colName)  
                 THEN  
 					-- 拼接修改表结构的语句  
					SET @exesql = CONCAT('ALTER TABLE ',tablename,' ADD COLUMN ',colName,' ',altStr);  
                    PREPARE sql1 FROM @exesql;  
                    -- 执行语句  
 					EXECUTE sql1;  
                 END IF;  
        -- 再次写入tablename，否则游标会断  
 		FETCH table_list INTO tablename ;  
        UNTIL flag END REPEAT;  
        -- 关闭游标  
	 CLOSE table_list ;  
	   ELSE  
			-- 如果传入了表名，则只修改该表的结构  
 			IF NOT EXISTS (select * from information_schema.columns  
                 where table_schema='uih_system_config_db'  
 				 and table_name=tabName  
                 and column_name=colName)  
                 THEN  
 						SET @exesql2 = CONCAT('ALTER TABLE ',tabName,' ADD COLUMN ',colName,' ',altStr);  
                        PREPARE sql2 FROM @exesql2;  
                        EXECUTE sql2;  
                 END IF;  
   END CASE;  
END //  
-- 分割符改回为 ;
DELIMITER ;  
-- 向所有表添加租户ID  
CALL add_column('uih_system_config_db',null,'TENANT_ID',"bigint(20) DEFAULT NULL COMMENT '租户ID'");
```

#### DELIMITER
delimiter 语句重新定义 sql 语句的分割符，因为sql中默认语句分割符为
";"，sql解释器会直接将一个";"结尾的语句解释为一个完成的sql并执行。
存储过程中可能存在多个关联语句，希望能够一次性存储执行，所以定义存储过程前一般会临时修改存储过程中的SQL语句分隔符，procedure结束后再修改回默认的";".