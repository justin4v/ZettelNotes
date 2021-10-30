# Visitor
Represent an operation to be performed on the elements of an objectstructure. 
Visitor lets you define a new operation without changing the classes of the elements on which it operates

## 适用场景
- 多个元素；
- 适用于业务规则**要求遍历多个元素，如统计**；
- **元素不变，对其操作可能变化**。

此外，Visitor还可用做拦截器（Interceptor）。

## 角色
角色有：
1. IElement，元素接口，定义接受访问的方法： accept(IVisitor);
2. IVisitor，访问者接口，定义访问具体元素的方法：visit(Element)，因为需要调用Element的对象方法，参数需要是IElement的具体实现；
3. Element，IElement的具体实现，包含具体业务方法；
4. Visitor，IVisitor具体实现，调用/组合Element方法与属性获取结果；
5. ObjectStruture，多个Element的容器，一般用于遍历。

visitor模式的最终结果是 visitor 调用/访问各个element 属性/方法得到的。

UML类图如下：
![[Visitor模式UML类图.png]]

## 优点
1. 符合单一职责 SRP：**Element 类负责元素的加载，Visitor负责访问元素**，并进行数据的处理；
2. 扩展性好、灵活：增加行为(不改变元素结构)，只需要在visitor中增加方法；

## 缺点
1. 不符合依赖倒置原则、最少了解原则(Element具体细节对Visitor公开)：visitor依赖于具体的Element实现类而不是接口；
2. 元素结构难以改变：如果Element中增加属性，那么依赖于它的所有Visitor都需要修改。


## 实例
1. [java.nio.file.FileVisitor (Java Platform SE 8 ) ](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileVisitor.html)