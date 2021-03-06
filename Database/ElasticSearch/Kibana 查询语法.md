#Elastic-search #Kibana #Query-language

# 全文搜索
在搜索栏输入login，会返回所有字段值中包含login的文档  
使用双引号包起来作为一个短语搜索  
`"like Gecko"`

# 字段
也可以按页面左侧显示的字段搜索  
限定字段全文搜索：`field:value`  
精确搜索：关键字加上双引号 `filed:"value"`  
`http.code:404` 搜索http状态码为404的文档

字段本身是否存在  
`_exists_:http`：返回结果中需要有http字段  
`_missing_:http`：不能含有http字段

# 通配符
`?` 匹配单个字符  
`*` 匹配0到多个字符

`kiba?a`, `el*search`

> `?` `*` 不能用作第一个字符，例如：`?text` `*text`

# 正则
es支持部分[正则](https://link.segmentfault.com/?enc=hyHs%2BpxARVIdq9KgAQ1jSA%3D%3D.AuWrTTd2SukIYuVQGTvrOEqM20QDqUvGLGcjTIAEhbC14txESCFa%2FjnjdwH2fm7fmZubSXD0w2lz9ymBBQeInkfwXfC7%2B1A8lSkaWvIG3c4BBc2sQWyC0FffAnaPUj%2Bt5fHloiHxNDSJH%2BcCttmfIg%3D%3D)功能,性能较差  
`name:/joh?n(ath[oa]n)/`

# 模糊搜索
`quikc~ brwn~ foks~`  
`~`:在一个单词后面加上`~`启用模糊搜索，可以搜到一些拼写错误的单词

`first~` 这种也能匹配到 frist

还可以设置编辑距离（整数），指定需要多少相似度  
`cromm~1` 会匹配到 from 和 chrome  
默认2，越大越接近搜索的原始值，设置为1基本能搜到80%拼写错误的单词

# 近似搜索
在短语后面加上`~`，可以搜到被隔开或顺序不同的单词  
`"where select"~5` 表示 select 和 where 中间可以隔着5个单词，可以搜到 _select password from users where id=1_

# 范围搜索
数值/时间/IP/字符串 类型的字段可以对某一范围进行查询  
`length:[100 TO 200]`  
`sip:["172.24.20.110" TO "172.24.20.140"]`  
`date:{"now-6h" TO "now"}`  
`tag:{b TO e}` 搜索b到e中间的字符  
`count:[10 TO *]` * 表示一端不限制范围  
`count:[1 TO 5}` [ ] 表示端点数值包含在范围内，{ } 表示端点数值不包含在范围内，可以混合使用，此语句为1到5，包括1，不包括5  
可以简化成以下写法：  
`age:>10`  
`age:<=10`  
`age:(>=10 AND <20)`

# 优先级
`quick^2 fox`  
使用`^`使一个词语比另一个搜索优先级更高，默认为1，可以为0~1之间的浮点数，来降低优先级

# 逻辑操作
`AND`  
`OR`

`+`：搜索结果中必须包含此项  
`-`：不能含有此项  
`+apache -jakarta test aaa bbb`：结果中必须存在apache，不能有jakarta，剩余部分尽量都匹配到

# 分组
`(jakarta OR apache) AND jakarta`

# 字段分组
`title:(+return +"pink panther")`  
`host:(baidu OR qq OR google) AND host:(com OR cn)`

# 转义特殊字符
`+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /`  
以上字符当作值搜索的时候需要用`\`转义  
`\(1\+1\)\=2`用来查询_(1+1)=2_

# 参考
1. [ELK：kibana使用的lucene查询语法](https://segmentfault.com/a/1190000002972420)
2. [kibana查询语法](https://www.cnblogs.com/mliu/p/9584395.html)