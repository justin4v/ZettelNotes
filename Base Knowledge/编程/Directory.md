# Directory Concepts

Many naming services are extended with a _directory service_. A directory service associates names with objects and also associates such objects with _attributes_.

**directory service = naming service + objects containing attributes**

You not only can look up an object by its name but also get the object's attributes or _search_ for the object based on its attributes.

![[directory示例.png]]

An example is the telephone company's directory service. It maps a subscriber's name to his address and phone number. A computer's directory service is very much like a telephone company's directory service in that both can be used to store information such as telephone numbers and addresses. The computer's directory service is much more powerful, however, because it is available online and can be used to store a variety of information that can be utilized by users, programs, and even the computer itself and other computers.

A _directory object_ represents an object in a computing environment. A directory object can be used, for example, to represent a printer, a person, a computer, or a network. A directory object contains _attributes_ that describe the object that it represents.

## Attributes

A directory object can have _attributes_. For example, a printer might be represented by a directory object that has as attributes its speed, resolution, and color. A user might be represented by a directory object that has as attributes the user's e-mail address, various telephone numbers, postal mail address, and computer account information.

An attribute has an _attribute identifier_ and a set of _attribute values_. An attribute identifier is a token that identifies an attribute independent of its values. For example, two different computer accounts might have a "mail" attribute; "mail" is the attribute identifier. An attribute value is the contents of the attribute. The email address, for example, might have:

Attribute Identifier : Attribute Value
                 mail   john.smith@example.com

## Directories and Directory Services

A _directory_ is a connected set of directory objects. A _directory service_ is a service that provides operations for creating, adding, removing, and modifying the attributes associated with objects in a directory. The service is accessed through its own interface.

Many examples of directory services are possible.

Network Information Service (NIS)

NIS is a directory service available on the UNIX operating system for storing system-related information, such as that relating to machines, networks, printers, and users.

[Oracle Directory Server](http://www.oracle.com/technetwork/testcontent/index-085178.html)

The Oracle Directory Server is a general-purpose directory service based on the Internet standard [LDAP](http://www.ietf.org/rfc/rfc2251.txt).

## Search Service

You can look up a directory object by supplying its name to the directory service. Alternatively, many directories, such as those based on the LDAP, support the notion of _searches_. When you search, you can supply not a name but a _query_ consisting of a logical expression in which you specify the attributes that the object or objects must have. The query is called a _search filter_. This style of searching is sometimes called _reverse lookup_ or _content-based searching_. The directory service searches for and returns the objects that satisfy the search filter.

For example, you can query the directory service to find:

-   all users that have the attribute "age" greater than 40 years.
-   all machines whose IP address starts with "192.113.50".

## Combining Naming and Directory Services

Directories often arrange their objects in a hierarchy. For example, the LDAP arranges all directory objects in a tree, called a _directory information tree (DIT)_. Within the DIT, an organization object, for example, might contain group objects that might in turn contain person objects. When directory objects are arranged in this way, they play the role of naming contexts in addition to that of containers of attributes.

# 参考
[Directory Concepts](https://docs.oracle.com/javase/tutorial/jndi/concepts/directory.html)