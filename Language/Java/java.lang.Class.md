# 定义
- java.lang.Class\<T> 的实例对象保存着对应 class 的运行时类型信息，用于*运行时类型识别([[#RTTI]])*；
- Class类只存私有构造函数，因此对应Class对象由 [[JVM类加载机制]] 创建和加载；
- 每个类（如boolean, byte, char, short, int等基本类型）都**有且只用一个Class对象**；
- *Class 类也是类的一种，与 class 关键字不同*。

类对象和 Class 对象关系如下：
![[类对象和Class的关系.png]]

## RTTI
**Conception**
RTTI（Run-Time Type Identification）运行时类型识别，其作用是**在运行时识别一个对象的类型和类的信息**。
RTTI 和 反射的区别：
1. RTTI：在编译期知道要解析的类型；
2.  [[反射]]：在运行期知道要解析的类型。
**无论是RTTI还是反射，其本质都是一样的，都是去动态的获取类的信息**

**运行期间，JVM 通过 Class 对象记录每个对象的RTTI **。



