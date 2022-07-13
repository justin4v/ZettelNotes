#Springboot #Ways-of-pass-param


# 四种传参方式

# path info传参
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

## Body 传参：@RequestParam

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

## Body 传参 和 Headers 传参

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

- @RequestBody 接收json形式的数据自动转换成实体类；
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
# 参考
1. [HTTP协议的四种传参方式](https://www.cnblogs.com/jinyuanya/p/13934722.html)