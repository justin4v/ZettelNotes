# 意图
Provide an **interface** for **creating families** of related or dependent objects without specifying their concrete classes.

# 别名
Kit

# 动机
Consider a user interface *toolkit* that supports multiple look-and-feel standards, such as Motif and Presentation Manager. Different look-and-feels define different appearances and behaviors for user interface `"widgets"` like scroll bars, windows, and buttons. 

To be portable across look-and-feel standards, an application should not hard-code its widgets for a particular look and feel. Instantiating look-and-feel-specific classes of widgets throughout the application makesit hard to change the look and feel later.

We can solve this problem by defining an abstract `WidgetFactory` class that declares an interface for creating each basic kind of widget. There's also an abstract class for each kind of widget, and concrete subclasses implement widgets for specific look-and-feel standards. 

WidgetFactory's interface has an operation that returns a new widget object for each abstract widget class. Clients call these operations to obtain widget instances, but clients aren't aware of the concrete classes they're using. 
Thus *clients stay independen* of the prevailing look and feel.

UML类图如下：

![[Abstract Factory示例UML类图.png]]

# 适用性
Use the *Abstract Factory* pattern when ：
- a system should be independent of how its products are created, composed, and represented. 
- a system should be configured with one of multiple families of products. 
- a family of related product objects is designed to be used together, and you need to enforce this constraint. 
- you want to provide a class library of products, and you want to reveal just their interfaces, not their implementations.

# 结构
*Abstract Factory* 一般结构如下：

![[Pasted image 20211020200019.png]]