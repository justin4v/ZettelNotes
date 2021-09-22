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

如果已获得目标类对象实例，通过 *`目标类对象实例.getClass()`返回该类 Class 对象*。

### 目标类名

如果已知目标类名为 myClass，通过 *`Class c = myClass.getClass()` 获得该类 Class 对象*。

### 目标类名运行期确定

如果目标类名在编译器不确定，在运行期可以确定，通过 *`Class.forName(目标类全限定名)`获取该类 Class 对象*。  
`Class.forName(目标类名)`内部通过反射 API 根据目标类名将类**手动加载**到内存中，注意此时尚未创建对象实例。

## Example
### 创建目标类对象实例

`Object newInstance()`：调用默认构造器创建一个对象实例，

> *反射机制只能调用无参的构造器*

### 获得构造器

-   `Constructor[] getConstructors()`：获得**所有public**构造器；
    
-   `Constructor[] getDeclaredConstructors()`：获得**所有访问权限**的构造器；
    
-   `Constructor getConstructor(Class[] params)`：根据指定参数获得对应构造器；
    
-   `Constructor getDeclaredConstructor(Class[] params)`：根据指定参数获得对应构造器。
    

### 获得方法

-   `Method[] getMethods()`：获得**所有public**方法；
    
-   `Method[] getDeclaredMethods()`：获得**所有访问权限**的方法；
    
-   `Method getMethod(String name, Class[] params)`：根据方法签名获取类自身对应**public方法**，或者从基类继承和接口实现的对应**public方法**；
    
-   `Method getDeclaredMethod(String name, Class[] params)`：根据方法签名获得对应的**类自身声明方法**，**访问权限不限**。
    

### 获得变量

-   `Field[] getFields()`：获得类中**所有public**变量
    
-   `Field[] getDeclaredFields()`：获得类中**所有访问权限**变量
    
-   `Field getField(String name)`：根据变量名得到对应的public变量
    
-   `Field getDeclaredField(String name)`：根据变量名获得对应的变量，**访问权限不限**；