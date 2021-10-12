# 结构
基于栈的特点：
- Hotspot 是**基于栈**的，而不是基于寄存器的；
 - **可移植性更好、指令更短**、实现起来简单；
- 不能随机访问栈中的元素，完成相同功能所需要的指令数更多，需要频繁出入栈。

基于寄存器的特点
- 速度快，有利于程序运行速度的优化；
- 操作数需要显式指定，指令也比较长。


# 字节码指令集概述

-   Java字节码对于虚拟机，就好比汇编语言对于计算机，属于基本执行指令。
-   Java虚拟机的指令由`一个字节长度`的、代表着某种特定操作含义的数字（称为`操作码，Opcode`）以及跟随其后的零至多个代表此操作所需参数（称为`操作数，Operands`）而构成。由于Java虚拟机采用面向操作数栈而不是寄存器的结构，所以大多数的指令都不包含操作数，只有一个操作码。
-   比如 **aload_0** 就是只有操作码没有操作数，而**invokespecial #1** 就是由操作数和操作码构成。
-   由于限制了Java虚拟机操作码的长度为一个字节（0 ~ 255），这意味着指令集的操作码总数不可能超过256条。

## 字节码与数据类型

对于大部分与数据类型相关的字节码指令，`它们的操作码助记符中都有特殊的字符来表明专门为哪种数据类型服务`

-   i代表int类型
-   l代表long
-   s代表short
-   b代表byte
-   c代表char
-   f代表float
-   d代表double

也有一些指令的助记符中没有明确地指明操作类型的字母，如arraylength指令，他没有代表数据类型的特殊字符，但操作数永远只能是一个数组类型的对象。

还有另外一些指令，如无条件跳转指令goto则是与数据类型无关的。

大部分的指令都没有支持整数类型byte、char和short，甚至没有任何指令支持boolean类型。编译器会在编译器或运行期将byte和short类型的数据带符号扩展（Sign-Extend）为相应的int类型数据，将boolean和char类型数据零位扩展（Zero-Extend）为相应的int类型数据。与之类似，在处理boolean、byte、short和char类型的数组时，也会转换为使用对应的int类型的字节码指令来处理。因此，大多数对于boolean、byte、short和char类型数据的操作，实际上都是使用相应的int类型作为运算类型（Computational Type）。


# 加载与存储指令

**作用**

加载和存储指令用于将数据从栈帧的局部变量表和操作数栈之间来回传递。

**常用指令**

-   **【局部变量压栈指令】**将一个局部变量加载到操作数栈：xload、xload_（其中x为i、l、f、d、a，n为0到3）**
-   **【常量入栈指令】**将一个常量加载到操作数栈：bipush、sipush、ldc、ldc_w、ldc2_w、aconst_null、iconst_ml、iconst、lconst、fconst、dconst**
-   **【出栈装入局部变量表指令】**将一个数值从操作数栈存储到局部变量表：xstore、xstore_（其中x为i、l、f、d、a，n为0到3）；xastore（其中x为i、1、f、d、a、b、c、s）**

上面所列举的指令助记符中，有一部分是以尖括号结尾的（例如iload）。这些指令助记符实际上代表了一组指令（例如iload代表了iload0、iload1、iload2和iload3这几个指令）。这几组指令都是某个带有一个操作数的通用指令（例如iload）的特殊形式，**对于这若干组特殊指令来说，它们表面上没有操作数，不需要进行取操作数的动作，但操作数都隐含在指令中。**

例如：
```java
iload_0：将局部变量表中索引为0位置上的数据压入操作数栈中。
iload 0：将局部变量表中索引为0位置上的数据压入操作数栈中。
iload 4：将局部变量表中索引为4位置上的数据压入操作数栈中。
```

前两种所表达的意思是相同的，不过iload_0相当于是只有操作码所以只占用1个字节，而iload 0 是操作码和操作数所组成的，而操作数占 2 个字节，所以占用3个字节。

默认最多只有0-3。

![[iload指令.png]]

## 局部变量压栈指令

-   **局部变量压栈指令将给定的局部变量表中的数据压入操作数栈**。
-   这类指令大体可以分为：
	-   xload_\<n>（x为i、1、f、d、a，n为0到3
	-   xload（x为i、1、f、d、a）
	-  说明：在这里，x的取值表示数据类型。
-   指令xload_n表示将第n个局部变量压入操作数栈，比如iload_1、fload_0、aload_0等指令。其中aload_n表示将一个对象引用压栈。
-   指令xload通过指定参数的形式，把局部变量压入操作数栈，当使用这个命令时，表示局部变量的数量可能超过了4个，比如指令iload、fload等。
	
```java
// 1 局部变量入栈命令
public void load(int num,Object obj,long count,boolean flag,short[] arr){
        System.out.println(num);
        System.out.println(obj);
        System.out.println(count);
        System.out.println(flag);
        System.out.println(arr);
}
```

字节码如下：
![[load()的字节码.png]]

所对应的局部变量表

![[load()的局部变量表.png]]

## 常量入栈指令

常量入栈指令的功能是将常数压入操作数栈，根据数据类型和入栈内容的不同，又可以分为const系列、push系列和ldc指令。

### 指令const系列
用于对特定的常量入栈，入栈的常量隐含在指令本身里。指令有：`iconst_<i>`（i从-1到5）、`lconst_<1>`（1从0到1）、`fconst_<f>`（f从0到2）、`dconst_<d>`（d从0到1）、`aconst_null`。

-   比如:
    
    -   iconst_m1将-1压入操作数栈；
    -   iconst_x（x为0到5）将x压入栈；
    -   lconst_0、lconst_1分别将长整数0和1压入栈；
    -   fconst_0、fconst_1、fconst_2分别将浮点数0、1、2压入栈；
    -   dconst_0和dconst_1分别将double型0和1压入栈；
    -   aconst_null将null压入操作数栈；
-   从指令的命名上不难找出规律，指令助记符的第一个字符总是喜欢表示数据类型，i表示整数，l表示长整数，f表示浮点数，d表示双精度浮点，习惯上用a表示对象引用。如果指令隐含操作的参数，会以下划线形式给出。

> const_x，是变量值，并且是有范围，比如int大于5，就要使用push系列

### 指令push系列
主要包括**bipush**和**sipush**。它们的区别在于接收数据类型的不同，bipush接收8位整数作为参数，sipush接收16位整数，它们都将参数压入栈。

### 指令ldc系列
如果以上指令都不能满足需求，那么可以使用万能的**ldc**指令，它可以接收一个8位的参数，该参数指向常量池中的int、float或者String的索引，将指定的内容压入堆栈。

-   类似的还有ldc_w，它接收两个8位数，能支持的索引范围大于ldc。
-   如果要压入的元素是long或者double类型的，则使用**ldc2_w**指令，使用方式都是类似的。

### 代码示例
```java
public void pushConstLdc() {
        int i = -1;
        int a = 5;
        int b = 6;
        int c = 127;
        int d = 128;
        int e = 32767;
        int f = 32768;
}
```
所对应的字节码指令如下：

![[pushConstLdc()的字节码.png]]

### 总结
![[常量入栈指令总结.png]]

## 出栈装入局部变量表

出栈装入局部变量表指令用于将操作数栈中栈顶元素弹出后，装入局部变量表的指定位置，用于给局部变量赋值。

这类指令主要以store的形式存在，比如xstore（x为i、l、f、d、a）、xstore_n（x为i、l、f、d、a，n为0至3）。

-   其中，指令istore_n将从操作数栈中弹出一个整数，并把它赋值给局部变量**索引**n位置。
-   指令xstore由于没有隐含参数信息，故需要提供一个byte类型的参数类指定目标局部变量表的位置。

说明：

-   `一般说来，类似像store这样的命令需要带一个参数，用来指明将弹出的元素放在局部变量表的第几个位置`。但是，**为了尽可能压缩指令大小，使用专门的istore_1指令表示将弹出的元素放置在局部变量表第1个位置**。类似的还有istore_0、istore_2、istore_3，它们分别表示从操作数栈顶弹出一个元素，存放在局部变量表第0、2、3个位置。
-   由于局部变量表前几个位置总是非常常用，`因此这种做法虽然增加了指令数量，但是可以大大压缩生成的字节码的体积`。如果局部变量表很大，需要存储的槽位大于3，那么可以使用istore指令，外加一个参数，用来表示需要存放的槽位位置。

### 代码示例
```java
public void store(int k, double d) {
        int m = k + 2;
        long l = 2;
        String str = "jack";
        float f = 10.0F;
        d = 10;
}
```

对应的字节码指令和局部变量表：

![[store()字节码和局部变量表.png]]

# 算术指令

## 作用

算术指令用于对两个操作数栈上的值进行某种特定运算，并把结果重新压入操作数栈。

## 分类

大体上算术指令可以分为两种：对`整型数据`进行运算的指令和对`浮点类型数据`进行运算的指令。

## byte、short、char和boolean类型说明

在每一大类中，都有针对Java虚拟机具体数据类型的专用算术指令。但没有直接支持byte、short、char和boolean类型的算术指令，对于这些数据的运算，都使用int类型的指令来处理。此外，在处理boolean、byte、short和char类型的数组时，也会转换为使用对应的int类型的字节码指令来处理。

## 运算时的溢出

数据运算可能会导致溢出，例如两个很大的正整数相加，结果可能是一个负数。其实Java虚拟机规范并无明确规定过整型数据溢出的具体结果，仅规定了在处理整型数据时，只有除法指令以及求余指令中当出现除数为0时会导致虚拟机抛出异常ArithmeticException。

## 运算模式

-   向最接近数舍入模式：JVM要求在进行浮点数计算时，所有的运算结果都必须舍入到适当的精度，非精确结果必须舍入为可被表示的最接近的精确值，如果有两种可表示的形式与该值一样接近，将优先选择最低有效位为零的；(类似四舍五入)
-   向零舍入模式：将浮点数转换为整数时，采用该模式，该模式将在目标数值类型中选择一个最接近但是不大于原值的数字作为最精确的舍入结果；(类似取整)

## NaN值使用

当一个操作产生溢出时，将会使用有符号的无穷大表示，如果某个操作结果没有明确的数学定义的话，将会使用NaN值来表示。而且所有使用NaN值作为操作数的算术操作，结果都会返回NaN；(Infinity无穷大)

public void test() {
        int i = 10;
        double j = i / 0.0;
        System.out.println(j); // Infinity

        double d1 = 0.0;
        double d2 = d1 / 0.0;
        System.out.println(d2); // NaN
}

## 所有算术指令

-   加法指令：iadd、ladd、fadd、dadd
-   减法指令：isub、lsub、fsub、dsub
-   乘法指令：imul、lmul、fmul、dmul
-   除法指令：idiv、ldiv、fdiv、ddiv
-   求余指令：irem、lrem、frem、drem
-   取反指令：ineg、lneg、fneg、dneg
-   自增指令：iinc
-   比较指令：dcmpg、dcmpl、fcmpg、fcmpl、lcmp
-   位运算指令：
    
    -   位移指令：ishl、ishr、iushr、lshl、lshr、lushr
    -   按位或指令：ior、lor
    -   按位与指令：iand、land
    -   按位异或指令：ixor、lxor

## 代码示例
```java
public void method2() {
        float i = 10;
        float j = -i;
        i = -j;
}
```

对应字节码如下：
![[method2（）字节码.png]]

## 比较指令的说明

-   比较指令的作用是比较栈顶两个元素的大，并将比较结果入栈。
-   比较指令有：dcmpg，dcmpl、fcmpg、fcmpl、lcmp。
    -   与前面讲解的指令类似，首字符d表示double类型，f表示float，l表示long。
-   对于double和float类型的数字，由于NaN的存在，各有两个版本的比较指令。以float为例，**有fcmpg和fcmpl两个指令，它们的区别在于在数字比较时，若遇到NaN值，处理结果不同。**
-   指令dcmpl和dcmpg也是类似的，根据其命名可以推测其含义，在此不再赘述。
-   指令lcmp针对long型整数，由于long型整数没有NaN值，故无需准备两套指令。