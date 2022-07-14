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
- 在 Spring Boot 中配置**文件上传大小限制**，**其默认限制是 1mb**，所以上传文件稍大就会失败
- 无限制可以填 `-1`。

```properties
# 设置内置Tomcat请求大小为20MB
server.tomcat.max-http-form-post-size=20MB
# 设置请求最大大小为20MB
spring.servlet.multipart.max-request-size=20MB
# 设置文件上传最大大小为20MB
spring.servlet.multipart.max-file-size=20MB
复制代码
```


Controller：
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
```
- `multipart/form-data` 格式上传的文件在 Java 中是`MultipartFile`类型；
- `@RequestParam` 中 `value` 对应前端表单每一项（`<input>`标签）的 `name` 属性值，或者`FormData`对象`append` 时的名字。
- 前端表单中的 `action` 属性即为表单提交至的地址，对应 Controller 该请求的地址 。
- 实例中通过 `transferTo` 方法把上传的文件保存至指定位置。
- 最好使用 `@RequestParam` 接收参数，`@RequestBody` 不支持 `multipart`类 型。

# 上传多个文件
- 上传多个文件和单个文件类似。
- 前端的 `<input type="file"...>` 标签中加入属性 multipl e即可，代码如下：

```html
<form enctype="multipart/form-data" method="POST" action="/upload">
    <input type="text" name="imgColName" />
    <input type="file" name="imgFile" multiple />
    <input type="submit" value="upload" />
</form>
```

- 后端接收到的是 MultipartFile 数组，在 Controller 中形参改成 `MultipartFile[]` 类型，然后遍历操作：

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