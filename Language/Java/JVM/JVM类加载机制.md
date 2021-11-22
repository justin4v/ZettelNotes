#Java基础 #JVM #ClassLoad
# JVM架构
JVM架构图如下：

![[JVM架构图.png]]

# 完整流程
**JVM 实际启动到 JVM 销毁的完整流程**如下图：

![[class加载的实际流程.png]]

## JVM启动销毁流程
1. 启动虚拟机 （C++负责创建。*windows : bin/java.exe调用 jvm.dll。 Linux: java 调用 libjvm.so*） ;
2. 创建一个引导类加载器（**Bootstrap** class loader）实例 （C++实现）; 
3. C++ 调用Java代码，创建JVM启动器和sun.misc.Launcher实例 [调用引导类加载器，加载创建 **Extend classloader** 和 **Application classloader**]；
4. sun.misc.Launcher.getLauncher() 获取运行类的加载器 ClassLoader ( 实际是AppClassLoader );  
5. 获取到ClassLoader后调用 loadClass(“A”) 方法加载运行的类A ；
6. 加载完成执行A类的main方法 ；
7. 程序运行结束 ；
8. JVM销毁；

# Class Loader SubSystem
- **Loading**：**加载 .class**（结构信息=>Metaspace、class对象=>Heap）; 
- **Linking**：**准备 class 对象**（校验class，变量赋予初始值，符号引用解析=>静态链接）；
- **Initialization**：**初始化**（初始化变量，动态链接）。

类加载过程：
![[类加载过程.png]]

## 注意
1. 加载、验证、准备、初始化和卸载这五个阶段的顺序是确定的，类型的加载过程必须按照这种顺序按部就班地开始；
2. 解析阶段则不一定：它在某些情况下可以在初始化阶段之后再开始，
这是为了支持Java语言的运行时绑定特性（也称为动态绑定或晚期绑定）

# Loading
1. 通过类的**全限定名**获取类的二进制字节流；
2.  加载类的**静态结构到元空间**(Metaspace)；
3. 在 Heap 中创建**java.lang.class 对象** 并指向 Metaspace 中的 class 结构。

# Linking
## Verification
### Target
 - **保证 Class 字节流符合虚拟机规范**

参考：[[class 文件结构]]

### 过程
1. **文件格式验证**：验证字节流是否符合Class文件格式的规范。需以**魔数`0xCAFEBABE`开头（身份标识）**、**主次版本号**是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
2. **元数据验证**：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了 `java.lang.Object`之外。
3. **字节码验证**：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
4. **符号引用验证**：确保解析动作能正确执行。

## Preparation
### Target
 - 在 **Metaspace 中为类变量默认初始化**

### Attention
-   Preparation 进行内存分配：
	1. **仅包括类变量（static），分配在 Metaspace**
	2. **不含有静态构造块（\<clinit>中初始化)** ；
	3. **不包括实例变量**。*实例变量会在对象实例化时随着对象一块分配在 Java 堆中*；
-   默认初始值是指**初始化为数据类型默认的零值**（如 `0`、`0L`、`null`、`false` 等），不执行Java 代码；
-   类字段**同时被 final 和 static 修饰（不允许修改，类似于常量，必须初始化时赋予值）**，称为`ConstantValue`属性，*Preparation 阶段变量就会被初始化为指定的值*。

## Resolution
### 目的
-  **符号引用替换成直接引用**

### 符号引用和直接引用
1. 符号引用：
**一组指明需要引用对象的符号**，是一个唯一标识，但不是实际内存地址，如静态方法、类变量等（类加载过程中只能确定属于类的属性）。 
2. 直接引用
**能确定数据在内存中的位置**的引用，如直接指针、相对偏移量、句柄

- 在**类加载过程（编译）中完成的解析**称为**静态绑定**。
- 在**运行期间完成的解析**称为**动态绑定**。
参考：[[静态绑定和动态绑定]]

# Initialization
## Target
 - **为类变量赋予正确的值，赋值初始化**；
 - **收集 static 初始化语句**，放入**类初始化方法 \<clinit>** 并执行。

## 过程
### 1 步骤
*Class Loader Subsystem 加载类（这里指包含 [[#Loading]]、[[#Linking]]、[[#Initialization]]三个阶段）的完整步骤*：
1. 满足[[#2 类 Initialization 时机|类加载时机]]要求，类加载开始；
3.  如果该类的直接父类还没有被初始化，先初始化其父类。
4. main 方法的类首先初始化；
6. 首先对类变量（static）进行默认初始化（[[#Preparation]]阶段）；
7. 对 ConstantValue 初始化（[[#Preparation]]阶段）；
8.  调用 \<clinit> （同类型-static 或非static-按源码中顺序排序）进行类初始化；
9.  如果调用 Constructor，则会调用 \<init> 进行实例初始化；
10.  最后调用构造函数初始化。

![[类初始化顺序示意.png]]

### 2 类 Initialization 时机
一般来说，只有在**==第一次主动调用==** 某个类时，才会去进行类，JVM规定了**6种主动调用**：
1.  一个类的实例被**创建**（`new`操作、反射Class.forName("")方法、`cloning`，反序列化）；
2.  **调用类的`static`方法**；
3.  **使用或对类/接口的`static`属性进行赋值**时（这不包括`final`和与在编译期确定的常量表达式）； ^dded1d
4.  当调用 API 中的某些**反射方法**时；
5.  **子类被初始化**（会先初始化父类）；
6.  被设定为 JVM 启动时的**启动类**（main方法的类首先初始化）。

### Attention
1. 对于[[#^dded1d|第3点]]，如果使用或赋值**编译期间能确定**的常量，不会对类进行初始化；
	1. 但是*如果编译期间无法确定，则需要正常初始化、运行以确定值*。
	如 `public static final double str=Math.random();//编译期间不能确定`，如果使用了 str 变量，需要对类初始化。

2. **其他被动调用不会初始化类**，如：
	-   **子类引用父类的静态字段**，不会导致子类初始化
	```java
	System.out.println(SubClass.value); // value 字段在 SuperClass 中定义
	```

	-   **通过数组定义来引用类**，不会触发此类的初始化。该过程**会对数组类进行初始化**，数组类是一个由虚拟机自动生成的、直接继承自 `Object` 的子类，其中包含了数组的属性和方法。

	```java
	SuperClass[] sca = new SuperClass[10];
	```

3. **调用类的常量**。常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此*不会触发定义常量的类的初始化*。
	```java
	System.out.println(ConstClass.HELLOWORLD);
	```


### 4 类构造器`<clinit>` 
-  *`<clinit>`只会执行一次*；
-   `<clinit>` 是*编译器自动收集类中所有类变量的赋值动作和静态语句块（static{} 块）中的语句合并产生*。编译器收集的顺序*由语句在源文件中出现的顺序决定*。特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它*之后的类变量只能赋值，不能访问*。例如以下代码：

```java
public class Test {
    static {
        i = 0;                // 给变量赋值可以正常编译通过
        System.out.print(i);  // 这句编译器会提示“非法向前引用”
    }
    static int i = 1;
}
```

-   与类的构造函数（或者说实例构造器 `<init>()`）不同，*不需要显式的调用父类的构造器*。虚拟机会自动保证在子类的 `<clinit>()` 方法运行之前，父类的 `<clinit>()` 方法已经执行结束。因此虚拟机中第一个执行 `<clinit>()` 方法的类肯定为 `java.lang.Object`。
-   由于父类的 `<clinit>()` 方法先执行，也就意味着*父类中定义的静态语句块要优于子类的*。例如以下代码：

```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
     System.out.println(Sub.B);  // 输出结果是父类中的静态变量 A 的值，也就是 2。
}
```

-   `<clinit>` 方法*对于类或接口不是必须的*，如果一个类中不包含静态语句块，也没有对类变量的赋值操作，编译器可以不为该类生成 `<clinit>` 方法。
-   接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成 `<clinit>` 方法。但接口与类不同的是，执行*接口的 `<clinit>` 方法不需要先执行父接口的 `<clinit>` 方法*。只有当*父接口中定义的变量使用时，父接口才会初始化*。另外，*接口的实现类在初始化时也不会执行接口的 `<clinit>` 方法*。
-   *JVM 保证一个类的 `<clinit>` 方法在多线程环境下被正确的加锁和同步*，如果多个线程同时初始化一个类，只会有一个线程执行这个类的 `<clinit>` 方法，其它线程都会阻塞等待，直到活动线程执行 `<clinit>` 方法完毕。如果在一个类的 `<clinit>` 方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽。

### 5 方法构造器`<init>`
1.  编译器按照出现顺序收集*成员变量赋值语句、普通代码块，最后收集构造函数*的代码，最终组成\<init>；
	- **\<init>** is the (or one of the) *constructor(s) for the instance, and non-static field initialization*；
	- **\<clinit>** are the *static initialization blocks for the class, and static field initialization*。
2. \<init> 是实例构造器方法，在程序**执行类的 constructor 时才会执行 \<init> 方法**;
	在![[字节码示例#^e8050d]] 中`main`方法的 `Code` 区第 7 行:
	`7 invokespecial #4 <java/lang/StringBuilder.<init> : ()V>` 
	可以看到*在new StringBuilder 时调用了 StringBuilder 的 \<init> 实例初始化方法*。
3. \<clinit> 是类构造器方法，也就是在jvm进行类的 *[[#Initialization]]阶段jvm会调用clinit* 方法。

## 注意
1. JVM 中可以有**零个或多个实例初始化\<init>方法**，一个方法称为实例初始化方法的条件是:
	-   It is **defined in a class**;
	-   Its name is *\<init>*;
	-   It returns *void*。

2. JVM中**自动收集 static 生成一个 \<clinit>**。
