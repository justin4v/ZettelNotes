# 语言影响思维
如果要问现代数学最重要的概念是什么，那毫无疑问就是函数了，或者更确切地说，是映射。泛函这个词，或许对非数学系的同学来说有些陌生，但如果写成英语 functional, 看起来就眼熟多了。狭隘一点地说，泛函就是以函数为参数，返回值是数值的一类函数。看到这里相信不少同学都发现了，这就是在很多语言中被称为高阶函数（high-order function）的那个东西。泛函在数学中是如此普遍的概念，现代数学几乎无处不会用到。数学家们很自然地在集合上添加运算，构造空间；从一个空间映射到另一个空间，创造泛函。对泛函做变换，构造泛函的泛函等等。

为什么我要在这里提到数学和泛函？因为在我看来， lisp 是一门以表达数学为己任的语言。正如 SICP 中希望表达的一种观点：语言会影响思维。如果数学推理过程中最频繁应用到的泛函，在计算机语言中却没有对应的表达，换言之数学思维不能很自然地表述为计算机语言的话，那么计算机对于数学研究的意义就显得很可疑了，毕竟那时候的计算机可不是用来玩大菠萝3的。所以这里就有了两拨人，务实的一拨人开发出了 fortran ，力主解决数值计算；务虚的一拨人则创造了 lisp ，试图一举解决符号计算的难题。在 John McCarthy 所作的 history of lisp 中这样写到： 

> Then mathematical neatness became a goal and led to pruning some features from the core of the language.（保证数学上的简洁性成为我们的目标，并因此拒绝了将一些特性加入到语言核心中。） 
> 
> This was partly motivated by esthetic reasons and partly by the belief that it would be easier to devise techniques for proving programs correct if the semantics were compact and without exceptions.（这部分是基于美学上的考虑，部分是因为我们相信，紧凑而没有特例的语法才更有可能设计出一种从数学上证明程序正确的方法。）