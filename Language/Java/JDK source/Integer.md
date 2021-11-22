#SourceCode #JDK
# 理解
1. primitive 类型 `int`的包装类；
2. 内部使用 `int` 存储 `value`;
3. 4 byte, 32 bit；
4. method 可被 `Byte`/`Short`复用；

# Learing
1. 使用 cache，提高响应速度，节省空间；
2. 属于对象的 method 无需外部参数；
3. 属于 Class 的 method 可传入参数。


# Cache
- Java® Language Specification 要求 IntegerCache 默认支持 [-128,127]的缓存；
- 可由JVM选项 -XX:AutoBoxCacheMax=[size] 控制最大值；
- 最小值不变（-128）。
- static 代码块，使用 Integer 类时 JVM 初始化 IntegerCache。


	


