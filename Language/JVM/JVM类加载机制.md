# JVM架构
JVM架构图如下：

![[JVM架构图.png]]

# 完整流程
**JVM 实际启动到 JVM 销毁的完整流程**如下图：

![[class加载的实际流程.png]]
**DES**
1. 启动虚拟机 （C++负责创建）[windows : bin/java.exe调用 jvm.dll。 Linux: java 调用 libjvm.so] ;
2. 创建一个引导类加载器（**Bootstrap** class loader）实例 （C++实现）; 
3. C++ 调用Java代码，创建JVM启动器和sun.misc.Launcher实例 [调用引导类加载器，加载创建 **Extend classloader** 和 **Application classloader**]；
4. sun.misc.Launcher.getLauncher() 获取运行类的加载器 ClassLoader ( 实际是AppClassLoader );  
5. 获取到ClassLoader后调用 loadClass(“A”) 方法加载运行的类A ；
6. 加载完成执行A类的main方法 ；
7. 程序运行结束 ；
8. JVM销毁；

# ClassLoader SubSystem
**conception**
- **Loading**：加载 .class JMV（结构信息=>Metaspace、class对象=>Heap）; 
- **Linking**：准备 class对象（校验class，变量赋予初始值，符号引用解析=>静态链接）；
- **Initialization**：初始化（初始化变量，动态链接）。

类加载过程：
![[类加载过程.png]]

## Loading
1. 通过类的**全限定名**获取类的二进制字节流；
2.  加载类的**静态结构到元空间**(Metaspace)；
3. 在 Heap 中创建**java.lang.class 对象** 并指向 Metaspace 中的 class 结构。


## Linking
### Verification
1.**Target**
 - **保证 Class 字节流符合虚拟机规范**

参考：[[Class 文件结构]]

2.**DES**
1. **文件格式验证**：验证字节流是否符合Class文件格式的规范。需以**魔数`0xCAFEBABE`开头（身份标识）**、**主次版本号**是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
2. **元数据验证**：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了 `java.lang.Object`之外。
3. **字节码验证**：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
4. **符号引用验证**：确保解析动作能正确执行。

### Preparation
1.**Target**
 - 在 **Metaspace 中为类变量默认初始化**

2.**Attention**
-   Preparation进行内存分配的**仅包括类变量（static），分配在 Metaspace**，而不包括实例变量。实例变量会在对象实例化时随着对象一块分配在 Java 堆中；
-   默认初始值是指**初始化为数据类型默认的零值**（如 `0`、`0L`、`null`、`false` 等），不执行Java 代码；
-   类字段**同时被 final 和 static 修饰（不允许修改，类似于常量，必须初始化时赋予值）**，称为`ConstantValue`属性，Preparation 阶段变量就会被初始化为指定的值。

### Resolution
1.**Target**
-  **符号引用替换成直接引用**

2.**DES**
符号引用是指**一组指明需要引用对象的符号**，是一个唯一标识，但不是实际内存地址，如静态方法、类变量等（类加载过程中只能确定属于类的属性）。 
直接引用是指**能确定数据在内存中的位置**的引用，如直接指针、相对偏移量、句柄

在类加载过程中完成的解析称为**静态链接**。
在运行期间完成的解析称为**动态链接**。

## Initialization
1.**Target**
 - **为类变量赋予正确的值**；
 - 真正开始执行类定义的（如static代码块）Java代码。

2.**DES**
 - 除了 ConstantValue 由 JVM 直接初始化外（[[#Preparation]]阶段），**其他的初始化代码被Java 编译器放入方法`<clinit>`中执行**。
 - 一般来说，只有在**==第一次主动调用==** 某个类时才会去进行类加载；
 - 如果有父类，先加载父类再加载自身。

2.1 **类初始化步骤**
1.  如果类还没有被加载和链接，开始加载该类。
2.  如果该类的直接父类还没有被初始化，先初始化其父类。
3.  如果该类有初始化语句，则依次执行这些初始化语句。

2.2 **类初始化时机**
JVM规定了**6种主动调用**：
1.  一个类的实例被创建（`new`操作、反射、`cloning`，反序列化）；
2.  调用类的`static`方法；
3.  使用或对类/接口的`static`属性进行赋值时（这不包括`final`的与在编译期确定的常量表达式）；
4.  当调用 API 中的某些反射方法时；
5.  子类被初始化（会先初始化父类）；
6.  被设定为 JVM 启动时的启动类（具有`main`方法的类，总是首先初始化）。

**其他被动调用不会初始化类**，如：

-   *子类引用父类的静态字段，不会导致子类初始化*
```java
System.out.println(SubClass.value); // value 字段在 SuperClass 中定义
```

-   *通过数组定义来引用类，不会触发此类的初始化*。该过程**会对数组类进行初始化**，数组类是一个由虚拟机自动生成的、直接继承自 `Object` 的子类，其中包含了数组的属性和方法。

```java
SuperClass[] sca = new SuperClass[10];
```

-   常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此*不会触发定义常量的类的初始化*。

```java
System.out.println(ConstClass.HELLOWORLD);
```


2.3 `\<clinit\>()` 的细节

-   `\<clinit\>()` 是*编译器自动收集类中所有类变量的赋值动作和静态语句块（static{} 块）中的语句合并产生*。编译器收集的顺序*由语句在源文件中出现的顺序决定*。特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它*之后的类变量只能赋值，不能访问*。例如以下代码：

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

-   `<clinit>()` 方法*对于类或接口不是必须的*，如果一个类中不包含静态语句块，也没有对类变量的赋值操作，编译器可以不为该类生成 `<clinit>()` 方法。
-   接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成 `<clinit>()` 方法。但接口与类不同的是，执行*接口的 `<clinit>()` 方法不需要先执行父接口的 `<clinit>()` 方法*。只有当*父接口中定义的变量使用时，父接口才会初始化*。另外，*接口的实现类在初始化时也不会执行接口的 `<clinit>()` 方法*。
-   *JVM 保证一个类的 `<clinit>()` 方法在多线程环境下被正确的加锁和同步*，如果多个线程同时初始化一个类，只会有一个线程执行这个类的 `<clinit>()` 方法，其它线程都会阻塞等待，直到活动线程执行 `<clinit>()` 方法完毕。如果在一个类的 `<clinit>()` 方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽。
