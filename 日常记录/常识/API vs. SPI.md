# 概念
## API 
[Application Programming Interface](http://en.wikipedia.org/wiki/Application_programming_interface)

## SPI
 [Service Provider Interface](http://en.wikipedia.org/wiki/Service_Provider_Interface)


# 比较
-   The API is the description of classes/interfaces/methods/... that you **_call and use_** to achieve a goal ;
-   the SPI is the description of classes/interfaces/methods/... that you **_extend and implement_** to achieve a goal.

# 示例
JDBC 中，`Driver class` 是 SPI：用户使用 JDBC 时，不直接使用 `Driver class` ，但是开发人员实现 JDBC 驱动时，必须实现 `Driver class`。
