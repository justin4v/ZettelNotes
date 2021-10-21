# 意图
Provide an **interface** for **creating families** of related or dependent objects without specifying their concrete classes.

# 别名
Kit

# 动机
Consider a user interface *toolkit*  that supports multiple look-and-feel standards, such as Motif and Presentation Manager. Different look-and-feels define different appearances and behaviors for user interface `"widgets"` like scroll bars, windows, and buttons. 

To be portable across look-and-feel standards, an application should not hard-code its widgets for a particular look and feel. Instantiating look-and-feel-specific classes of widgets throughout the application makesit hard to change the look and feel later.

We can solve this problem by defining an abstract `WidgetFactory` class that declares an interface for creating each basic kind of `widget`. There's also an abstract class for each kind of widget, and concrete subclasses implement widgets for specific look-and-feel standards. 

`WidgetFactory`'s interface has an operation that returns a new widget object for each abstract widget class.
Clients call these operations to obtain widget instances, but clients aren't aware of the concrete classes they're using. 
Thus *clients stay independen* of the prevailing look and feel.

UML类图如下：

![[Abstract Factory示例UML类图.png]]

`WidgetFactory` also **enforces dependencies between the concrete widget classes**. A Motif scroll bar should be used with a Motif button and a Motif text editor, and that constraint is enforced automatically as a consequence of using a `MotifWidgetFactory`.

# 适用性
Use the *Abstract Factory* pattern when ：
>- a system should *be independent of how its products are created, composed, and represented*. 
>- a system should *be configured with one of multiple families of products.* 
>- *a family of related product objects is designed to be used together*, and you need to enforce this constraint. 
>- you want to* provide a class library of products*, and you want to reveal just their interfaces, not their implementations.

# 结构
*Abstract Factory* 一般结构如下：

![[Abstract Factory一般结构.png]]

## 解释
- `AbstractFactory (WidgetFactory)`
	- declares an interface for operations that create abstract product objects. 
- `ConcreteFactory (MotifWidgetFactory, PMWidgetFactory)`
	-  implements the operations to create concrete product objects. 
-  `AbstractProduct (Window, ScrollBar)`
	- declares an interfacefor a type of product object. 
-  `ConcreteProduct (MotifWindow, MotifScrollBar)`
	-  defines a product objecttobecreated bythe corresponding concrete factory. 
	-  implements theAbstractProduct interface. 
- `Client`
	- uses *only interfaces declared by AbstractFactory and AbstractProduct* classes


# 协作
- Normally a single instance of a `ConcreteFactory` class is created at run-time.
- `AbstractFactory` defers creation of product objects to its `ConcreteFactory` subclass.

# 效果
## 优点
1. **It isolates concrete classes**: The Abstract Factory pattern helps you control the classes of objectsthat an application creates. Because a *factory encapsulates the responsibility and the process of creating product objects, it isolates clients from implementation classes.* Clients manipulate instances through their abstract interfaces. Product class names are isolated in the implementation of the concrete factory; they do not appear in client code.
2. **It makes exchanging product families easy**.
3. **It promotes consistency among products**: When product objects in a family are designed towork together,it's important that an application use objects from only one family at a time. AbstractFactory makes this easy to enforce.
	
## 缺点
- Supporting new kinds of `Products` is difficult : Extending abstract factories to produce new kinds of `Products` isn't easy. That's because the AbstractFactory interface *fixes the set of products that can be created.(对外提供的接口已经确定：createProductA/createProductB)* Supporting new kinds of products requires extending the factory interface, which *involves changing the `AbstractFactory` class and all of its subclasses*. 

# 实现
1. **Factories as singletons.** An application typically needs only one instance of a *ConcreteFactory* per product family. So it's usually best implemented as a *[[Singleton]]*

3. *Creating the products*.It's up to ConcreteProduct subclasses to actually create them. 
	1. The most common way to do this is to define a factory method (see *[[Factory Method]]*) for each product. A concrete factory will specify its products by *overriding the factory method* for each. While this implementation is simple, it requires a new concrete factory subclass for each product family.
	2. If *many product families are possible（不适用 Factory Method）*, the concrete factory can be implemented using the *[[Prototype]]*  pattern. The concrete factory is initialized with a prototypical instance of each product in the family, and it *creates a new product by cloning its prototype*.
	3. *可以将多个系列的 Prototype 存储在字典（Map）中，使用时搜索并复制 Prototype。* 


4. *Defining extensible factories.* 
	1. AbstractFactory usually defines a different operation for each kind of product it can produce. The kinds of products are encoded in the operation signatures. *Adding a new kind of product requires changing the AbstractFactory interface and all the classes that depend on it*.
	2. A more flexible but less safe design is to *add a parameter to operations that create objects*. This parameter specifies the kind of object to be created. It could be a class identifier, an integer, a string, or anything else that identifies the kind of product. 
	3. In fact with this approach, *AbstractFactory only needs a single `"Make"` operation with a parameter indicating the kind of object to create.* This technique can be used in the Prototype-based  and the class-based abstract factories discussed earlier.
