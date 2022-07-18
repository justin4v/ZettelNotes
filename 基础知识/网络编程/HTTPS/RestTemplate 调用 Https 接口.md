#Springboot #Https

# 背景
本系统接口都是http的，调用第三方接口，因为做了安全性校验，所以不能通过RestTemplate调用

# 解决
- 重写覆盖 `SimpleClientHttpRequestFactory` 抽象类的 `prepareConnection` 方法

```
`
```
# 参考
1. [SpringBoot系列之RestTemplate调https接口 ](https://juejin.cn/post/684490419921435