#Tools #Flyway #Todo 
# 简介
- Flyway是独立于数据库的应用；
- 管理并跟踪数据库变更的使用Java编写的数据库版本管理工具。
- Flyway可以像 Git 管理不同人的代码那样，管理不同人的sql脚本，从而做到数据库同步。


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