#Spring 

# 目的
- Aware 接口意为感知捕获。
- 单纯的bean（未实现Aware系列接口）是没有知觉的；
- 实现了Aware系列接口的bean可以访问 Spring 容器。这些Aware系列接口增强了Spring bean的功能，但是也会造成对Spring框架的绑定，增大了与Spring框架的耦合度

# 参考
1. [Spring之Aware接口介绍](https://cloud.tencent.com/developer/article/1409274)