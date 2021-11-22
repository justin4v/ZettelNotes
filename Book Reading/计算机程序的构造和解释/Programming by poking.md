#SICP #Fundamental-Of-Programing
# Programming by Poking
Sussman and Abelson, author of \<SICP>

Sussman said that in the 80s and 90s, engineers built complex systems by combining simple and well-understood parts. *The goal of SICP was to provide the abstraction language for reasoning about such systems*.

Sussman pointed out that engineers now routinely write code for complicated hardware that they don’t fully understand (and often can’t understand because of trade secrecy.) The same is true at the software level, since *programming environments consist of gigantic libraries with enormous functionality*.

*He said that programming today is “More like science. You grab this piece of library and you poke at it.*

The *“analysis-by-synthesis”* view of SICP ：where you *build a larger system out of smaller, simple parts*。

[Programming by poking](http://lambda-the-ultimate.org/node/5335)

# Fundamental
1. You need to know about Turing-completeness, and that in addition to Turing machines there are PDAs and FSAs. 
2. You need to know that TMs can do things that PDAs can't (like parse context-sensitive grammars), and that PDAs can to things that FSAs can't (like parse context-free grammars) and that FSAs can parse regular expressions, and that's all they can do.
3. You need some programming language theory：
	1. You need to know what a binding is, and that there are two types of bindings that matter: lexical and dynamic. 
	2. You need to know what an environment is (a mapping between names and bindings) and how environments are chained. 
	3. You need to know how evaluation and compilation are related, and the role that environments play in both processes. 
	4. You need to know the difference between static and dynamic typing. 
	5. You need to know how to compile a high-level language down to an RTL. 
4. For operating systems:
	1. you need to know what a process is, what a thread is, some of the ways in which parallel processes lead to problems, and some of the mechanisms for dealing with those problems, including the fact that some of those mechanisms require hardware support (e.g. atomic test-and-set instructions). 
5. You need a few basic data structures.:
	1. Mainly what you need is to understand that what data structures are really all about is making associative maps that are more efficient for certain operations under certain circumstances.
6. You need a little bit of database knowledge. Again, what you really need to know is that what databases are really all about is dealing with the architectural differences between RAM and non-volatile storage, and that a lot of these are going away now that these architectural differences are starting to disappear.