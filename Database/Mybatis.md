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
- 语句名为 `selectPerson`;
- 接受一个 int（或 Integer）类型的参数;
- 返回一个 HashMap 类型的对象;
- 键是列名，值是结果行中的值.


```sql
#{id}
```
- 预处理语句（PreparedStatement）参数;
- JDBC 中，由一个 “?” 来标识，并被传递到预处理语句中，如
```
// JDBC 代码，非 MyBatis 代码... 
String selectPerson = "SELECT * FROM PERSON WHERE ID=?"; 
PreparedStatement ps = conn.prepareStatement(selectPerson); 
ps.setInt(1,id);
```
- JDBC 需要更多的代码，mybatis 则不需要。

### 语法
```sql
<select
  id="selectPerson"
  parameterType="int"
  parameterMap="deprecated"
  resultType="hashmap"
  resultMap="personResultMap"
  flushCache="false"
  useCache="true"
  timeout="10"
  fetchSize="256"
  statementType="PREPARED"
  resultSetType="FORWARD_ONLY">
```


| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | 命名空间中的唯一标识符，可被用来引用这条语句。           |
| parameterType | 传入参数的类全限定名(fully qualified class name)或别名。可选，MyBatis 可通过TypeHandler推断出参数类型，默认值为 unset |
| parameterMap  | 引用外部 parameterMap，目前已被废弃 |
| resultType    | 返回结果的类全限定名或别名。resultType和resultMap 只能同时使用一个。 |
| resultMap     | 对外部 resultMap 的引用。resultType 和 resultMap 之间只能同时使用一个。 |
| flushCache    | true时只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。 |
| useCache      | true时将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。 |
| timeout       | 抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
| fetchSize     | 给驱动的建议值，让驱动程序每次批量返回的结果行数等于这个设置值。  默认值为未设置（unset）（依赖驱动）。 |
| statementType | 可选 STATEMENT，PREPARED 或  CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或  CallableStatement，默认值：PREPARED。 |
| resultSetType | FORWARD_ONLY，SCROLL_SENSITIVE,  SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。 |
| databaseId    | 数据库厂商标识（databaseIdProvider），MyBatis会加载所有带 databaseId；如果带和不带的语句都有，则不带的会被忽略。 |
| resultOrdered | 仅针对嵌套结果 select 语句：默认值：false。 |
| resultSets    | 仅适用于多结果集的情况。列出语句执行后返回的结果集并赋予每个结果集一个名称，以逗号分隔。 |

# insert,update 和 delete
```sql
<insert
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="updateAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="deleteAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

```
- `useGeneratedKeys` : insert 和 update， 使用 JDBC 的 `getGeneratedKeys` 取出数据库内部生成的主键（比如自动递增），默认值：false
- `keyProperty` ：insert 和 update，指定能唯一识别对象的属性，MyBatis 用 `getGeneratedKeys` 的返回值或 insert 语句的 `selectKey` 子元素设置值，默认值：未设置（`unset`）

示例：
```sql
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>

<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>
```
# 参考
1. [mybatis – MyBatis 3](https://mybatis.org/mybatis-3/zh/index.html)