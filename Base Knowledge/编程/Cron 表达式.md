*Cron expressions* are used to configure instances of CronTrigger, a subclass of org.quartz.Trigger. 
A cron expression is a string consisting of *six or seven subexpressions* (fields) that describe individual details of the schedule.

# 语法
1. *Seconds Minutes Hours DayofMonth Month DayofWeek Year*
2. *Seconds Minutes Hours DayofMonth Month DayofWeek*
3. *秒 分 小时 月份中的日期 月份 星期中的日期 年份*。年份可省略。


## 各字段的含义
| 字段               | 允许值                   | 允许的特殊字符  |
| ------------------ | ------------------------ | --------------- |
| 秒（Seconds）      | 0~59                     | , - * /         |
| 分（Minutes）      | 0~59                     | , - * /         |
| 小时（Hours）      | 0~23                     | , - * /         |
| 日期（DayofMonth） | 1~31                     | ,- * ? / L W C  |
| 月份（Month）      | 1~12 or JAN-DEC          | , - * /         |
| 星期（DayofWeek）  | 1~7 or SUN(1)-SAT(7) | , - * ? / L C # |
| 年(可选)（Year）   | 1970~2099                | , - * /         |



## 特殊字符
1. **\*(all)**：**匹配该域的任意值**。如在Minutes域使用*, 即表示每分钟都会触发事件。
2. **\? (any)**：*只能用在 DayofMonth 和 DayofWeek 两个域，作为占位符*。因为 DayofMonth 和 DayofWeek 互斥，二者*至少有一个表示为 \?*。
3. **\– (range)**：**范围**。例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次 
4. **/ (increments)**：**起始时间/触发间隔**。从起始时间开始每隔固定时间触发一次。例如在Minutes域使用5/20,则意味着第 5 分钟触发一次，之后每隔 20 分钟触发一次。 
5. **\, (values)**：**列出枚举值**。例如：在Minutes域使用 5,20，则意味着在5和20分每分钟触发一次。 
6. **L (last)**：**表示最后**，只能用于 DayofWeek 和 DayofMonth 域。如果在 DayofWeek 域使用5L，意味着在最后的一个周四(数字5)触发。 
7. **W (weekday)**:离指定 DayOfMonth 最近的工作日(周一到周五)，只能出现在DayofMonth域。例如：在 DayofMonth 使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发。另外一点，**W 的最近寻找不会跨过月份** 。
8. **#**:用于确定每个月第几个星期几，只能出现在DayofMonth 域。例如在4#2，表示某月的第二个星期三。