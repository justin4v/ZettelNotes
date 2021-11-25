#Design-Pattern
# Intent
Define an interface for creating an object, but let subclasses decide which class to
instantiate. 
Factory Method lets a class defer instantiation to subclasses.

# Also Known As
Virtual Constructor

# Motivation 
Frameworks use abstract classes to define and maintain relationships between objects. A framework is often responsible for creating these objects as well. 

Consider a framework for applications that can present multiple documents to the user. Two key abstractions in this framework are the classes Application and Document. 

Both classes are abstract, and clients have to subclass them to realize their application-specific implementations. To create a drawing application, for example,we define the classes DrawingApplication and DrawingDocument.

The Application classis responsible for managing Documents and will create them as required—when the user selects Open or New from a menu, for example. Because the particular Document subclass to instantiate is application-specific,the Application class can't predict the subclass of Document to instantiate—the Application class only knows when a new document should be created, not what kind of Document to create. 

This creates a dilemma: The framework must instantiate classes, but it only knows about abstract classes, which it cannot instantiate. The Factory Method pattern offers a solution. It encapsulates the knowledge of which Document subclass to create and moves this knowledge out of the framework

Application subclasses redefine an abstract CreateDocument operation on Application to return the appropriate Document subclass.
Once an Application subclass is instantiated, it can then instantiate application-specific Documents without knowing their class. We call CreateDocument a factory method because it's responsible for "manufacturing" an object.

![[Factory method示例结构.png]]

# Applicability 
Use the Factory Method pattern when 
- a class can't anticipate the class of objects it must create. 
- a class wants its subclasses to specify the objectsit creates. 
- classes delegate responsibility to one of several helper subclasses, and you want to localize the knowledge of which helper subclass is the delegate.

# Structure
![[FactoryMethod结构.png]]

# Participants 
- Product (Document)
	- definesthe interface of objects the factory method creates. 
- ConcreteProduct (MyDocument) 
	- implements theProduct interface. 
- Creator (Application) 
	- declares the factory method, which returns an object oftype Product.
	- Creator may also define a default implementation of the factory method that returns a default ConcreteProduct object. 
	- may call the factory method to create a Product object
- ConcreteCreator (MyApplication) 
	- overrides the factory method to return an instance of a ConcreteProduct.

# Collaborations 
- Creator relies on its subclasses to define the factory method so that it returns an instance of the appropriate ConcreteProduct.


# Consequences 
Factory methods eliminate the need to bind application-specific classes into your code. The code only deals with the Product interface; therefore it can work with any user-defined ConcreteProduct classes.

A potential disadvantage offactory methods is that clients might have to subclass the Creator classjust to create a particular ConcreteProduct object. Subclassing is fine when the client has to subclass the Creator class anyway, but otherwise the client now must deal with another point of evolution. 

Here are two additional consequences of the FactoryMethod pattern: 
1. Provides hooks for subclasses. Creating objects inside a class with a factory method is always more flexible than creating an object directly. Factory Method gives subclasses a hook for providing an extended version of an object. In the Document example, the Document class could define a factory method called CreateFileDialog that creates a default file dialog object for opening an existing document. A Document subclass can define an application-specific file dialog by overriding thisfactory method. In this case the factory method is not abstract but provides a reasonable default implementation. 
2. Connects parallel class hierarchies. In the examples we've considered so far, the factory method is only called by Creators. But this doesn't have to be the case; clients can find factory methods useful, especially in the case of parallel class hierarchies.

# Implementation 
Consider the following issues when applying the FactoryMethod pattern: 
1. Two major varieties。The two main variations of the FactoryMethod pattern are ：
	1.  the case when the Creator class is an abstract class and does not provide an implementation for the factory method it declares, 
	2.  the case when the Creator is a concrete class and provides a default implementation for the factory method. It's also possible to have an abstract class that defines a default implementation, but this is less common. 

The first case requires subclasses to define an implementation,because there's no reasonable default. It gets around the dilemma of having to instantiate unforeseeable classes. In the second case, the concrete Creator uses the factory method primarily for flexibility. It's following a rule that says, "Create objects in a separate operation so that subclasses can override the way they're created." This rule ensures that designers of subclasses can change the class of objects their parent class instantiates if necessary. 

2. Parameterized factory methods. Another variation on the pattern lets the factory method create multiple kinds of products. The factory method takes a parameter that identifies the kind of object to create. All objects the factory method creates will share the Product interface. In the Document example, Application might support different kinds of Documents. You pass CreateDocument an extra parameter to specify the kind of document to create
3. Language-specific variants and issues. Different languages lend themselves to other interesting variations and caveats
4. Using templates to avoid subclassing. As we've mentioned, another potential problem with factory methods is that they might force you to subclass just to create the appropriate Product objects. A