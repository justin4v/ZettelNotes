# Java 调用 JavaScript
## 简介
JavaScript 调用 Java 方法比较常见。
Java 调用 JavaScript 比较少见（java 功能比较齐全），可以实现 java 中复杂的算数计算

## 步骤
JDK1.6 新增了 `ScriptEngine` ，可直接执行支持的脚本。

> 获得 JavaScript 脚本引擎

```java
ScriptEngineManager scriptEngineManager = new ScriptEngineManager();

ScriptEngine engine = scriptEngineManager.getEngineByName("javascript");
```

> 定义获取变量

```java
// 定义变量
engine.put("msg", "this is msg");

// 获取变量
System.out.println(engine.get("msg"));
```

> 定义并执行方法

```java
// 定义函数
engine.eval("function add (num1, num2) {return num1 + num2;}");

// engine 该类同样实现了 Invocable 接口 : 转型过去拿到 API
Invocable invocable = (Invocable) engine;

// 调用函数
Object invokeFunction = invocable.invokeFunction("add", new Object[] {10, 10});
System.out.println(invokeFunction);
```

> 执行本地 JavaScript 代码

```java
URL resource = JavaScriptDemo.class.getClassLoader().getResource("JavaScriptDemo.js");

FileReader fileReader = new FileReader(resource.getPath());
engine.eval(fileReader);
fileReader.close();
```

> 外部 JavaScript 代码

```javascript
/**
 * 被java调用的外部 JavaScript
 */
function demo() {
    var i = 110;
    var j = 110;
    console.log("this is a i + j : " + (i + j));
}

demo();
```

> 执行复杂的算数表达式

```java
Object eval = engine.eval("10 * 10 + 10");
System.out.println(eval);
```