# utf8
-   MySQL的“utf8mb4”是真正的“UTF-8”。
-   MySQL的“utf8”是一种“专属的编码”，它能够编码的Unicode字符并不多。
## Mysql 的 utf8
*mysql 的 utf8 不是真正的 UTF-8*：Mysql的 utf-8 只支持每个字符最多三个字节，而真正的 UTF-8 是每个字符最多四个字节。


### 限制
Mysql的 utf8 只支持到了 Unicode 中的 [基本多文本平面](https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84) U+0000至U+FFFF，包含了控制符、拉丁文，中、日、韩等绝大多数国际字符，但常用的表情字符 emoji 和一些不常用的汉字，如 “墅” 等无法编码。


### 原因
MySQL从*4.1 版本* 开始支持 UTF-8，也就是2003年，而今天使用的 UTF-8 标准（RFC 3629）是随后才出现的。


# utf8mb4
MySQL在2010年发布了 *utf8mb4 （mb4: most bytes 4 -- 最多 4 字节）* 字符集，实现了标准的 utf8 编码。

