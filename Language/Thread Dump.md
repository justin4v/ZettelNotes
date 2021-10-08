# Dump
Thread Dump 是一个 Java 工具。
Thread Dump 常用于诊断 java 问题。

thread dump 是一个文本文件，打开后可以看到每一个线程的执行栈，以stacktrace 的方式显示。
通过对多个 thread dump 的分析可以得到应用是否“卡”在某一点上，即在某一点运行的时间太长，如数据库查询，长期得不到响应，最终导致系统崩溃。

## Te