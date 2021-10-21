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