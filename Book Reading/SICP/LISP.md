# 语言影响思维
如果要问现代数学最重要的概念是什么，那毫无疑问就是**函数**了，或者更确切地说，是**映射**。 

狭隘一点地说，泛函就是以函数为参数，返回值是数值的一类函数。其实就是在很多语言中的**高阶函数（high-order function）**。

泛函在数学中是如此普遍的概念，现代数学几乎无处不会用到。数学家们很自然地在集合上添加运算，构造空间；从一个空间映射到另一个空间，创造泛函。对泛函做变换，构造泛函的泛函等等。

为什么我要在这里提到数学和泛函？因为 **Lisp 是一门以表达数学为己任的语言**。

正如 SICP 中希望表达的一种观点：语言会影响思维。
如果数学推理过程中最频繁应用到的泛函，在计算机语言中却没有对应的表达，换言之数学思维不能很自然地表述为计算机语言的话，那么计算机对于数学研究的意义就显得很可疑了。

所以这里就有了两拨人，*务实的一拨人开发出了 fortran ，力主解决数值计算*；务虚的一拨人则*创造了 lisp ，试图一举解决符号（symbol）计算的难题*。

在 John McCarthy 所作的 history of lisp 中这样写到： 
> Then mathematical neatness became a goal and led to pruning some features from the core of the language.（保证*数学上的简洁性*成为我们的目标，并因此拒绝了将一些特性加入到语言核心中。） 
> 
> This was partly motivated by esthetic reasons and partly by the belief that it would be easier to devise techniques for proving programs correct if the semantics were compact and without exceptions.（这部分是基于美学上的考虑，部分是因为我们相信，*紧凑而没有特例的语法*才更有可能设计出一种从数学上证明程序正确的方法。）

参考
1. [向热爱计算机科学的你推荐SICP](http://www.nowamagic.net/librarys/veda/detail/1905)
2. *The Little Schemer*
3. *The Seasoned Schemer*


# Lisp

Lisp 名字来自于**列表处理（LISt Processing）**，其设计是为了提供**符号计算**的能力，以解决一些程序设计问题，如符号微分和积分。
并提供了新的数据对象，原子 atom 和 列表 list。


# 特性
**计算过程的Lisp 描述（称为过程）本身又可以作为 Lisp 的数据进行表示和操作。**
Lisp descriptions of processes,called *procedures* , can themselves be represented and manipulated as Lisp data. 

这一特性的重要性在于：
1. 现存的许多强大的设计技术，都依赖于**填平*被动的数据*和*主动的过程*之间的传统划分的能力**，LISP可将过程作为数据进行处理的灵活性，使其成为探索这些技术的最方便的语言之一。
	The importance of this is that there are powerful program-design techniques that rely on the ability to blur the traditional distinction between “passive” data and “active” processes。
2. 能将过程表示为数据的能力，也使 Lisp 成为**编写那些必须将其他程序作为数据去操作的程序的最佳语言**，例如支撑计算机语言的解释器和编译器。
	 The ability to represent procedures as data also makes Lisp an excellent language for writing programs that must manipulate other programs as data, such as the interpreters and compilers that support computer languages.
