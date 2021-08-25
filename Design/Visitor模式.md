## Visitor
**KEYS**
- 多个元素；
- 业务规则要求遍历多个元素，如统计；
- 元素规则不变，对其操作可能变化。

**DES**
角色有：
1. IElement，元素接口，定义接受访问的规范： accept(IVisitor);
2. IVisitor，访问者接口，

