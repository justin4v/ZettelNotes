#Mybatis #Database
# 简介
- 持久层框架（persistence framework）；
- 支持自定义 SQL、存储过程（[[Stored Procedure]]）以及高级映射。

# 类型别名
- 目的是为 Java 类型设置一个缩写名字；
- 为常见的 Java 类型内建了类型别名，不区分大小写；


# 类型处理器(typeHanders)
- MyBatis 在赋予预处理语句（PreparedStatement）中的参数值或从结果集中取出一个值时，会用类型处理器将获取到的值转换成 Java 类型；
- 可 override 或 新建 typeHanders；


# 映射 XML
## select
```sql
<select id="selectPerson" parameterType="int" resultType="hashmap">
SELECT * FROM PERSON WHERE ID = #{id} 
</select>
```
- 语句名为 selectPerson;
- 接受一个 int（或 Integer）类型的参数;
- 返回一个 HashMap 类型的对象;
- 键是列名，值是结果行中的对应值

# 参考
1. [mybatis – MyBatis 3](https://mybatis.org/mybatis-3/zh/index.html)