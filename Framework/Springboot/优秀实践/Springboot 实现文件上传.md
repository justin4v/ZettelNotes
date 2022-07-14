#Springboot #Uploadfile

# 上传单个文件
## 前端
先做一个简易的表单，代码如下：

```html
<form enctype="multipart/form-data" method="POST" action="/upload">
    <input type="text" name="imgName" />
    <input type="file" name="imgFile" />
    <input type="submit" value="upload" />
</form>
```

- **注意**：如果要上传文件，必须设定表单为`multipart/form-data`格式；
- `<form>` 标签中的 `enctype` 是设定表单数据格式的属性。
	- ![[HTTP后端接收参数方式#前端HTML参数编码方式]]
- `<input type="file" .....>` 指定了该条目为文件类型。

![[文件上传界面示意.png]]
### javascript 代码实现
- 如果不想使用 form 标签中的按钮，而是自己组装数据，可以写如下函数：
```javascript
/**
 * 上传文件函数
 * @param {*} requestURL 请求地址
 * @param {*} file 文件（input对象的文件）
 */
function upload(requestURL, file) {
	let form = new FormData(); //新建表单对象
	form.append('img', file); //向表单对象添加文件，名字为img
	fetch(requestURL, {
		method: 'POST',
		body: form
	}).then((response) => {
		return response.json();
	}).then((result) => {
		//请求完成后对结果的处理
		console.log(result);
	})
}
```

- 其中获取`input`标签中的`file`：
```javascript
let input = document.querySelector('input'); //查询到input标签
let file = input.files[0]; //input的属性files表示选择的文件数组，如果是单文件就指定下标0
```
- 可以不用指定 `fetch` 中的 `headers` 的`content-type`，发送`FormData`对象可以自行适配。


## 后端
- 在 Spring Boot 中配置**文件上传大小限制**，因为其默认限制是 1mb，所以上传文件稍大就会失败

```properties
# 设置内置Tomcat请求大小为20MB
server.tomcat.max-http-form-post-size=20MB
# 设置请求最大大小为20MB
spring.servlet.multipart.max-request-size=20MB
# 设置文件上传最大大小为20MB
spring.servlet.multipart.max-file-size=20MB
复制代码
```

三个选项最好是都设定一下，如果想无限制可以都填`-1`。

然后在Controller类中写如下方法：

```java
@PostMapping("/upload")
public String upload(@RequestParam("imgFile") MultipartFile file, @RequestParam("imgName") String name) throws Exception {
    // 设置上传至项目文件夹下的uploadFile文件夹中，没有文件夹则创建
    File dir = new File("uploadFile");
    if (!dir.exists()) {
        dir.mkdirs();
    }
    file.transferTo(new File(dir.getAbsolutePath() + File.separator + name + ".png"));
    return "上传完成！文件名：" + name;
}
复制代码
```

需要注意的是，上传过来的文件在Java中是`MultipartFile`类型，`@RequestParam`中的值即为我们前端表单每一项（`<input>`标签）里面的`name`属性值，或者是使用`FormData`对象`append`时对应的那个名字，要一一对应，必须相同。前端表单中的`action`属性即为表单提交至的地址，对应我们Controller的`@PostMapping`中的值。

MultipartFile实例通过使用方法`transferTo`方法实现把上传的文件保存至指定位置

注意最好是使用`@RequestParam`逐个接受参数，因为`@RequestBody`不支持`multipart`类型，参数多了可以分接口接收。

# 上传多个文件

上传多个文件其实也很简单，和上面上传单个文件差别不大。在前端的`<input type="file"...>`这一条表示文件的条目标签中加入属性multiple即可，我的代码如下：

```html
<form enctype="multipart/form-data" method="POST" action="/upload">
    <input type="text" name="imgColName" />
    <input type="file" name="imgFile" multiple />
    <input type="submit" value="upload" />
</form>
复制代码
```

这时，再上传文件时后端接收到的是MultipartFile数组，因此在上述的Controller方法中的对应的形参改成`MultipartFile[]`类型即可，然后遍历操作，具体例子如下：

```java
@PostMapping("/upload")
public String upload(@RequestParam("imgFile") MultipartFile[] files, @RequestParam("imgColName") String name) throws Exception {
    File dir = new File("uploadFile");
    if (!dir.exists()) {
        dir.mkdirs();
    }
    for (MultipartFile file : files) {
        file.transferTo(new File(dir.getAbsolutePath() + File.separator + file.getOriginalFilename()));
    }
    return "上传完成！图集名：" + name;
}
```


# 参考
1. [Spring Boot实现文件上传](https://juejin.cn/post/6989115926503227399)