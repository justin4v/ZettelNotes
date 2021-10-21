# 意图
**Separate the construction of a complex object from its representation** so that the same construction process can create different representations.

# 动机
A reader for the *RTF (Rich Text Format)* document exchange format should be able to convert RTF to many text formats. The reader might convert RTF documents into plain ASCII text or into a text widget that can be edited interactively. 
The *problem*, however, is that the number of possible conversions is open-ended. So it should be easy to add a new conversion without modifying the reader. 

A solution is to configure the RTFReader class with a TextConverter object that converts RTF to another textual representation. Whenever the *RTFReader recognizes an RTFtoken* (either plain text or an RTFcontrol word), it issues a request to the *TextConverter to convert* the token. 

TextConverter objects are responsible both for performing the data conversion and for representing the token in a particular format. Subclasses of TextConverter specialize in different conversions and formats.

UML 示意图如下：

![[Builder示例示意图.png]]

The Builder pattern captures all these relationships. *Each converter class is called a builder in the pattern, and the reader is called the director.* Applied to this example, the Builder pattern separates the *algorithm for interpreting a textual format* (that is, the parser for RTF documents) from *how a converted format gets created and represented*.

# 适用性
Use the Builder pattern when：
- *the algorithm for creating a complex object should be independent of the parts that make up the object and how they're assembled*. 
- *the construction process must allow different representations for the object that's constructed*.

# 结构
![[Builder结构.png]]

## 解释
- Builder (TextConverter) 
	-  specifies an abstract *interface* for creating parts of a Product object. 
-  ConcreteBuilder (ASCIIConverter, TeXConverter, TextWidgetConverter) 
	-  constructs and assembles parts of the product by *implementing* the Builder interface. 
	-  defines and keeps track of the representation it creates. 
	-  provides an interface for retrieving the product (e.g., GetASCIIText, GetTextWidget). 
-  Director (RTFReader) 
	- (recognize the token and) *constructs* an object using the Builder interface. 
-  Product (ASCIIText, TeXText, TextWidget) 
	-  represents the *complex object* under construction. ConcreteBuilder builds the product's internal representation and defines the process by which it's assembled. 
	-  includes classes that define the constituent parts, including interfaces for assembling the parts into the final result.

# 协作
1. The client creates the Director object and configures it with the desired Builder object.
2. Director notifies the builder whenever a part ofthe product should be built. 
3. Builder handles requests from the director and adds parts to the product. 
4. The client retrieves the product from the builder

时序图如下：

![[Builder模式时序图.png]]


# 效果
1. It lets you vary a product's internal representation.
2. It isolates code for construction and representation.
3. It gives you finer control over the construction process.


# 实现
需要考虑的问题：
1. Assembly and construction interface. 
2. Why no abstract classfor products? 
	1. In the common case, the products produced by the concrete builders differ so greatly in their representation that there is little to gain from giving different products a common parent class. 
	2. In the RTF example, the ASCIIText and the TextWidget objects are unlikely to have a common interface, nor do they need one. Because the client usually configures the director with the proper concrete builder, the client is in a position to know which concrete subclass ofBuilder is in use and can handle its products accordingly.
