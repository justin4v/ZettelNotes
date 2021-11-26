#Design-Pattern 
# Intent 
Specify the kinds of objects to create using a prototypical instance, and *create new objects by copying this prototype*

# Motivation
1. You could build an editor for music scores(乐谱) by customizing a general framework for graphical editors and adding new *objects that represent notes(音符), rests(休止符), and staves(五线谱).*- 抽象概念：**Graphics**
2. The editor framework may have a palette(画板) of tools for adding these music objects to the score. The palette would also include *tools for selecting, moving, and otherwise manipulating music objects.* -- 抽象概念：**GraphicTools**

- Let's assume the framework provides an *abstract Graphic class* for graphical components, like notes and staves. 
- Moreover, it'll provide an *abstract Tool class* for defining tools like those in the palette. 
- The framework also predefines a GraphicTool subclass for tools that create instances of graphical objects and add them to the document. 

## 问题
But GraphicTool presents a problem to the framework designer.：
- The classes for notes and staves are specific to our application, but the GraphicTool class belongs to the framework. 
- *GraphicTool doesn't know how to create instances of our music classes* to add to the score. 
- We could *subclass GraphicTool for each kind of music object, but that would produce lots of subclasses* that differ only in the kind of music object they instantiate.

We know **object composition is a flexible alternative to subclassing**. 
The question is, **how can the framework use *object composition* to parameterize instances of GraphicTool** by the class of Graphic they're supposed to create? 

GraphicTool 中的动作需要使用 Graphic 具体类（完成具体动作是 Graphic 的责任），如何在 GraphicTool 中参数化聚合 Graphic 具体类。

## 解决
The solution lies in：
1. making **GraphicTool create a new Graphic by copying or "cloning" an instance of a Graphic subclass**. We call this instance a prototype. 
2. GraphicTool is parameterized by the prototype it should clone and add to the score. 
3. If all Graphic subclasses support a *Clone* operation, then the GraphicTool can clone any kind of Graphic. 

In music editor, each tool for creating a music object is an instance of GraphicTool that's *initialized with a different prototype*. 
Each GraphicTool instance will produce a music object by cloning its prototype and adding the clone to the score。

![[原型模式音谱示例.png]]

We can use the Prototype pattern to reduce the number of classes even further：
1. We have separate classes for whole notes and half notes,but that's probably unnecessary.
2. Instead they could be instances of the same class initialized with different bitmaps and durations. 

# Applicability
Use the Prototype pattern **when a system should be independent of how its products are created, composed, and represented**; 
and：
1. when the classes to instantiate are **specified at run-time**, for example, by **dynamic loading**; or 
2. to **avoid building a class hierarchy of factories that [[Factory Method#Parallel Class Hierarchy|parallels the class hierarchy]] of products**; or 
3. when **instances of a class only can have  one of a few different combinations of state**. It may be more convenient to install a corresponding number of prototypes and clone them rather than instantiating the class manually, each time with the appropriate state.

# Structure
![[原型模式结构.png]]

# Participants 
- **Prototype** (Graphic) 
	- declares an interface for cloning itself. 
- **ConcretePrototype** (Staff, WholeNote, HalfNote) 
	- implements an operation for cloning itself. 
- **Client** (GraphicTool) 
	- creates a new object by asking a prototype to clone itself

# Collaborations 
- A client asks a prototype to clone itself.

# Consequences 
Prototype has many of the same consequences that Abstract Factory (87) and Builder (97) have: 
1. It hides the concrete product classes from the client, thereby reducing the number of names clients know about. 
2. Moreover, these patterns let a client work with application-specific classes withoutmodification. 
3. Additional benefits of the Prototype pattern are listed below. 
	1. Adding and removing products at run-time. Prototypes let you incorporate a new concrete product class into a system simply by registering a prototypical instance with the client.
	2. Specifying new objects by varying values. Highly dynamic systems let you define new behavior through object composition—by specifying values for an object's variables, for example—and not by defining new classes.
	3. Specifying new objects by varying structure. Many applications build objects from parts and subparts. 
	4. Reduced subclassing. FactoryMethod (107)often produces a hierarchy of Creator classes that parallels the product class hierarchy. The Prototype pattern lets you clone a prototype instead of asking a factory method to make a new object. Hence you don't need a Creator class hierarchy at all.
	5. Configuring an application with classes dynamically.