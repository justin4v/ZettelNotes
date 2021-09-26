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
rem 输出echo命令行和“hello world”
echo hello world

rem 只输出“hello world”
@echo hello world

rem 打开（以后的）命令行回显，但是本命令行回显
echo on

rem 关闭（以后的）命令行回显，但是本命令行回显
echo off
```

使用 '@' 关闭本命令行回显

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
用户会看到“请输入你的名字:”的提示，用户输入后，会赋值给name 变量。

### SET /A
可进行表达式运算后赋值
```batch
set /a sum=2+2+2
```
结果 a=6

### 访问变量
使用 *%varName%* 访问变量
 

## pause
```batch
pause 等待并输出"请按任意键继续. . ."
pause > nul 等待但不输出提示语（输出到名为 null 的文件）
echo wait a moment.. & pause > nul 输出"wait a moment.."并等待操作
```

*pause 命令可以保持当前窗口不会立即关闭*


## errorlevel
程序返回码：
- *成功返回 0；*
- *失败返回为 1*。

## start
打开一个单独的窗口以运行指定的程序或命令，*主程序继续向下执行*
```batch
start [command/program] [parameters]
```


## exit
退出CMD.EXE程序或当前批处理脚本
```batch
EXIT [/B] [exitCode]
```

*'/B'参数指定退出当前 batch 脚本*。
直接执行
```batch
rem 退出 CMD 程序并标记为成功状态
exit 0
rem 退出 CMD 程序并标记为失败状态
exit 1
```

## cls
清除当前屏幕

## help
查看帮助信息
```batch
help [command]
```


# 文件操作命令

## copy
```batch
rem 将文件file1.txt复制到temp2目录，有相同文件提示
copy d:\temp1\file1.txt d:\temp2 

rem 将文件复制到目录,有相同文件覆盖原文件，不提示
copy d:\temp1\file1.txt d:\temp2 /y  

rem 将目录下的所有文件复制到目录,有相同文件覆盖原文件，不提示
copy d:\temp1\* d:\temp2 /y 
```


## xcopy
```batch
rem 将temp1目录下的文件复制到temp2目录，不包括temp1子目录下的文件
xcopy temp1 d:\temp2 /y  

rem 将目录下的文件复制到目录，包括子目录下的文件
xcopy temp1 d:\temp2 /s /e /y 
```

xcopy 可以忽略子目录

## type 
显示文件内容命令
```batch
rem 查看file1文件内容
type file1.txt  
type file1.txt file2.txt  


rem 将file1.txt文件内容重定向到file2.txt
type file1.txt > file2.txt   

rem 创建文件
type nul > file1.txt 
```

## ren 
Rename 重命名文件命令
```batch
rem 修改file1.txt文件名为file2.txt
ren d:\temp1\file1.txt file2.txt 
```

## del 
Delete 删除文件命令
```batch
rem 删除temp目录下的file1.txt文件
del d:\temp1\file1.txt  

rem 删除temp目录下的后缀为.txt的文件
del d:\temp\*.txt  
```

# 目录操作命令

##  cd 
Change Directory 切换目录

```batch
:: 切换到temp1目录，当前目录是d盘
cd d:\temp1 

:: 切换到d盘，然后切换到temp1目录，当前目录非d盘
cd /d d:\temp1  

:: 切换到上一级目录
cd .. 
```

## mkdir 
Make Directory 创建目录

```batch
:: 在当前目录下创建test目录
mkdir test  

:: 在temp1目录下创建test目录；
:: 如果temp1目录不存在，先自动创建temp1，再创建test目录
mkdir d:\temp1\test 
```

## rmdir
Remove Directory 删除目录
```batch
:: 删除空目录temp1，非空则删除失败 
rmdir d:\temp1 

:: 删除temp1目录，包括子目录(/s)，并且删除时不提示(/q)
rmdir d:\temp1 /s /q 
```


## dir
Directory 显示目录下的子目录和文件

```batch
:: 显示temp1目录下的文件和目录信息
:: 显示信息包含日期、时间、文件类型和文件名
dir d:\temp1 

:: 只显示temp1目录下（不包括子目录）的文件的绝对路径
:: 不显示日期、时间、文件类型和文件名
dir d:\temp1 /a:a /b 

:: 显示temp1路径下（包括子目录）的所有文件的绝对路径。
:: 输出文件按照文件名数字顺序排序 
dir d:\temp1 /b /s /o:n /a:a 

:: 显示.txt后缀文件，并且按照文件名顺序排序(/on)
:: 其他排序方法查看help dir
dir d:\temp1\*.txt /a:a /b /o:n 
```
 
 说明：
- /b表示去除摘要信息，仅显示完整路径;
- /s表示循环列举文件夹中的内容;
- /o:n 表示根据文件名排序;
- /a:a 表示只枚举文件而不枚举其他;
- 单独 dir /b 与 dir /s 都不会显示完整路径，只有这两个组合才会显示完整路径。

# 控制流

## if/else
```batch
IF [NOT] ERRORLEVEL number command
IF [NOT] string1==string2 command
IF [NOT] EXIST filename command

rem 扩展

::  /I 说明字符串比较不分大小写
IF [/I] string1 compare-op string2 command
IF CMDEXTVERSION number command

rem 变量是否被定义
IF DEFINED variable command
```

说明：
1. NOT: 逻辑非；
2. ERRORLEVEL number：如果最后运行的程序返回*一个等于或大于number*的退出代码，条件为 true；
3. string1\==string2：字符串相等；
4.  EXIST filename：如果指定的文件名存在，指定条件为 true；
5.  compare-op：EQU - 等于、NEQ - 不等于、LSS - 小于、LEQ - 小于或等于、GTR - 大于、GEQ - 大于或等于；

 ## for
 
 ```batch
 FOR %variable IN (set) DO command [command-parameters]
 ```
 
 说明：
 1. \%variable  指定一个单一字母可替换的参数。注意：批处理脚本中使用 \%%variable；
 2. (set)  指定一个或一组文件。可以使用通配符；
 3. command  指定对每个文件执行的命令；
 4. command-parameters 为特定命令指定参数或命令行开关。
 
  
  for语句还有4个参数，*指定 (set) 的形式和匹配规则*，分别是 /d /r /l /f
  
  ### /d
  /d: Directory set 中与目录名匹配
  
实例：打印C盘根目录下的目录名
   
```batch
@echo off
for /d %%i in (c:/*) do (
  echo %%i 
  )
pause
```


### /r
/r: recursive 递归查询指定目录下的匹配文件

实例：打印D盘目录及子目录下的后缀为.txt和.py的文件
```batch
@echo off
for /r d:/temp %%i in ( *.txt *.py ) do (
  echo %%i 
  )
pause
```


### /L
/L: list 表示以增量形式从开始到结束的一个数字序列。
(1,1,5) 将产生序列1 2 3 4 5，(5,-1,1)将产生序列(5 4 3 2 1)

实例：打印10以内的奇数
```batch
@echo off
for /l %%i in (1,2,10) do (
  echo %%i 
  )
pause
```


### /f
三种形式:

```batch
FOR /F ["options"] %variable IN (file-set) DO command [command-parameters] 

FOR /F ["options"] %variable IN ("string") DO command [command-parameters] 

FOR /F ["options"] %variable IN ('command') DO command [command-parameters]
```

 说明：可以处理
 - 文件内容（file-set）、
 - 字符串("string") 以及执行指定命令('command') 返回回的值。

# DOS

- DOS（Disk Operating System）是一款早期的操作系统；
- 主要面向磁盘操作；
- 最早由 Microsoft 为 IBM 计算机开发；
- 图形界面OS windows NT 出现后，DOS 作为系统的一个应用程序出现，通过 CMD 进入。