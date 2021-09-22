# 定义
- java.lang.Class\<T> 的实例对象保存着对应 class 的运行时类型信息，用于*运行时类型识别*，虚拟机通常使用运行时类型信息选取正确方法去执行；
- Class 没有公共构造方法。在[[JVM类加载机制]]调用 [[JVM ClassLoader]] 中的 *defineClass* 方法自动构造的，不能显式地声明一个Class对象；
- 每个类（如boolean, byte, char, short, int等基本类型）都有一个Class对象。


## RTTI
**Conception**
RTTI（Run-Time Type Identification）运行时类型识别，其作用是**在运行时识别一个对象的类型和类的信息;**


# 反射
Class 对象记录了类的结构信息，如 static 变量和非static变量（类加载过程初始化）、方法（运行时）等。
Class 提供了可以获取结构信息的方法
