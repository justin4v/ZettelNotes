# Intent 
Ensure a class only has one instance, and provide a global point of access to it

# Motivation
It'simportant forsome classes to have exactly one instance.
How do we ensure that a class has only one instance and that the instance is easily accessible? 
1. A global variable makes an object accessible, but it doesn't keep you from instantiating multiple objects. 
2. A better solution is to make the classitself responsible for keeping track ofits sole instance.

# Applicability 
Use the Singleton pattern when •
- there must be exactly one instance of a class, and it must be accessible to clients from a well-known access point. 
- when the sole instance should be extensible by subclassing, and clients should be able to use an extended instance without modifying their code.

# Structure
![[单例模式结构.png]]

# Participants 
- **Singleton**
	- defines an Instance operation that lets clients access its unique instance. Instance is a class operation (that is, a static member function in C++). 
	- may b e responsible for creating its own unique instance.

# Collaborations
- Clients access a Singleton instance solely through Singleton's Instance operation

# Consequences
The Singleton pattern has several benefits: 
1. **Controlled access to sole instance**. Because the Singleton class encapsulates its sole instance, it can have strict control over how and when clients access it. 
2. **Reduced name space**. The Singleton pattern is an improvement over global variables. It avoids polluting the name space with global variables that store sole instances. 
3. **Permits refinement of operations and representation**.The pattern makesit easy to change your mind and allow more than one instance of the Singleton class. Moreover, you can use the same approach to control the number of instances that the application uses.

# Implementation
1. Ensuring a unique instance
2. 
