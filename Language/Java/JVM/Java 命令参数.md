#Java基础 #JVM #Command-param

# 描述
The `java` command supports a wide range of options that can be divided into the following categories:

-   [Standard Options](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html#BABDJJFI)
    
-   [Non-Standard Options](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html#BABHDABI)
    
-   [Advanced Runtime Options](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html#BABCBGHF)
    
-   [Advanced JIT Compiler Options](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html#BABDDFII)
    
-   [Advanced Serviceability Options](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html#BABFJDIC)
    
-   [Advanced Garbage Collection Options](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html#BABFAFAE)


# 形式
- Standard options are guaranteed to be supported by all implementations of the Java Virtual Machine (JVM). They are used for common actions, such as checking the version of the JRE, setting the class path, enabling verbose output, and so on.

- Non-standard options are general purpose options that are specific to the Java HotSpot Virtual Machine, so they are not guaranteed to be supported by all JVM implementations, and are subject to change. These options start with `-X`.

- Advanced options are not recommended for casual use. These are developer options used for tuning specific areas of the Java HotSpot Virtual Machine operation that often have specific system requirements and may require privileged access to system configuration parameters. They are also not guaranteed to be supported by all JVM implementations, and are subject to change. Advanced options start with `-XX`.

To keep track of the options that were deprecated or removed in the latest release, there is a section named [Deprecated and Removed Options](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html#BABDCEGG) at the end of the document.

- Boolean options are used to either enable a feature that is disabled by default or disable a feature that is enabled by default. Such options do not require a parameter. Boolean `-XX` options are enabled using the plus sign (`-XX:+`_OptionName_) and disabled using the minus sign (`-XX:-`_OptionName_).

- For options that require an argument, the argument may be separated from the option name by a space, a colon (:), or an equal sign (=), or the argument may directly follow the option (the exact syntax differs for each option). If you are expected to specify the size in bytes, you can use no suffix, or use the suffix `k` or `K` for kilobytes (KB), `m` or `M` for megabytes (MB), `g` or `G` for gigabytes (GB). For example, to set the size to 8 GB, you can specify either `8g`, `8192m`, `8388608k`, or `8589934592` as the argument. If you are expected to specify the percentage, use a number from 0 to 1 (for example, specify `0.25` for 25%).


# 参考
1. [java command parameter](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html)