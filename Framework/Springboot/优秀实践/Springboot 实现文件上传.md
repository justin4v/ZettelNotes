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

![[Pasted image 20220714135943.png]]

# 参考
1. [Spring Boot实现文件上传](https://juejin.cn/post/6989115926503227399)