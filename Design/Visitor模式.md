## Visitor
**KEYS**
- 多个元素；
- 业务规则要求遍历多个元素，如统计；
- 元素规则不变，对其操作可能变化。

**DES**
角色有：
1. IElement，元素接口，定义接受访问的规范方法： accept(IVisitor);
2. IVisitor，访问者接口，定义访问具体元素的规范方法：visit(Element)，因为需要调用Element的对象方法，参数需要是IElement的具体实现；
3. Element，IElement的具体实现，包含具体业务方法；
4. Visitor，

