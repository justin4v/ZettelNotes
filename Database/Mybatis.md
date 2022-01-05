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
- `useGeneratedKeys` :  true /false ，仅限 insert 和 update， 使用 JDBC 的 `getGeneratedKeys` 取出数据库内部生成的主键（比如自动递增）。
- `keyProperty` ：属性名，仅限 insert 和 update，指定能*唯一识别对象的属性*，MyBatis 用 `getGeneratedKeys` 的返回值或 insert 语句的 `selectKey` 子元素设置值，默认值：未设置（`unset`）

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


# Result Maps
具体见下代码：
```sql
## 1.将 column 映射到 map的key,row映射到 map的value
<select id="selectUsers" resultType="map">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>

## 2.映射为 JavaBean/POJO
<select id="selectUsers" resultType="com.someapp.model.User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>

## 3.为properties 设置别名 aliases
<select id="selectUsers" resultType="User"> select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id} </select>
```


# Advanced Result Maps
```sql
<!-- 复杂的语句 --> 
<select id="selectBlogDetails" resultMap="detailedBlogResultMap"> select
       B.id as blog_id,
       B.title as blog_title,
       B.author_id as blog_author_id,
       A.id as author_id,
       A.username as author_username,
       A.password as author_password,
       A.email as author_email,
       A.bio as author_bio,
       A.favourite_section as author_favourite_section,
       P.id as post_id,
       P.blog_id as post_blog_id,
       P.author_id as post_author_id,
       P.created_on as post_created_on,
       P.section as post_section,
       P.subject as post_subject,
       P.draft as draft,
       P.body as post_body,
       C.id as comment_id,
       C.post_id as comment_post_id,
       C.name as comment_name,
       C.comment as comment_text,
       T.id as tag_id,
       T.name as tag_name
  from Blog B
       left outer join Author A on B.author_id = A.id
       left outer join Post P on B.id = P.blog_id
       left outer join Comment C on P.id = C.post_id
       left outer join Post_Tag PT on PT.post_id = P.id
       left outer join Tag T on PT.tag_id = T.id
  where B.id = #{id} </select>
```
- 表示了一篇博客(blog)，由某位作者(author)所写;
- 有多篇博文(post);
- 每篇博文有多条的评论(comment)和标签(tag)；

```sql
<!-- 复杂的语句对应的映射 --> 
<resultMap id="detailedBlogResultMap" type="Blog">
  <constructor>
    <idArg column="blog_id" javaType="int"/>
  </constructor>
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
    <result property="favouriteSection" column="author_favourite_section"/>
  </association>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <association property="author" javaType="Author"/>
    <collection property="comments" ofType="Comment">
      <id property="id" column="comment_id"/>
    </collection>
    <collection property="tags" ofType="Tag" >
      <id property="id" column="tag_id"/>
    </collection>
    <discriminator javaType="int" column="draft">
      <case value="1" resultType="DraftPost"/>
    </discriminator>
  </collection>
</resultMap>
```


其中涉及的属性如下：
## constructor
- resultMap 初始化时传入到构造函数的参数；
- `idArg` ：作为 resultMap ID的参数
-  `arg` ：普通参数

```sql
!<--pojo-->
public class User {
   //...
   public User(Integer id, String username, int age) {
     //...
  }
//...
}

## constructor
<constructor>
   <idArg column="id" javaType="int"/>
   <arg column="username" javaType="String"/>
   <arg column="age" javaType="_int"/>
</constructor>

```


##  id & result
```sql
<id property="id" column="post_id"/> 
<result property="subject" column="post_subject"/>
```

- 都是将 `column` 的值映射到一个简单数据类型（String, int, double）的 `property`
- _id_ 中 `property` 会被*标记为对象的标识符*，比较对象实例时使用。


## association
```sql
<association property="author" column="blog_author_id" javaType="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
</association>
```
- *use-a* 的关系；
- 关联方式：
	-   嵌套 Select：执行另外一个 SQL 语句。
	-   嵌套 resultmap；

### select 方式
```sql
<select id="selectAuthor" resultType="Author">
  SELECT * FROM AUTHOR WHERE ID = #{id}
</select>

<resultMap id="blogResult" type="Blog">
	## 嵌套 select：selectAuthor 得到 result map
  <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

```

- 虽然很简单，但在大型数据集或大型数据表上表现不佳。
- 称为 “*N+1 Selects Problem*” ：
	-   执行单独的 SQL 语句`selectBlog` 得到查询结果（“+1”） 。
	-   结果中每条记录，执行 `selectAuthor` 查询加载详细信息（“N”）。
- 会导致成百上千的 SQL 语句被执行.

### Nested Results
```sql
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  ## Nested Results 
  <association property="author" column="blog_author_id" javaType="Author" resultMap="authorResult"/>
</resultMap>

<resultMap id="authorResult" type="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
  <result property="email" column="author_email"/>
  <result property="bio" column="author_bio"/>
</resultMap>
```


## collection
- 集合元素和关联元素几乎一致；

有属性
```java
private List<Post> posts;
```

### select
```sql
<resultMap id="blogResult" type="Blog">
  <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectPostsForBlog" resultType="Post">
  SELECT * FROM POST WHERE BLOG_ID = #{id}
</select>
```
- ofType 指定 `property` `posts` 中存放的类型是 `Post`


# 参考
1. [mybatis – MyBatis 3](https://mybatis.org/mybatis-3/zh/index.html)