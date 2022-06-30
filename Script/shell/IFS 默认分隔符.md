#Linux #IFS

# 简介
在Window中，我们对文件命名时候经常使用用空格，例如目录“meda a”。但是这种命名方式给Linux命令行工具和Shell带来了困扰，因为大多数命令中，都是默认以空格做为值与值之间的分隔符，而不是做为文件名的一部分。即环境变量IFS（the Internal Field Separator）为值为 空格、Tab、回车。

先来看下IFS的man page,
IFS: The Internal Field Separator that is used for word splitting after expansion and to split lines into words with the read built-in command. The default value is “<space><tab><new-line>”.




# 参考
1. [Linux shell 技巧：对文件名中包含空格的处理方法](https://blog.csdn.net/keypeople/article/details/78147288)