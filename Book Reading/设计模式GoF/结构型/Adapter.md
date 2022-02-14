# Intent
- Convert the interface of a class into another interface clients expect. 
- Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

# Also Known As
Wrapper

# Motivation/Example
Sometimes a toolkit class that's designed for reuse isn't reusable only because its interface doesn't match the domain-specific interface an application requires.

Consider for example a drawing editor that lets users draw and arrange graphical
elements (lines, polygons-多边形, text, etc.) into pictures and diagrams. The drawing
editor's key abstraction is the graphical object, which has an editable shape and
can draw itself. The interface for graphical objectsis defined by an abstract class
called Shape. The editor defines a subclass of Shape for each kind of graphical
object: a LineShape class for lines, a PolygonShape class for polygons, and so
forth.

Classes for elementary geometric shapes like LineShape and PolygonShape are rather easy to implement, because their drawing and editing capabilities are inherently limited. But a TextShape subclass that can display and edit text is considerably more difficult to implement, since even basic text editing involves complicated screen update and buffer management. Meanwhile, an off-the-shelf user interface toolkit might already provide a sophisticated TextView class for displaying and editing text. Ideally we'd like to reuse TextView to implement TextShape, but the toolkit wasn't designed with Shape classes in mind. So we can't use TextView and Shape objects interchangeably.

How can existing and unrelated classes like TextView work in an application that
expects classes with a different and incompatible interface? We could change the
TextView class so that it conformsto the Shape interface, but that isn't an option
unless we have the toolkit's source code. Even ifwe did, it wouldn't make sense to
change TextView; the toolkit shouldn't have to adopt domain-specificinterfaces
just to make one application work.

Instead, we could define TextShape so that it adapts the TextView interface to
Shape's. We can do this in one of two ways: 
1. by inheriting Shape's interface and TextView's implementation or 
2. by composing a TextView instance within a TextShape and implementing TextShape in terms of Text View's interface.

These two approaches correspond to the class and object versions ofthe Adapterpattern. We call TextShape an *adapter*.
![[Adapter示例结构.png]]