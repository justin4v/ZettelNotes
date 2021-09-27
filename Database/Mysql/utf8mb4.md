# utf8
mysql 的 utf8 并不是真正的 UTF-8 ，Mysql的 utf-8 只支持每个字符最多三个字节，而真正的UTF-8是每个字符最多四个字节。
MySQL在2010年发布了一个叫作“utf8mb4” （mb4: most bytes 4 -- 最多 4 字节） 的字符集，绕过了这个问题。
MySQL从4.1版本开始支持UTF-8，也就是2003年，而今天使用的UTF-8标准（RFC 3629）是随后才出现的。

-   MySQL的“utf8mb4”是真正的“UTF-8”。
-   MySQL的“utf8”是一种“专属的编码”，它能够编码的Unicode字符并不多。




MySQL里实现的utf8最长使用3个字节，也就是只支持到了 Unicode 中的 [基本多文本平面）](https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84)U+0000至U+FFFF，包含了控制符、拉丁文，中、日、韩等绝大多数国际字符，但并不是所有，最常见的就算现在手机端常用的表情字符 emoji和一些不常用的汉字，如 “墅” ，这些需要四个字节才能编码出来。