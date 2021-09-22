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
1. 传统的”RTTI”，它假定我们在编译期已知道了所有类型(在没有反射机制创建和使用类对象时，一般都是编译期已确定其类型，如 new 对象时该类必须已定义好)；
2. 反射机制，允许在运行时发现和使用类型的信息；
3. **运行期间，JVM 通过 Class 对象记录每个对象的RTTI**。



# 反射
## Conception
- 反射指程序能**访问、检测和修改它本身状态或行为**的一种能力；
- 大部分框架也都是运用反射原理实现。
- Class 对象记录了类的结构信息，如 static 变量和非static变量（类加载过程初始化）、方法（运行时）等，是**反射机制的核心入口**。

## 使用步骤
### 目标类对象实例
```java
// Object类方法。所有类都继承这一方法。
public final native Class<?> getClass();
```

如果已获得目标类对象实例，通过`目标类对象实例.getClass()`返回该类 Class 对象；

### 目标类名

如果已知目标类名为 myClass，通过 `Class c = myClass.getClass()` 获得该类 Class 对象；

#### 目标类名在编译器不确定，在运行期确定

如果目标类名在编译器不确定，在运行期可以确定，使用`Class.forName(目标类名)`获取该类Class对象，**要求目标类名必须是全限定**；  
`Class.forName(目标类名)`内部通过反射API根据目标类名将类**手动加载**到内存中，称为**类加载器加载方法**。加载过程中会把目标类的static方法，变量，代码块加载到JVM，注意此时尚未创建对象实例；
