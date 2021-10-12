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
    -   首字符d表示double类型，f表示float，l表示long。
-   对于double和float类型的数字，由于NaN的存在，各有两个版本的比较指令。以float为例，**有fcmpg和fcmpl两个指令，它们的区别在于在数字比较时，若遇到NaN值，处理结果不同: **
	-   遇到NaN值， fcmpg会压入1
	-   fcmpl会压入-1
-   指令dcmpl和dcmpg也是类似的，根据其命名可以推测其含义，在此不再赘述。
-   指令lcmp针对long型整数，由于long型整数没有NaN值，故无需准备两套指令。

# 类型转换指令

## 宽化类型转换转换规则

Java虚拟机直支持以下数值的**宽化类型转换**（widening numeric conversion，**小范围类型向大范围类型**的安全转换）。也就是说，并不需要指令执行，包括：

-   从int类型到long、float或者double类型。对应的指令为：i2l、i2f、i2d
-   从long类型到float、double类型。对应的指令为：l2f、l2d
-   从float类型到double类型。对应的指令为：f2d

> 简化为：int–>long–>float–>double

// 宽化类型转换
public void test() {
    int i = 10;
    long l = i; // i2l
    float f = i; // i2f
    double d = i; // i2d

    float f1 = l; // l2f
    double d1 = l; // l2d
    double d2 = f1; // f2d
}

### 精度损失问题

-   宽化类型转换是不会因为超过目标类型最大值而丢失信息的，例如，从int转换到long，或者从int转换到double，都不会丢失任何信息，转换前后的值是精确相等的。
-   从int、long类型数值转换到float，或者long类型数值转换到double时，将可能发生精度丢失——可能丢失掉几个最低有效位上的值，转换后的浮点数值是根据IEEE754最接近舍入模式所得到的正确整数值。
-   尽管宽化类型转换实际上是可能发生精度丢失的，但是这种转换永远不会导致Java虚拟机抛出运行时异常。

### 代码示例：
```java
 public void upCast2() {
        int i = 123123123;
        float f = i;
        System.out.println(f); // 1.2312312E8 = 123123120 精度丢失
        long l = 123123123123123123L; //1.2312312312312312E17 
        double d = l; //123123123123123120 精度丢失
        System.out.println(d);
}
```

### 补充说明

**从byte、char和short类型到int类型的宽化类型转换实际上是不存在的**。对于byte类型转为int，虚拟机并没有做实质性的转化处理，只是简单地通过操作数栈交换了两个数据。而将byte转为long时，使用的是i2l，可以看到在内部byte在这里已经等同于int类型处理，类似的还有short类型，这种处理方式有两个特点：

-   一方面可以减少实际的数据类型，如果为short和byte都准备一套指令，那么指令的数量就会大增，而虚拟机目前的设计上，只愿意使用一个字节表示指令，因此指令总数不能超过256个，为了节省指令资源，将short和byte当做int处理也在情理之中。
-   另一方面，由于局部变量表中的槽位固定为32位，无论是byte或者short存入局部变量表，都会占用32位空间。从这个角度说，也没有必要特意区分这几种数据类型。

## 窄化类型转换(强制转换)规则

Java虚拟机也直接支持以下**窄化类型转换**：

-   从int类型至byte、short或者char类型。对应的指令有：i2b、i2s、i2c
-   从long类型到int类型。对应的指令有：l2i
-   从float类型到int或者long类型。对应的指令有：f2i、f2l
-   从double类型到int、long或者float类型。对应的指令有：d2i、d2l、d2f

### 代码示例
```java
public void downCastl() {
        int i = 10;
        byte b = (byte) i; // i2b
        short s = (short) i; // i2s
        char c = (char) i; // i2c
        long l = 10L;
        int il = (int) l; // l2i
        byte b1 = (byte) l; // l2i i2b
    }

public void downCast2() {
    float f = 10;
    long l = (long) f; // f2l
    int i = (int) f; // f2i
    byte b = (byte) f; // f2i i2b 
    double d = 10;
    byte b1 = (byte) d; // d2i i2b
}
```

### 精度损失问题

-   窄化类型转换可能会导致转换结果具备不同的正负号、不同的数量级，因此，转换过程很可能会导致数值丢失精度。
-   尽管数据类型窄化转换可能会发生上限溢出、下限溢出和精度丢失等情况，但是Java虚拟机规范中明确规定数值类型的窄化转换指令永远不可能导致虚拟机抛出运行时异常。

### 补充说明

-   当将一个浮点值窄化转换为整数类型T（T限于int或long类型之一）的时候，将遵循以下转换规则：
    
    -   如果浮点值是NaN，那转换结果就是int或long类型的0。
    -   如果浮点值不是无穷大的语浮点值使用IEEE754的向零舍入模式取整，获得整数值v，如果v在目标类型T（int或long）的表示范围之内，那转换结果就是v。否则，将根据v的符号，转换为T所能表示的最大或者最小正数
-   当将一个double类型窄化转换为float类型时，将遵循以下转换规则：通过向最接近数舍入模式舍入一个可以使用float类型表示的数字。最后结果根据下面这3条规则判断：
    
    -   如果转换结果的绝对值太小而无法使用float来表示，将返回float类型的正负零。
    -   如果转换结果的绝对值太大而无法使用float来表示，将返回float类型的正负无穷大。
    -   对于double类型的NaN值将按规定转换为float类型的NaN值。


# 对象的创建与访问指令

Java是面向对象的程序设计语言，虚拟机平台从字节码层面就对面向对象做了深层次的支。有一系列指令专门用于对象操作，可进一步细分为创建指令、字段访问指令、数组操作指令、类型检查指令。

## 创建指令

-   虽然类实例和数组都是对象，但Java虚拟机对类实例和数组的创建与操作使用了不同的字节码指令：

### 创建类实例的指令
    
  创建类实例的指令：**new**
  
  它接收一个操作数，为指向常量池的索引，表示要创建的类型，执行完成后，将对象的引用压入栈。
  
### 创建数组的指令
    
 创建数组的指令：**newarray、anewarray、multianewarray。**
 - newarray：创建基本类型数组
 - anewarray：创建引用类型数组
 - multianewarray：创建多维数组

上述创建指令可以用于创建对象或者数组，由于对象和数组在Java中的广泛使用，这些指令的使用频率也非常高。

### 代码示例
```java
// 创建对象
public void newInstance() {
    Object obj = new Object();

    File file = new File("Hello.txt");
}
```

对应的字节码如下：
![[newInstance()字节码.png]]

> dup是将栈顶数值复制一份并送入至栈顶。因为invokespecial会消耗掉一个当前类的引用，因而需要复制一份。

```java
// 创建数组
public void newArray() {
    int[] intArray = new int[10]; // newarray
    Object[] objArray = new Object[10]; // anewarray
    int[][] mintArray = new int[10][10]; // multianewarray
    String[][] strArray = new String[10][]; // newarray
    String[][] strArray2 = new String[10][5]; // multianewarray
}
```


## 字段访问指令

对象创建后，就可以通过对象访问指令获取对象实例或数组实例中的字段或者数组元素。

-   访问类字段（static字段，或者称为类变量）的指令：getstatic、putstatic
-   访问类实例字段（非static字段，或者称为实例变量）的指令：getfield、putfield

## 数组操作指令

数组操作指令主要有：xastore和xaload指令。

具体为：

-   把一个数组元素加载到操作数栈的指令：baload、caload、saload、iaload、laload、faload、daload、aaload
-   将一个操作数栈的值存储到数组元素中的指令：rbastore、castore、sastore、iastore、lastore、fastore、dastore、aastore
-   取数组长度的指令：arraylength
    
    -   该指令弹出栈顶的数组元素，获取数组的长度，将长度压入栈。

| 数组类型      | 加载指令 | 存储指令 |
| ------------- | -------- | -------- |
| byte(boolean) | baload   | bastore  |
| char          | caload   | castore  |
| short         | saload   | sastore  |
| int           | iaload   | iastore  |
| long          | laload   | lastore  |
| float         | faload   | fastore  |
| double        | daload   | dastore  |
| reference     | aaload   | aastore  |

> *虚拟机栈中并不存储数组元素信息，所以astore改变的是堆中的实例数组*

### 说明

-   指令xaload表示将数组的元素压栈，比如saload、caload分别表示压入short数组和char数组。指令xaload在执行时，要求操作数中栈顶元素为数组索引i，栈顶顺位第2个元素为数组引用a，该指令会弹出栈顶这两个元素，并将a[i]重新压入堆栈。
-   xastore则专门针对数组操作，以iastore为例，它用于给一个int数组的给定索引赋值。在iastore执行前，操作数栈顶需要以此准备3个元素：值、索引、数组引用，iastore会弹出这3个值，并将值赋给数组中指定索引的位置。

### 代码示例
```java
public void text() {
        int[] intArray = new int[10];
        intArray[3] = 20;
        System.out.println(intArray[1]);
}
```

字节码
![[text()字节码.png]]

## 类型检查指令

检查类实例或数组类型的指令：instanceof、checkcast。

-   指令cheqkcast用于检查类型强制转换是否可以进行。如果可以进行，那么checkcast指令不会改变操作数栈，否则它会抛出ClassCastException异常。|
-   指令instanceof用来判断给定对象是否是某一个类的实例，它会将判断结果压入操作数栈。
```java
//类型检查指令
public String checkcast(Object obj) {
    if (obj instanceof String) {
        return (String) obj;
    } else {
        return null;
    }
}
```

对应的字节码如下：
![[checkcast()字节码.png]]

# 方法的调用和返回指令

## 方法调用指令

方法调用指令：invokevirtual、invokeinterface、invokespecial、invokestatic、invokedynamic以下5条指令用于方法调用：

-   **invokevirtual**：指令用于调用对象的实例方法，根据对象的实际类型进行分派（虚方法分派），支持多态。这也是Java语言中**最常见的方法分派方式**。
-   **invokeinterface：**指令用于**调用接口方法**，它会在运行时搜索由特定对象所实现的这个接口方法，并找出适合的方法进行调用。
-   **invokespecial：**指令用于调用一些需要特殊处理的实例方法，包括**实例初始化方法（构造器）、私有方法和父类方法**。这些方法都是**静态类型绑定**的，不会在调用时进行动态派发。
-   **invokestatic：**指令用于调用命名类中`的类方法（static方法）`。这是**静态绑定**的。
-   **invokedynamic：**：调用动态绑定的方法，这个是JDK1.7后新加入的指令。用于在运行时动态解析出调用点限定符所引用的方法，并执行该方法。前面4条调用指令的分派逻辑都固化在java虚拟机内部，而invokedynamic指令的分派逻辑是由用户所设定的引导方法决定的。

> invokespecial 和invokestatic 都不可能重写

### 代码示例
```java
public void invoke() {
        //情况1：类实例构造器方法：<init>()
        Date date = new Date();
        Thread t1 = new Thread();
        //情况2：父类的方法
        super.toString();
        //情况3：私有方法
        methodPrivate();
    }
private void methodPrivate() {}
```

对应的字节码如下：
![[invoke()字节码.png]]

```java
//方法调用指令：invokestatic
public void invoke() {
    methodstatic();
    //0 invokestatic #2 <com/test/Demo.methodstatic>
    //3 return
}
private static void methodstatic() {}
```

```java
 //方法调用指令：invokeinterface
public void invoke() {
    Thread t1 = new Thread();

    ((Runnable) t1).run(); 
    Comparable<Integer> com = null;
    com.compareTo(123); 
}
```

对应的字节码如下：
![[invokeinterface 字节码.png]]

## 方法返回指令

方法调用结束前，需要进行返回。**方法返回指令是根据返回值的类型区分的**。

-   包括ireturn（当返回值是boolean、byte、char、short和int 类型时使用）、lreturn、freturn、dreturn和areturn
-   另外还有一条return 指令供声明为void的方法、实例初始化方法以及类和接口的类初始化方法使用。


| 返回类型                       | 返回指令 |
| ------------------------------ | -------- |
| void                           | return   |
| int  (boolean,byte,char,short) | ireturn  |
| long                           | lreturn  |
| float                          | freturn  |
| double                         | dreturn  |
| reference                      | areturn  |

**注意：**

-   通过ireturn指令，将当前函数操作数栈的顶层元素弹出，并将这个元素压入调用者函数的操作数栈中（因为调用者非常关心函数的返回值），所有在当前函数操作数栈中的其他元素都会被丢弃。
-   如果当前返回的是synchronized方法，那么还会执行一个隐含的monitorexit指令，退出临界区。
-   最后，会丢弃当前方法的整个帧，恢复调用者的帧，并将控制权转交给调用者。

### 代码示例

```java
public float returnFloat() {
    int i = 10;
    return i;
}
```

字节码
![[returnFloat()字节码.png]]

# 操作数栈管理指令

这类指令包括如下内容：

-   将一个或两个元素从栈顶弹出，并且直接废弃：pop，pop2；
-   复制栈顶一个或两个数值并将复制值或双份的复制值重新压入栈顶：dup，dup2，dup_×1，dup2_×1，dup_×2，dup2_×2；
-   将栈最顶端的两个slot数值位置交换：swap。Java虚拟机没有提供交换两个64位数据类型（ long、doub1e）数值的指令。
-   指令nop，是一个非常特殊的指令，它的字节码为exee。和汇编语言中的nop一样，它表示什么都不做。这条指令一般可用于调试、占位等。

**这些指令属于通用型，对栈的压入或者弹出无需指明数据类型。**

## 说明

-   不带x的指令是复制栈顶数据并压入栈顶。包括两个指令，dup和dup2。dup的系数代表要复制的Slot个数。
    
    -   dup开头的指令用于复制1个slot的数据。例如1个int或1个reference类型数据
    -   dup2开头的指令用于复制2个Slot的数据。例如1个long，或2个int，或1个int+1个float类型数据
-   带_x的指令是复制栈顶数据并插入栈顶以下的某个位置。共有4个指令_
-   > dup_×1，dup2_×1，dup_×2，dup2_×2
    
-   对于带_x的复制插入指令，只要将指令的dup和x的系数相加，结果即为需要插入的位置。因此
    
    -   dup_×1插入位置：1+1=2，即栈顶2个slot下面
    -   dup_×2插入位置：1+2=3，即栈顶3个slot下面
    -   dup2_×1插入位置：2+1=3，即栈顶3个Slot下面
    -   dup2_×2插入位置：2+2=4，即栈顶4个Slot下面
-   pop：将栈顶的1个slot数值出栈。例如1个short类型数值
-   pop2：将栈顶的2个slot数值出栈。例如1个double类型数值，或者2个int类型数值

# 控制转移指令

程序流程离不开条件控制，为了支持条件跳转，虚拟机提供了大量字节码指令，大体上可以分为：

1.  比较指令
2.  条件跳转指令
3.  比较条件转指令
4.  多条件分支跳转指令
5.  无条件跳转指令等

## 比较指令

-   比较指令的作用是比较栈顶两个元素的大，并将比较结果入栈。
-   比较指令有：dcmpg，dcmpl、fcmpg、fcmpl、lcmp。
    
    -   与前面讲解的指令类似，首字符d表示double类型，f表示float，l表示long。
-   对于double和float类型的数字，由于NaN的存在，各有两个版本的比较指令。以float为例，**有fcmpg和fcmpl两个指令，它们的区别在于在数字比较时，若遇到NaN值，处理结果不同。**当有1个值为NaN时， fcmpg指令会将1压入栈，而fcmpl值会将-1压入栈。
-   指令dcmpl和dcmpg也是类似的，根据其命名可以推测其含义，在此不再赘述。
-   指令lcmp针对long型整数，由于long型整数没有NaN值，故无需准备两套指令。

## 条件跳转指令

-   条件跳转指令通常和比较指令结合使用。在条件跳转指令执行前，一般可以先用比较指令进行栈顶元素的准备，然后进行条件跳转。
-   条件跳转指令有：ifeq，iflt，ifle，ifne，ifgt，ifge，ifnull，ifnonnull。这些指令都接收两个字节的操作数，用于计算跳转的位置（16位符号整数作为当前位置的offset）。
-   它们的统一含义为：**弹出栈顶元素，测试它是否满足某一条件，如果满足条件，则跳转到给定位置**。


| 指令      | 说明                             |
| --------- | -------------------------------- |
| ifeq      | 当栈顶int类型数值等于0时跳转     |
| ifne      | 当栈顶int类型数值不等于0时跳转   |
| iflt      | 当栈顶int类型数值小于0时跳转     |
| ifle      | 当栈顶int类型数值小于等于0时跳转 |
| ifgt      | 当栈顶int类型数值大于0时跳转     |
| ifge      | 当栈顶int类型数值大于等于0时跳转 |
| ifnull    | 为null时跳转                     |
| ifnonnull | 不为null时跳转                   |



注意：

-   与前面运算规则一致：
    
    -   对于boolean、byte、char、short类型的条件分支比较操作，都是使用int类型的比较指令完成
    -   对于long、float、double类型的条件分支比较操作，则会先执行相应类型的比较运算指令，运算指令会返回一个整型值到操作数栈中，随后再执行int类型的条件分支比较操作来完成整个分支跳转
-   由于各类型的比较最终都会转为int类型的比较操作，所以Java虚拟机提供的int类型的条件分支指令是最为丰富。

### 代码示例
```java
public void compare() {
        int a = 0;
        if (a == 0) {
            a = 10;
        } else {
            a = 20;
        }
}
```

字节码
![[compare()字节码.png]]

## 比较条件跳转指令

比较条件跳转指令类似于比较指令和条件跳转指令的结合体，它将比较和跳转两个步骤合二为一。

这类指令有：

-   if_icmpeq
-   if_icmpne
-   if_icmplt
-   if_icmpgt
-   if_icmple
-   if_icmpge
-   if_acmpeq
-   if_acmpne

其中指令助记符加上“if_”后，以字符“i”开头的指令针对int型整数操作（也包括short和byte类型），以字符“a”开头的指令表示对象引用的比较。

| 指令      | 说明                                                  |
| --------- | ----------------------------------------------------- |
| if_icmpeq | 比较栈顶两个int类型数值大小，当前者等于后者时跳转     |
| if_icmpne | 比较栈顶两个int类型数值大小，当前者不等于后者时跳转   |
| if_icmplt | 比较栈顶两个int类型数值大小，当前者小于后者时跳转     |
| if_icmple | 比较栈顶两个int类型数值大小，当前者小于等于后者时跳转 |
| if_icmpgt | 比较栈顶两个int类型数值大小，当前者大于后者时跳转     |
| if_icmpge | 比较栈顶两个int类型数值大小，当前者大于等于后者时跳转 |
| if_acmpeq | 比较栈顶两个引用类型数值，当结果相等时跳转            |
| if_acmpne | 比较栈顶两个引用类型数值，当结果不相等时跳转          |

### 代码示例
```java
public void compare2() {
        int i = 10;
        int j = 20;
        System.out.println(i > j);
}
```

字节码
![[compare 2 字节码.png]]

```java
public void compare3() {
    Object obj1 = new Object();
    Object obj2 = new Object(); // new 指向堆内存地址不一样
    System.out.println(obj1 == obj2); //false
    System.out.println(obj1 != obj2);//true
}
```

![[compare3 字节码.png]]


## 多条件分支跳转指令

多条件分支跳转指令是专为switch-case语句设计的，主要有tableswitch和lookupswitch。

从助记符上看，两者都是switch语句的实现，它们的区别：

-   tableswitch要求多个**条件分支值是连续的**，它内部只存放起始值和终止值，以及若干个跳转偏移量，通过给定的操作数index，可以立即定位到跳转偏移量位置，因此**效率比较高**
-   指令lookupswitch内部**存放着各个离散的case-offset对**，每次执行都要搜索全部的case-offset对，找到匹配的case值，并根据对应的offset计算跳转地址，因此**效率较低**。

指令tableswitch的示意图如下图所示。由于tableswitch的case值是连续的，因此只需要记录最低值和最高值，以及每一项对应的offset偏移量，根据给定的index值通过简单的计算即可直接定位到offset。

![[table switch.png]]

指令lookupswitch处理的是离散的case值，但是出于效率考虑，将case-offset对按照case值大小排序，给定index时，需要查找与index相等的case，获得其offset，如果找不到则跳转到default。指令lookupswitch如下图所示。

![[lookupswitch.png]]

### 代码示例
```java
public void switchTest(int select) {
    int num;
    switch (select) {
        case 1:
            num = 10;
            break;
        case 2:
            num = 20;
            // break; 
        case 3:
            num = 30;
            break;
        default:
            num = 40;
    }
}
```
字节码
![[switchTest.png]]

```java
public void switchTest2(int select) {
        int num;
        switch (select) {
            case 100:
                num = 10;
                break;
            case 500:
                num = 20;
                break;
            case 200:
                num = 30;
                break;
            default:
                num = 40;
        }
}
```
![[switchtest2.png]]

## 无条件跳转指令

-   目前主要的无条件跳转指令为goto。指令goto接收两个字节的操作数，共同组成一个带符号的整数，用于指定指令的偏移量，指令执行的目的就是跳转到偏移量给定的位置处。
-   如果指令偏移量太大，超过双字节的带符号整数的范围，则可以使用指令goto_w，它和goto有相同的作用，但是它接收4个字节的操作数，可以表示更大的地址范围。
-   指令jsr、jsr_w、ret虽然也是无条件跳转的，但主要用于try-finally语句，且已经被虚拟机逐渐废弃。


```java
public void whileInt() {
        int i = 0;
        while (i < 100) {
            String s = "Jack";
            i++;
        }
}
```

![[whileInt字节码.png]]

# 异常处理指令

## 抛出异常指令

**athrow指令**

-   在Java程序中显示抛出异常的操作（throw语句）都是由athrow指令来实现。
-   除了使用throw语句显示抛出异常情况之外，**JVM规范还规定了许多运行时异常会在其他Java虚拟机指令检测到异常状况时自动抛出**。例如，在之前介绍的整数运算时，当除数为零时，虚拟机会在idiv或ldiv指令中抛出ArithmeticException异常。

注意：

正常情况下，操作数栈的压入弹出都是一条条指令完成的。唯一的例外情况是**在抛异常时，Java虚拟机会清除操作数栈上的所有内容，而后将异常实例压入调用者操作数栈上**。

### 代码示例
```java
public void throwZero(int i) {
        if (i == 0) {
            throw new RuntimeException("参数错误");
        }
}
```


![[throwZero字节码.png]]

## 异常处理与异常表

### 处理异常

-   在Java虚拟机中，**处理异常**（catch语句）不是由字节码指令来实现的（早期使用jsr、ret指令），而是**采用异常表**来完成的。

## 异常表

-   如果一个方法定义了一个try-catch或者try-finally的异常处理，就会创建一个异常表。它包含了每个异常处理或者finally块的信息。异常表保存了每个异常处理信息。比如：
-   起始位置·结束位置
-   程序计数器记录的代码处理的偏移地址
-   被捕获的异常类在常量池中的索引

**当一个异常被抛出时，JVM会在当前的方法里寻找一个匹配的处理，如果没有找到，这个方法会强制结束并弹出当前栈帧，并且异常会重新抛给上层调用的方法（在调用方法栈帧）**。

如果在所有栈帧弹出前仍然没有找到合适的异常处理，这个线程将终止。如果这个异常在最后一个非守护线程里抛出，将会导致JVM自己终止，比如这个线程是个main线程。

**不管什么时候抛出异常，如果异常处理最终匹配了所有异常类型，代码就会继续执行。**在这种情况下，如果方法结束后没有抛出异常，仍然执行finally块，在return前，它直接跳到finally块来完成目标**

### 代码示例
```java
public void tryCatch() {
    try {
        File file = new File("hello.txt");
        FileInputStream fis = new FileInputStream(file);
        String info = "hello!";
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (RuntimeException e) {
        e.printStackTrace();
    }
}
```

![[trycatch字节码.png]]


# 同步控制指令

java虚拟机支持两种同步结构：**方法级的同步**和**方法内部一段指令序列的同步**，这两种同步都是使用**monitor**来支持的。

## 方法级的同步

-   方法级的同步：是**隐式**的，即无须通过字节码指令来控制，它实现在方法调用和返回操作之中。虚拟机可以从方法常量池的方法表结构中的ACC_SYNCHRONIZED访问标志得知一个方法是否声明为同步方法。
-   当调用方法时，调用指令将会检查方法的ACC_SYNCHRONIZED访问标志是否设置。
    
    -   如果设置了，执行线程将先持有同步锁，然后执行方法。最后在方法完成（无论是正常完成还是非正常完成）时释放同步锁
    -   在方法执行期间，执行线程持有了同步锁，其他任何线程都无法再获得同一个锁。
    -   如果一个同步方法执行期间抛出了异常，并且在方法内部无法处理此异常，那这个同步方法所持有的锁将在异常抛到同步方法之外时自动释放。

```java
public synchronized void test() {}
```

![[访问标志.png]]

![[synchronize字节码.png]]

## 方法内指定指令序列的同步

-   同步一段指令集序列： 通常是由java中的synchronized语句块来表示的。jvm的指令集有monitorenter和monitorexit两条指令来支持synchronized关键字的语义。
-   当一个线程进入同步代码块时，它使用monitorenter指令请求进入。如果当前对象的监视器计数器为0，则它会被准许进入，若为1，则判断持有当前监视器的线程是否为自己，如果是，则进入，否则进行等待，直到对象的监视器计数器为0，才会被允许进入同步块。
-   当线程退出同步块时，需要使用monitorexit声明退出。在Java虚拟机中，任何对象都有一个监视器与之相关联，用来判断对象是否被锁定，当监视器被持有后，对象处于锁定状态。
-   指令monitorenter和monitorexit在执行时，都需要在操作数栈顶压入对象，之后monitorenter和monitorexit的锁定和释放都是针对这个对象的监视器进行的。
-   下图展示了监视器如何保护临界区代码不同时被多个线程访问，只有当线程4离开临界区后，线程1、2、3才有可能进入。

![线程图](https://segmentfault.com/img/remote/1460000037628906 "线程图")

代码示例：
```java
private int i = 0;
    private Object obj = new Object();

    public void subtract() {
        synchronized (obj) {
            i--;
        }
}
```

![字节码](https://segmentfault.com/img/remote/1460000037628908 "字节码")