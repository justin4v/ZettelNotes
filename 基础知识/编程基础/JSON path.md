#JSON
## JsonPath的来源

看它的名字你就能知道，这家伙和JSON文档有关系，正如XPath之于XML文档一样，JsonPath为Json文档提供了解析能力，通过使用JsonPath，你可以方便的查找节点、获取想要的数据，JsonPath是Json版的XPath。

## JsonPath语法

JsonPath的语法相对简单，它采用开发语言友好的表达式形式，如果你了解类C语言，对JsonPath就不会感到不适应。

JsonPath语法要点：

-   `$` 表示文档的根元素
-   `@` 表示文档的当前元素
-   `.node_name` 或 `['node_name']` 匹配下级节点
-   `[index]` 检索数组中的元素
-   `[start:end:step]` 支持数组切片语法
-   `*` 作为通配符，匹配所有成员
-   `..` 子递归通配符，匹配成员的所有子元素
-   `(<expr>)` 使用表达式
-   `?(<boolean expr>)`进行数据筛选
# 参考
1. [JsonPath基本用法](https://www.cnblogs.com/youring2/p/10942728.html)