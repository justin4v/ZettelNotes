# Static
## Conception
1. Static意为静态的，*被static 修饰的属于类，不属于类的对象*
2. Static 可以修饰 `内部类、方法、成员变量、代码块`。

## Attention
- static在类加载时初始化（加载）完成
- *Static不可修饰 `外部类、局部变量`*【static 属于类的，局部变量属于其方法，并不属于类】

### 外部类
*和内部类相对，指的就是当前类*。
如
```java
// 外部类
class outClass{
	// 内部类
	class innerClass{
	}
}
```

# Final
## Conception
1. final “最终的”的意思，在Java中又有意为常量的意思，*被final修饰的只能进行一次初始化*
2. final 可修饰`类、内部类、方法、成员变量、局部变量、基本类型、引用类型`。

## Attention
- final 可以在编译（类加载）时初始化，也可以在运行时初始化，**初始化后不能被改变**。

# Static final
