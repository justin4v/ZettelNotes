# 定义
- java.lang.Class\<T> 的实例对象保存着对应 class 的运行时类型信息，虚拟机通常使用运行时类型信息选取正确方法去执行。
- Class 没有公共构造方法。调用 [[JVM ClassLoader]] 中的 *defineClass* 方法自动构造的，因此不能显式地声明一个Class对象

虚拟机为每种类型管理一个独一无二的Class对象。也就是说，每个类（型）都有一个Class对象。