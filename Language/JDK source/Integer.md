## Cache
**KEYS**
- Java® Language Specification 要求 IntegerCache 默认支持 [-128,127]的缓存；
- 可由JVM选项 -XX:AutoBoxCacheMax=<size> 控制最大值；
- 最小值不变（-128）。
- static 代码块，JVM加载时初始化。
	


