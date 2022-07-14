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

  

# 参考
1. [Spring Boot实现文件上传](https://juejin.cn/post/6989115926503227399)