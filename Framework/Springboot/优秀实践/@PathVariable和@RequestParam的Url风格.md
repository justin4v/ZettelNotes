#RESTful #Url-style #PathVariable #RequestParam

# 比较
- @RequestParam和@PathVariable都能够接收用户的输入；
- 输入的部分不同，一个在URL路径部分，另一个在参数部分。
- 要访问一篇博客文章，这两种URL设计都是可以的：
	-   通过@PathVariable，例如/blogs/1
	-   通过@RequestParam，例如blogs?blogId=1

## 建议
1.  当URL指向的是某一*具体业务资源（或者资源列表）*，使用@PathVariable，例如 `/blogs/1`
2.  当URL需要对*资源或者资源列表进行过滤筛选时*，用@RequestParam，例如 `blogs?blogId=1`

例如这样设计URL：
-   /blogs/{blogId}：操作 blogId 的博客（查询/删除）
-   /blogs?state=publish 表示处于发布状态的博客文章，而不是/blogs/state/publish来