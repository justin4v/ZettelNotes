#Tools #Flyway #Todo 
# flyway是什么

- Version control for database， 可以像 Git 管理不同人的代码那样，管理不同人的sql脚本，从而做到数据库同步。
-   Flyway is an open-source database migration tool.（社区版免费）
-   支持 [Oracle](https://flywaydb.org/documentation/database/oracle), [DB2](https://flywaydb.org/documentation/database/db2), [MySQL](https://flywaydb.org/documentation/database/mysql) , [MariaDB](https://flywaydb.org/documentation/database/mariadb), [PostgreSQL](https://flywaydb.org/documentation/database/postgresql) , [Aurora PostgreSQL](https://flywaydb.org/documentation/database/aurora-postgresql), [H2](https://flywaydb.org/documentation/database/h2), [SQLite](https://flywaydb.org/documentation/database/sqlite) 等数据库。
    

## flyway工作方式

-   flyway执行sql脚本时左对齐原则，缺位补0，V20220719.1.sql > V20220719.sql

-   **Flyway** 将 **SQL** 文件分为 **Versioned** 、**Repeatable** 和 **Undo** 三种：
    -   **Versioned** 用于版本升级, 每一个版本有惟一的版本号并只能执行一次。
    -   **Repeatable** 可重复执行；
        -   当 **Flyway**检测到 **Repeatable** 类型的 **SQL** 脚本的 `checksum` 有变更, **Flyway** 就会从新应用该脚本.
        -   不用于版本更新, Repeatable 的 `migration` 总是在 **Versioned** 执行以后才被执行。
    -   **Undo** 用于撤销。通常建议使用 **Versioned** 模式来解决回滚需求。
        
![[flyway 脚本命名方式.png]]

-   扫描特定的 classpath或文件夹；
-   按照版本顺序，执行特定版本之后（schema_history 表）的脚本

![[flyway迁移示意图.png]]

-   Schema history表内容示例：

`flyway_schema_history`


## Flyway 集成方式

-   Springboot 已经默认支持引入；
    

-   在 nacos中配置 `spring.flyway...`
    

## Flyway 最佳实践

### SQL 文件

-   开发环境和生产环境的 migration SQL 不共用.
    

-   开发环境脚本多人协作开发,
    

-   生产环境 DB migration 一般由 DBA 完成, 每次升级通常需要提交一个 SQL 脚本.
    

-   DDL DML 分开
    
    -   开发环境 SQL 文件采用时间戳作为版本号, 示例:
        
        -   V20180317.14.59__V1.2_Add_SomeTables.sql
            
        -   V20180317.14.59__V1.0.1_junjie_add_hl7_data.sql
            
    -   生产环境 SQL 文件, 手动 merge 开发环境的 SQL 脚本, 如 V2.1.5_001__Unique_User_Names.sql
        

### 总是新增数据库脚本，而不是修改旧版本 SQL 脚本

### 无序执行脚本配置 spring.flyway.outOfOrder

-   开发环境设置 spring.flyway.outOfOrder=true, flyway 将加载遗漏的老版本 SQL 文件;
    

-   生产环境设置 spring.flyway.outOfOrder=false
    

### 多个服务共用一个库时，配置不同 metadata

-   多服务共用一个库时，为不同的服务配置不同的 metadata 记录表（默认为 flyway_schema_history）
    

-   如：
    
    -   Research ： research_schema_history
        
    -   Prefetch ： prefetch_schema_history
        

### 尽量将 sql 放置到 filesystem，而不是classpath

-   方便替换，不占用jar包大小
    

### 不推荐使用 flyway Undo，过于简单粗暴

-   多为直接删除操作，如 delete drop等语句

# springboot 集成
时序图如下：


![[Flyway_migrate.png]]



# Flyway 最佳实践

## SQL 的文件名

开发环境和生产环境的 migration SQL 不共用. 开发过程往往是多人协作开发, DB migration 也相对比较频繁, 所以 SQL 脚本会很多. 而生产环境 DB migration 往往由 DBA 完成, 每次升级通常需要提交一个 SQL 脚本.

-   DDL DML 分开
    -   开发环境 SQL 文件建议采用时间戳作为版本号, 多人一起开发不会导致版本号争用, 同时再加上生产环境的版本号, 这样的话, 将来手工 merge 成生产环境 V1.2d migration 脚本也比较方便, SQL 文件示例:
        -   V20180317.14.59__V1.2_Add_SomeTables.sql
        -   V20180317.14.59__V1.0.1_ProjectName_{Feature|fix}_Developer_Description.sql
    -   生产环境 SQL 文件, 应该是手动 merge 开发环境的 SQL 脚本, 版本号按照正常的版本, 比如 V2.1.5_001__Unique_User_Names.sql
        

## migration 后的SQL 脚本不应该再被修改.

## spring.flyway.outOfOrder 取值

-   对于开发环境, 可能是多人协作开发, 很可能先 apply 了自己本地的最新 SQL 代码, 然后发现其他同事早先时候提交的 SQL 代码还没有 apply, 开发环境应该设置 spring.flyway.outOfOrder=true, flyway 将能加载漏掉的老版本 SQL 文件;
-   生产环境应该设置 spring.flyway.outOfOrder=false
    

## 公用 DB schema

很多时候多个系统公用一个 DB schema , 这时候使用 spring.flyway.table 为不同的系统设置不同的 metadata 表, 缺省为 flyway_schema_history

## 尽量配置.sql到filesystem
不在项目工程下：方便替换，并不占用jar包大小

## 不推荐使用Undo，太过粗暴


# 参考
1. [Flyway基本介绍及基本使用](https://developer.aliyun.com/article/842712)
2. [Flyway (flywaydb.org)](https://flywaydb.org/)