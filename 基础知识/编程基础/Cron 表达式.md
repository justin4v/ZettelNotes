#Cron #Cron-expression


*Cron expressions* are used to configure instances of CronTrigger, a subclass of org.quartz.Trigger. 
A cron expression is a string consisting of *six or seven subexpressions* (fields) that describe individual details of the schedule.

# 语法
1. *Seconds Minutes Hours DayofMonth Month DayofWeek Year*
2. *Seconds Minutes Hours DayofMonth Month DayofWeek*
3. *秒 分 小时 月份中的日期 月份 星期中的日期 年份*。年份可省略。


## 各字段的含义

| 字段          | 是否必填 | 允许值     | 特殊字符       | 备注            |
| ------------ | ---- | --------------- | ------------------ | ---------------- |
| Seconds      | 是    | 0–59            | *\* , -*        | 标准实现不支持此字段。                                                              |
| Minutes      | 是    | 0–59            | *\* , -*       |                                                                          |
| Hours        | 是    | 0–23            | *\* , -*       |                                                                          |
| Day of month | 是    | 1–31            | *\* , - \? L W* | 只有部分软件实现 *? L W*                                                       |
| Month        | 是    | 1–12 or JAN–DEC | *\* , -*       |                                                                          |
| Day of week  | 是    | 0–7 or SUN–SAT  | *\* , - ? L \#* | 只有部分软件实现 *? L \#* <br>Linux 和 Spring 的允许值为 0-7，0和7为周日<br>Quartz 的允许值为 1-7，1 为周日 |
| Year         | 否    | 1970–2099       | *\* , -*       | 标准实现不支持此字段。|


## 特殊字符
1. **\* (all)**：**匹配该域的任意值**。如在Minutes域使用*, 即表示每分钟都会触发事件。
2. **\?  (any)**：*只能用在 DayofMonth 和 DayofWeek 两个域，作为占位符*。因为 DayofMonth 和 DayofWeek 互斥，二者*至少有一个表示为 \?*。
3. **\–  (range)**：**范围**。例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次 
4. **/  (increments)**：**起始时间/触发间隔**。从起始时间开始每隔固定时间触发一次。例如在Minutes域使用5/20,则意味着第 5 分钟触发一次，之后每隔 20 分钟触发一次。 
5. **\, (values)**：**列出枚举值**。例如：在Minutes域使用 5,20，则意味着在5和20分每分钟触发一次。 
6. **L (last)**： **表示最后**，只能用于 DayofWeek 和 DayofMonth 域。如果在 DayofWeek 域使用5L，意味着在最后的一个周四(数字5)触发。 
7. **W (weekday)**: **离指定 DayOfMonth 最近的工作日(周一到周五)，只能出现在DayofMonth域**。例如：在 DayofMonth 使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发。另外一点，**W 的最近寻找不会跨过月份** 。
8. **#**: **确定每个月第几个星期几，只能出现在DayofMonth 域**。例如在4#2，表示某月的第二个星期三。


# 问题
- cron 频率表达式，会从距离当前时间最近的计划执行时间开始执行。
- 如当前时间为 11：23， 表达式 *0 0/10 * * * ?*；
- 下次执行时间并不是 11:33，而是距离最近的十分钟整点数：11:30 、11:40、11:50....
- 因为 0/10 表达：从 0 开始，每 10 分执行一次。
- 开始时间优先级更高：
	- 如 *0/18* : 执行序列 17:54:00  18:00:00 18:18:00。
	- 17:54 + 18 = 18:12，但却在 18:00 执行。

## 解决
- 目标：
	- 设定起始时间和固定间隔；
	- 当跨越零点的
	- *固定的间隔，且在*，可将时间*分解成两个因子*， 如每 33 秒执行，将 33 可分为 3 X 11；
- 分解成*每 3S 执行*一次， 然后*用当前时间戳（秒数）对频率 3 取整，再对 11 取余， 如果为 0 就执行*。 
- 43，分解成 1 X 43：1 秒执行一次，对 1 取整然后对 43 取余。


```java
// 每43s执行一次
@Scheduled(cron = "0/1 * * * * ?")
public void test() {
    final long l = System.currentTimeMillis() / 1000  / 1;
    if (l % 43 == 0) {
	    // 执行代码
        log.info("=============");
    }
}

// 每33s执行一次
@Scheduled(cron = "0/3 * * * * ?")
public void test() {
    final long l = System.currentTimeMillis() / 1000 / 3;
    if (l % 11 == 0) {
	    // 执行代码
        log.info("=============");
    }
}
```

## Spring Schedule

[[Spring Schedule]]


# 参考
1. [cron - ArchWiki](https://wiki.archlinux.org/title/Cron)