#Skills
# Utils
- utils 只作为工具类，不应该依赖 "有状态" 的 Bean；
- 但是有时候又需要作为 Util

## RedisUtil的做法
- 依赖 `redisTemplate` Bean;

```java
public class RedisUtil {
    private RedisUtil(){}

	// 导入
    @SuppressWarnings("unchecked")
    private static RedisTemplate<String, Object> redisTemplate = SpringContextUtil.getBean("redisTemplate",RedisTemplate.class);

    /**
     * 设置有效时间
     * 单位默认秒
     *
     * @param key     Redis键
     * @param timeout 超时时间
     * @return true=设置成功；false=设置失败
     */
    public static boolean expire(final String key, final long timeout) {

        return expire(key, timeout, TimeUnit.SECONDS);
    }

```