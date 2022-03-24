#JSON
# 来源
- XPath 为 XML文档提供了解析能力；
- *JsonPath 为 Json文档提供了解析能力*；
- 通过JsonPath，可以方便的查找节点、获取想要的数据，JsonPath 是 Json 版的 XPath。

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

# 语法

| XPath | JsonPath           | 说明                                            |
| ----- | ------------------ | --------------------------------------------- |
| `/`   | `$`                | 文档根元素                                         |
| `.`   | `@`                | 当前元素                                          |
| `/`   | `.`或`[]`           | 匹配下级元素                                        |
| `..`  | `N/A`              | 匹配上级元素，JsonPath不支持此操作符                        |
| `//`  | `..`               | 递归匹配所有子元素                                     |
| `*`   | `*`                | 通配符，匹配下级元素                                    |
| `@`   | `N/A`              | 匹配属性，JsonPath不支持此操作符                          |
| `[]`  | `[]`               | 下标运算符，根据索引获取元素，**XPath索引从1开始，JsonPath索引从0开始** |
| `     | `                  | `[,]`                                         |
| `N/A` | `[start:end:step]` | 数据切片操作，XPath不支持                               |
| `[]`  | `?()`              | 过滤表达式                                         |
| `N/A` | `()`               | 脚本表达式，使用底层脚本引擎，XPath不支持                       |
| `()`  | `N/A`              | 分组，JsonPath不支持                                |

注意：

- JsonPath 索引从 0 开始计数
- JsonPath 中字符串使用单引号表示，例如 `$.store.book[?(@.category=='reference')]`中的`'reference'`

# JsonPath 示例

```json
{
    "store": {
        "book": [{
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            }, {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99
            }, {
                "category": "fiction",
                "author": "Herman Melville",
                "title": "Moby Dick",
                "isbn": "0-553-21311-3",
                "price": 8.99
            }, {
                "category": "fiction",
                "author": "J. R. R. Tolkien",
                "title": "The Lord of the Rings",
                "isbn": "0-395-19395-8",
                "price": 22.99
            }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    }
}
```

接下来我们看一下如何对这个文档进行解析：

| XPath      | JsonPath        | Result                    |
| -----------------------| --------------- | ---------------|
| `/store/book/author`                 | `$.store.book[*].author`       | 所有book的author节点              |
| `//author`                             | `$..author`                | 所有author节点                   |
| `/store/*`                          | `$.store.*`                | store下的所有节点，book数组和bicycle节点 |
| `/store//price`                       | `$.store..price`           | store下的所有price节点             |
| `//book[3]`                         | `$..book[2]`                 | 匹配第3个book节点                  |
| `//book[last()]`               | `$..book[(@.length-1)]`，或 `$..book[-1:]` | 匹配倒数第1个book节点                |
| `//book[position()<3]`           | `$..book[0,1]`，或 `$..book[:2]`           | 匹配前两个book节点                  |
| `//book[isbn]`                      | `$..book[?(@.isbn)]`                   | 过滤含isbn字段的节点                 |
| `//book[price<10]`                   | `$..book[?(@.price<10)]`    | 过滤`price<10`的节点              |
| `//*`                                | `$..*`               | 递归匹配所有子节点                    |
| 在 [JSONPath Online Evaluator](http://jsonpath.com/) 验证JsonPath的执行效果。 |                                          |                              |


# 参考
1. [JsonPath基本用法](https://www.cnblogs.com/youring2/p/10942728.html)