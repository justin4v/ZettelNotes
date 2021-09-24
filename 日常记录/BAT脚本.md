# Conception
bat 脚本实际是 DOS下的批处理脚本（Batch Script File ），是将一系列 [[#DOS]] 命令按顺序排列而形成的集合，运行在 windows 命令行环境上。

# 基本语法
batch 脚本语法特点：
1. 大小写不敏感，不论是变量名还是命令，大小写是一样的；
2. 命令的前后空格会被忽略，set 变量的前后尽量不要有空格，set 字符值会将空格当成值。

## 注释
- rem 标记注释语句

```batch
rem 这是注释语句
::  这也是注释
echo hello
```


## 输出
```batch
echo hello world
```
将会输出 “hello world”

## 变量
使用 `SET` 命令定义变量并为赋值
```batch
SET name=justin
```

### SET /P
用于提示用户输入
```batch
set /p name=请输入你的名字:
```


# DOS

- DOS（Disk Operating System）是一款早期的操作系统；
- 主要面向磁盘操作；
- 最早由 Microsoft 为 IBM 计算机开发；
- 图形界面OS windows NT 出现后，DOS 作为系统的一个应用程序出现，通过 CMD 进入。