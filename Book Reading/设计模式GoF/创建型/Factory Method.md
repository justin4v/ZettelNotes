#Design-Pattern
# Intent
- Define an interface for creating an object, but *let subclasses decide which class to instantiate*. 
- Factory Method lets a class *defer instantiation to subclasses*.

# Also Known As
*Virtual Constructor*

# Motivation/Example
Frameworks use abstract classes to define and maintain relationships between objects. A framework is *often responsible for creating these objects* as well. 

Consider a framework for applications that can present multiple documents to the user：
1. Two *key abstractions* in this framework are the classes *Application* and *Document*. 
2. To create a drawing application, for example,we define the classes *DrawingApplication* and *DrawingDocument*.
3. The Application class is responsible for managing Documents and will create them as required—when the user selects Open() or New() from a menu, for example.
4. Because the particular Document subclass to instantiate is application-specific,the Application class can't predict the subclass of Document to instantiate—the Application class only knows when a new document should be created, not what kind of Document to create. 
5. The **Factory Method** pattern offers a solution. *It encapsulates the knowledge of which Document subclass to create and moves this knowledge out of the framework*.
	1. Application subclasses redefine an abstract CreateDocument operation on Application to return the appropriate Document subclass.
	2. Once an Application subclass is instantiated, it can then instantiate application-specific Documents. 
	3. We call CreateDocument a factory method because it is *responsible for "manufacturing" an object*.

![[Factory method示例结构.png]]

# Applicability 
**Use the Factory Method pattern when** 
- a class can't anticipate the class of objects it must create. 推迟到子类决定。
- a class wants its subclasses to specify the objects it creates. 
- classes *delegate responsibility to one of several helper subclasses*, and you want to localize the knowledge of using which helper subclass.

# Structure
![[FactoryMethod结构.png]]

# Participants 
- **Product** (Document)
	- defines the *interface* of objects the factory method creates. 
- **ConcreteProduct** (MyDocument) 
	- *implements the Product interface*. 
- **Creator** (Application) 
	- *declares the factory method*, which returns an object oftype Product.
	- call the factory method to create a Product object
- **ConcreteCreator** (MyApplication) 
	- *overrides* the factory method to return an instance of a ConcreteProduct.

# Collaborations 
- Creator relies on its subclasses to define the factory method so that it returns an instance of the appropriate ConcreteProduct.

# Consequences 
Factory methods *eliminate the need to bind application-specific classes into your code*. The code only deals with the Product interface; therefore it can work with any user-defined ConcreteProduct classes.

A potential disadvantage of factory methods is that clients might have to subclass the Creator class just to create a particular ConcreteProduct object. 

Here are two additional consequences of the FactoryMethod pattern: 
1. *Provides hooks for subclasses*. Creating objects inside a class with a factory method is always more flexible than creating an object directly. Factory Method gives subclasses a hook for providing an extended version of an object. 
2. *Connects parallel(平行) class hierarchies*. Parallel class hierarchies result when a class delegates some of its responsibilities to a separate class.

## Parallel Example
Consider graphical figures that can be manipulated interactively(交互); 
- that is, they can be stretched, moved, or rotated using the mouse.
- It often requires *storing and updating information that records the state of the manipulation at a given time*. This state is needed only during manipulation; therefore it needn't be kept in the figure object. Moreover, different figures behave differently when the user manipulates them.
- With these constraints, it's better to *use a separate Manipulator object* that implements the interaction and keeps track of any manipulation-specific state that's needed. 

![[FactoryMethod 连接平行类示例.png]]

# Implementation 
issues when applying the FactoryMethod pattern: 
1. 两种情况
	1. *Creator class is an abstract class* and does not provide an implementation for the factory method it declares, 
	2. *Creator is a concrete class* and provides a default implementation for the factory method. 
2. *Parameterized factory methods*. Another variation on the pattern lets the factory method create multiple kinds of products. The factory method takes a parameter that identifies the kind of object to create. 

# Known Uses
Factory methods 主要应用于 toolkits and frameworks

# Related Patterns
1. *Abstract Factory* is often implemented with factory methods. The Motivation example in the Abstract Factory pattern illustrates Factory Method as well. 
2. *Factory methods* are usually called within Template Methods . In the document example above, NewDocument is a template method. 
3. *Prototypes*  don't require subclassing Creator. However, they often require an Initialize operation on the Product class.Creator uses Initialize to initialize the object. Factory Method doesn't require such an operation.