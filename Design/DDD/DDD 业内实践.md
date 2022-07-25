#DDD #领域驱动设计


# DDD领域设计实践
## DDD设计思路
### 战略设计
- 从业务视角出发，建立业务领域模型；
- 划分领域边界，建立通用语言的限界上下文；
- 限界上下文可以作为微服务设计的参考边界。

- 业务领域可按照：核心领域、支撑领域、通用领域，划分边界；
- 但是领域都是业务相关的；
- 如风险控制在银行金融业务中一般是作为支撑领域，但是在一些领域可能就是核心领域。

### 战术设计
- 从技术视角出发，侧重于领域模型的技术实现，完成软件开发和落地；
- 包括：聚合根、实体、值对象、领域服务、应用服务和资源库等代码逻辑的设计和实现

## DDD设计方法
1.  一步：在事件风暴（或者*四色分析*）中梳理业务过程中的用户操作、事件以及外部依赖关系等，根据这些要素梳理出领域实体等领域对象。
    
2.  第二步：务紧密相关的实体进行组合形成聚合，同时确定聚合中的聚合根、值对象和实体。
    
3.  第三步：根据业务及语义边界等因素，将一个或者多个聚合划定在一个限界上下文内，形成领域模型。在这个图根据领域实体之间的业务关联性，将业里，限界上下文之间的边界是第二层边界，这一层边界可能就是未来微服务的边界，不同限界上下文内的领域逻辑被隔离在不同的微服务实例中运行，物理上相互隔离
    

# 总结
- 贫血对象和富对象

![[架构设计目标总结.png]]




# 参考

阿里技术专家详解DDD系列
第一讲 Domain Primitive   
https://mp.weixin.qq.com/s/kpXklmidsidZEiHNw57QAQ   
   
第二讲 - 应用架构   
https://mp.weixin.qq.com/s?__biz=MzAxNDEwNjk5OQ%3D%3D&mid=2650404060&idx=1&sn=cacf40d19528f6c2d9fd165151d6e8b4&scene=45#wechat_redirect   
   
第三讲 - Repository模式   
https://mp.weixin.qq.com/s/9efoPbvp0GaMi24V0RxFcw   
   
第四讲：领域层设计规范   
https://mp.weixin.qq.com/s/NoRTUSovcO2Yz237k0ceIw  
第五讲：聊聊如何避免写流水账代码  
https://mp.weixin.qq.com/s?__biz=MzAxNDEwNjk5OQ==&mid=2650427571&idx=1&sn=bfc3c1c6f189965a1a4c7f3918012405&chksm=839698abb4e111bd5e02344f27d86c928ccfe4d3da1649817b02924c07f681fc1a7ea818f442&scene=178&cur_album_id=1452661944472977409#rd


vivo互联网技术  
领域驱动设计(DDD)实践之路(一)  
https://mp.weixin.qq.com/s/gk-Hb84Dt7JqBRVkMqM7Eg  
  
领域驱动设计(DDD)实践之路(二)：事件驱动与CQRS  
https://mp.weixin.qq.com/s/Z3uJhxJGDif3qN5OlE_woA  
  
领域驱动设计(DDD)实践之路(三)：如何设计聚合  
https://mp.weixin.qq.com/s/oAD25H0UKH4zujxFDRXu9Q  
  
领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用  
https://mp.weixin.qq.com/s/YL_NRiEKgR-D3_yx3FARbg


开源代码实践：  
1、https://gitee.com/meekyun/dddbook  
2、https://gitee.com/murmuring/leave-sample


【美团】领域驱动设计在互联网业务开发中的实践  
https://tech.meituan.com/2017/12/22/ddd-in-practice.html  
  
【百度】领域驱动设计(DDD)在百度爱番番的实践  
https://mp.weixin.qq.com/s/vGhEQoHfxlIuoOuGp6AGNg  
  
【逻辑思维】用领域驱动设计实现订单业务的重构  
https://mp.weixin.qq.com/s/fGQAz7mG146bxyzAtIveMg