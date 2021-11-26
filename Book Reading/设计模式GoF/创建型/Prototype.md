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
GraphicTool 中的动作需要使用 Graphic 具体类，如何在 GraphicTool 中参数化聚合 Graphic 具体类。

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

A tool for creating whole notes becomes just a GraphicTool whose prototype is a MusicalNote initialized tobe a whole note. This can reduce the number of classes in the system dramatically.It also makesit easier to add a new kind of note to the music editor

# Applicability
Use the Prototype pattern
1. when a system should be independent of how its products are created, composed, and represented; and：
2. when the classes to instantiate are specified at run-time, for example, by dynamic loading; or 
3. to avoid building a class hierarchy offactories that parallels the class hierarchy of products; or 
4. when instances of a class can have one of only a few different combinations of state. It may be more convenient to install a corresponding number of prototypes and clone them rather than instantiating the class manually, each time with the appropriate state.

# Structure
![[原型模式结构.png]]