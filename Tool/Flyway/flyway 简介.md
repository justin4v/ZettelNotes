#Tools #Flyway 
# flyway是什么

- **Version control for database**， 可以像 Git 管理不同人的代码那样，管理不同人的sql脚本，从而做到数据库同步。
-   Flyway is an open-source database migration tool.（社区版免费）
-   支持 [Oracle](https://flywaydb.org/documentation/database/oracle), [DB2](https://flywaydb.org/documentation/database/db2), [MySQL](https://flywaydb.org/documentation/database/mysql) , [MariaDB](https://flywaydb.org/documentation/database/mariadb), [PostgreSQL](https://flywaydb.org/documentation/database/postgresql) , [Aurora PostgreSQL](https://flywaydb.org/documentation/database/aurora-postgresql), [H2](https://flywaydb.org/documentation/database/h2), [SQLite](https://flywaydb.org/documentation/database/sqlite) 等数据库。


## flyway工作方式
-   扫描特定的 classpath 或文件夹；
-   按照版本顺序，执行特定版本之后（schema_history 表）的脚本

![[flyway迁移示意图.png]]

-   Schema history表内容示例：

`flyway_schema_history`
| installed_rank | version | description   | type | script                | checksum   | installed_by | installed_on          | execution_time | success |
| -------------- | ------- | ------------- | ---- | --------------------- | ---------- | ------------ | --------------------- | -------------- | ------- |
| 1              | 1       | Initial Setup | SQL  | V1__Initial_Setup.sql | 1996767037 | axel         | 2016-02-04 22:23:00.0 | 546            | true    |
| 2              | 2       | First Changes | SQL  | V2__First_Changes.sql | 1279644856 | axel         | 2016-02-06 09:18:00.0 | 127            | true    |
| 3              | 2.1     | Refactoring   | JDBC | V2_1__Refactoring     |            | axel         | 2016-02-10 17:45:05.4 | 251            | true    |

### 脚本类型
-   flyway执行sql脚本时左对齐原则，缺位补0，V20220719.1.sql > V20220719.sql
-   **Flyway** 将 **SQL** 文件分为 **Versioned** 、**Repeatable** 和 **Undo** 三种：
    -   **Versioned** 用于版本升级, 每一个版本有惟一的版本号并只能执行一次。
    -   **Repeatable** 可重复执行；
        -   当 **Flyway**检测到 **Repeatable** 类型的 **SQL** 脚本的 `checksum` 有变更, **Flyway** 就会从新应用该脚本.
        -   不用于版本更新, Repeatable 的 `migration` 总是在 **Versioned** 执行以后才被执行。
    -   **Undo** 用于撤销；
	    - 多为直接删除操作，如 delete drop等语句。
	    - 通常不建议使用，可用 **Versioned** 模式来解决回滚需求。

![[flyway 脚本命名方式.png]]

## 集成方式
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



# 配置示例
common
```YAML
flyway:
    # 启用或禁用 flyway
    enabled: true
    # 字符编码
    encoding: utf-8
    # 对执行迁移时基准版本的描述
    baseline-description: ===== Baseline(create tables add data) =====
    # 若连接的数据库非空库，是否初始化
    # 当迁移时发现目标schema非空，而且带有没有元数据的表时，是否自动执行基准迁移，默认false.
    baseline-on-migrate: true
    # 指定 baseline 的版本号,缺省值为 1, 低于该版本号的 SQL 文件, migrate 的时候被忽略
    # 开始执行基准迁移时对现有的schema的版本打标签，默认值为1.
    baseline-version: 1
    # 是否开启校验
    # 迁移时是否校验，默认为 true
    validate-on-migrate: true
    # flyway 的 clean 命令会删除指定 schema 下的所有 table，默认 false
    locations:
      - classpath:db/migration/iteration
    clean-disabled: true
    # 发环境最好开启 outOfOrder, 生产环境关闭 outOfOrder
    # 是否允许无序的迁移，默认 false
    out-of-order: false
    # 检查迁移脚本的位置是否存在，默认false
    check-location: false
    # 当读取元数据表时是否忽略错误的迁移，默认false
    ignore-future-migrations: false
    # 当初始化好连接时要执行的SQL
    init-sqls: 
      - show tables;
```

服务中只需要配置 metadata 表名
workflow-prefetch
```YAML
spring:
  flyway:
    # 启用或禁用 flyway
    enabled: true
    # 版本记录表
    table: prefetch_schema_history
```

# 参考
1. [Flyway基本介绍及基本使用](https://developer.aliyun.com/article/842712)
2. [Flyway (flywaydb.org)](https://flywaydb.org/)