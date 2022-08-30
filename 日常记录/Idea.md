#Idea
# WM
新闻播报   个人照片
两个人照片 出去玩 平时视频
照片墙  组成心型
走过的地点 连成图形

感人部分


# 专利方向

## 前端UI
1. 静态展现类的：如IOS界面中四排APP图表，下面DOCK栏一排常用图标，苹果外观设计专利。
2. 交互类：如滑动解锁技术，苹果前后累计申请十多件发明。

## 数据传输
1. 保证数据传输的效率，速度、安全性的技术；
2. 降低网络占用、提高稳定性，网络切换的技术；如云加速、离线下载、P2P类技术；
3. 编解码类技术、客户端和服务器之间交互或同步的流程方案等等。

## 服务器后台
1. 数据存储;
2. 查询
3. 响应处理
4. 服务器负载管理等。


# 依赖数据存在校验
问题 ：
1. 假如有关联项 A，B，C，D；
2. 合法性要求：A 存在时 B 才能存在，B 存在时 C 才能存在...依次类推。
3. 现在给定 A B C D 的一种可能判断其输入合法性

分析思路：
1. 其实可以看做是要求保持输入值的某种*连续性*。
2. 假设存在时记为 1，不存在记为 0；
3. 依次读入 A B C D：
	1. 当读入到 1 时，后续值存在或者不存在都有可能；
	2. 当读入到 0 时，后续值必须为 0 ，否则整体将*不连续*，不合法。

java 实现：

```java
public Boolean valid() {
	// list 存储需要判断的值，方便遍历
    List<String> params = List.of(Optional.ofNullable(this.domain).orElse(""),
        Optional.ofNullable(this.grade).orElse(""),
        Optional.ofNullable(this.className).orElse(""));
    boolean checkBit = false;
    for (String param : params) {
      // 如果已经出现0，后续不能再出现 1
      if (checkBit && StringUtils.isNotBlank(param)) {
        return false;
      }
      // 出现 0，则后续不允许再出现 1
      if (StringUtils.isBlank(param)) {
        checkBit = true;
      }
    }
    return true;
  }
```



# 备选
## HL7 大量消息导致出现mysql数据库无法及时持久化
- 出现 `Lock wait timeout exceeded`与`Dead Lock`
- 线程池满了，无法提交任务
- 数据库连接池满了


优化：
hl7 大量请求（快）  和  数据持久化不及时（慢），两者如何适配？

- 参考 mysql 方式，使用日志记录
- 优先队列，排序重要类型的 HL7 消息（其次根据日期排序），优先持久化


疑问
1. 为什么不用消息队列，redis缓存？
	1. 需要引入中间件，增加维护成本
	2. HL7 消息数目正常一天有 15-20万条，文本类型，暂用空间较大。使用redis会暂用较大空间内存，甚至出现 OOM 空间不够情况
	3. 使用redis  bgsave 等持久化功能，需要在空闲时间处理（阻塞其他线程），无法及时分析以及统计。
2. 创新点
	1. 计算消息权重（时间局部性原理--检查时间，空间局部性原理--重要消息关联消息）消息类型与检查series数目
	2. 优先队列排队
	3. 轻量，无需中间件

3. 日志如何设计
	1. 文件大小设计为 page 大小




# 一种基于 HL7 消息的影像一致性校验方法

![[hl7影像一致性 1.svg]]


- 近似计数算法进行聚合
- 对 HotKey的访问进行计数，使用 LFU 算法保留访问频次高的 hotkey；
	- 使用优先队列按频次排序 hotkey；
	- 更新计数，则设置其状态 invalid，查询命中之前需要从数据库同步最新数据；
- PACS -> redis ：缓存并进行消息合并
	- redis 更新到数据库，mysql 从左到右消费，从右向左 merge 消息；
- storage -> redis：缓存命中、缓存更新


## 参考
[Redis LFU 实现 -- 掷硬币的艺术](https://blog.csdn.net/jh0218/article/details/95389361)