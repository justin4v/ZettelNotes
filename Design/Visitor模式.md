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
4. Visitor，IVisitor具体实现，调用/组合Element方法获取结果；
5. ObjectStruture，多个Element的容器，一般用于遍历。


UML类图如下：
![[Visitor模式UML类图.png]]

### 优点
1. 符合单一职责 SRP：Element 类负责元素的加载，Visitor负责访问元素，并进行数据的处理；
2. 扩展性好：增加行为，只需要