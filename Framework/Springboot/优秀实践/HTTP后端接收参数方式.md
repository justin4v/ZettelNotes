#Springboot #Ways-of-pass-param

# 前端HTML参数编码方式
- HTML \<form> 的 enctype 属性规定在发送到服务器之前应该如何对表单数据进行编码

![[HTML form enctype 属性值及含义.png]]
1. HTTP 是以 ASCII 码传输，建立在 TCP/IP 协议之上的应用层规范。
2. 规范把 HTTP 请求分为三个部分：状态行、请求头、消息主体。  
3. 协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须 **使用什么编码方式** 。开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。
4. 数据发送出去，服务端要解析成功才有意义。一般服务端语言如 java、python 及相关 framework 等，都内置了自动解析常见数据格式的功能。
5. 服务端通常是根据请求头（headers）中的 Content-Type 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。
6. `application/x-www-form-urlencoded` 和 `multipart/form-data` 都是**浏览器原生支持**。


# 四种传参方式

| HTTP协议组成         | 协议内容示例                                     | 对应Spring注解 |
| -------------------- | ------------------------------------------------ | -------------- |
| path info传参        | /articles/12 (查询id为12的文章，12是参数)        | @PathVariable  |
| URL Query String传参 | /articles?id=12                                  | @RequestParam  |
| Body 传参            | Content-Type: application/x-www-form-urlencoded  | @RequestParam  |
| Body 传参            | Content-Type: multipart/form-data                | @RequestParam  |
| Body 传参            | Content-Type: application/json，或其他自定义格式 | @RequestBody   |
| Headers 传参         |                                                  | @RequestHeader |

# path info 传参
请求路径：`http://localhost:8080/rest/articles/1`

```java
    /**
     * 获取一篇Article，使用GET方法,根据id查询一篇文章
     * restful注意：
     * 请求路径要是复数的，加上s
     * 又因为是{id}，所以要接收参数值用@PathVariable("id")
     * method = RequestMethod.GET：必须要求是get请求
     */
    //@RequestMapping(value = "/articles/{id}", method = RequestMethod.GET)
    @GetMapping("/articles/{id}")
    public AjaxResponse<Article> getArticle(
            // @PathVariable作用：接收请求路径中占位符的值
            @PathVariable("id") Long id
    ) {
        // 不搞数据库查询，做一个伪代码
        Article article = Article.builder().id(1l).author("周周").title("z1").content("RESTful风格查询一篇文章").createTime(new Date()).build();
        // 打印一下内容
        log.info("article:", article);
        // 返回给前端内容
        return AjaxResponse.success(article);
    }
```

# URL Query String传参
请求路径：`http://localhost:8080/rest/articles?name=zhoujinyuan`

```java
    //@RequestMapping(value = "/articles", method = RequestMethod.GET)
    @GetMapping("/articles")
    public AjaxResponse<Article> getArticle(
            @RequestParam String name
    ) {
        // 打印一下内容
        log.info("name:"+name);
        // 返回给前端内容
        return AjaxResponse.success();
    }
```

# Body 传参：@RequestParam

请求路径：`http://localhost:8080/rest/articles1`

```java
/**
     * 新增一篇Article，使用Post方法
     *
     * @param author
     * @param title
     * @param content
     * @param createTime
     * @return
     */
    // @RequestMapping(value = "/articles1", method = RequestMethod.POST)
    @PostMapping("/articles1")
    public AjaxResponse saveArticle(
            // @RequestParam：将请求参数绑定到你控制器的方法参数上（是springmvc中接收普通参数的注解）
            @RequestParam Long id,
            @RequestParam String author,
            @RequestParam String title,
            @RequestParam String content,
            /*
             这里加了两个注解：
                @DateTimeFormat：在SpringMVC中Controller中方法参数为Date类型想要限定请求传入时间格式时，可以通过@DateTimeFormat来指定，但请求传入参数与指定格式不符时，会返回400错误。
                @RequestParam：请求的参数和方法中的参数做对应的注解
             */
            @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") @RequestParam Date createTime
            ) {
        log.info("saveArticle:" + createTime);
        return AjaxResponse.success();
    }
```

报错：required string parameter 'XXX'is not present

```java
情况一：原因是由于头文件类型不对，可以在MediaType中选择合适的类型，例如GET和POST

情况二：jquery提交delete时，不支持@RequestParam，只支持@PathVariable形式

情况三：若api在调用的时候，如果存在重类型，但不重名；例如：/id与/name，两者在类型上是一样的

情况四：这里提示Required String parameter 'XXX' is not present并不一定是XXX的错，也有可能是后面的参数错误。总的来说就是页面传递的参数和后台接受参数名自不匹配。

情况五：传递的参数里面包含特殊符号，比如前台传递字符串不能包含逗号等。

情况六：传的参数是undefined；
```

- 用了 @RequestParam 接收参数，则只能接受表单形式的提交。类似get中 ❓ 后一键一值。
- @RequestBody才是接收json形式的数据的！！！

# Body 传参 和 Headers 传参

请求路径：`http://localhost:8080/rest/articles2`
```java
    /**
     * 新增一篇Article，使用Post方法
     * restful注意：
     */
    // @RequestMapping(value = "/articles2", method = RequestMethod.POST)
    @PostMapping("/articles2")
    public AjaxResponse saveArticle(
            // body形式的传参：传过来的参数自动封装成实体类对象，还有一个好处就是可以接收集合类型的参数。
            @RequestBody Article article,
            // 请求头的方式传参：用@RequestHeader接收键名，形参为值；postman中
            @RequestHeader(value = "name") String name
    ) {
        // 请求头参数中的Host属性
        System.out.println(name);
        // 日志打印
        log.info("article:" + article);
        return AjaxResponse.success();
    }
```

- @RequestBody 接收 json 形式的数据*自动转换成实体类*；
- 还可以接受集合类型的数据。

```json
{
    "id": 1,
    "author": "zhoujinyuan",
    "title": "boot",
    "content": "body传参",
    "createTime": "2020-10-20 12:12:12",
    "reader":[{"name":"zhou","age":18},{"name":"tom","age":37}]
}
```

- 用注解 @RequestHeader 之后，传数据的同时也还要把请求头中的数据传过来（没有的话自定义一个也要穿过来，不然报错400）。


# postman 提交参数几种格式
## form-data
- 等价于 http 请求中的 *multipart/form-data* ；
- 会将表单的数据*转换为一条消息*，以标签为单元，用分隔符（boundary）分开。
- 既可以上传键值对，也可以上传文件。
	- 上传文件时，会有 Content-Type 来表名文件类型；
	- content-disposition，用来说明字段的一些信息；  
	- 可以上传多个文件

```http
POST /api/research/v2/researchUser/importByExcel/preParser?locationCode=lc20200101&locationName=测试机构&officeCode=0101&officeName=测试科室 HTTP/1.1
Host: localhost:8091
User-Agent: apifox/1.0.0 (https://www.apifox.cn)
Content-Length: 235
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="C:\Users\junjie.fu\Desktop\templates\student-info.xlsx"
Content-Type: <Content-Type header here>

(data)
----WebKitFormBoundary7MA4YWxkTrZu0gW
```

## x-www-form-urlencoded
- 等价于 application/x-www-from-urlencoded，会将表单内的数据转换为键值对；
- 如 `name=java&age = 23`


# raw
- raw 对应的是入参是任意格式的可以上传任意格式的【文本】，可以上传text、json、xml、html等

![[raw 格式示意.png]]


# binary
- 相当于 Content-Type:application/octet-stream；
- 只可以上传二进制数据，通常用来上传文件，但是*一次只能上传一个文件*；

![[Springboot 实现文件上传#post binary方式]]

# 参考
1. [HTTP协议的四种传参方式](https://www.cnblogs.com/jinyuanya/p/13934722.html)
2. [[Springboot 实现文件上传]]