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
![[Pasted image 20211020195555.png]]