# RTTI
## Conception
RTTI（Run-Time Type Identification）运行时类型识别，其作用是**在运行时识别一个对象的类型和类的信息**。**运行期间，JVM 通过 [[java.lang.Class]]对象记录每个对象的RTTI**。

RTTI 和 反射的区别：
1. RTTI（传统RTTI）：在**编译期**知道要解析的类型；
2.  [[反射]]（新型"RTTI"）：在**运行期**知道要解析的类型。

相同点：本质都是去动态的获取类的信息。

## Example

### RTTI
有两种方式可以获取到 Class 对象：
```java
public static void main(String[] args)
{
	try {
		//第一种方式
		Class rtti = Class.forName("com.jomoo.test.rtti.RTTI");
		//第二种方式
		Class type=RTTI.class;
	} catch (ClassNotFoundException e) {
		e.printStackTrace();
	}
}
```

RTTI 的方式必须在*写程序的时候就知道了类的名字*，才能获取到Class对象对这个类的引用。利用这个引用，得到大量关于这个类的信息，包括接口，父类，方法，静态成员，甚至是像newInstance() 方法这样的一个实现“虚拟构造器”的一种方式。
 
```java
RTTI r =(RTTI)rtti.newInstance();//newInstance的类必须要有一个缺省构造器
```

 另外，instanceof 关键字其实使用了 Class 对象比较，并且返回一个布尔值。

```java
Object o = rtti.newInstance();
if (o instanceof RTTI){
	System.out.println(true);//这里需要注意的是，如果 o 是 RTTI的子类的话，返回的也会true;
}
```

### 反射

Java 中有时候在编译的时候根本无法获知这个对象所属的类，需要利用Java的反射机制。
示例如下：

```java
public class RTTI
{
    private  static final String usage="usage";
    private static Pattern pattern=Pattern.compile("\\w+\\.");

    public static void main(String[] args)
    {
        if (args.length<1){
            System.out.println(usage);
            System.exit(0);
        }
        int lines=0;
        try {
		   // 这里类的名字在编译的时候是无法得知的，只有在运行的时候动态传进去
		   // 用了反射机制
            Class c = Class.forName(args[0]);
            Method[] method = c.getMethods();
            Constructor[] constructors = c.getConstructors();
            if (args.length==1){
                for (int i=0;i<method.length;i++){
                    System.out.println(pattern.matcher(method[i].toString()).replaceAll(""));
                }
                for (int i=0;i<constructors.length;i++){
                    System.out.println(pattern.matcher(constructors[i].toString()).replaceAll(""));
                }
                lines=method.length+constructors.length;
            }else {
                for (int i=0;i<method.length;i++){
                    if (method[i].toString().indexOf(args[1])!=-1){
                        System.out.println(pattern.matcher(method[i].toString()).replaceAll(""));
                    }
                    lines++;
                }
                for (int i=0;i<constructors.length;i++){
                    if (constructors[i].toString().indexOf(args[1])!=-1){
                        System.out.println(pattern.matcher(constructors[i].toString()).replaceAll(""));
                    }
                    lines++;
                }
            }
        } catch (ClassNotFoundException e) {
            System.out.println("NO!!!!");
        }
    }
}
```