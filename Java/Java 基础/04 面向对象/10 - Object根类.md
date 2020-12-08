# Object根类

## 如何理解根父类

类 java.lang.Object 是类层次结构的根类，即所有类的父类。每个类都直接或间接继承Object类。

-   Object类型的变量与除Object以外的任意引用数据类型的对象都多态引用
-   所有对象（包括数组）都实现这个类的方法。
-   如果一个类没有特别指定父类，那么默认则继承自Object类。例如：

```java
public class MyClass /*extends Object*/ {
    // ...
}
```



## Object类的API

根据JDK源代码及Object类的API文档，Object类当中包含的方法有11个。今天我们主要学习其中的5个：

#### toString()

public String toString()

①默认情况下，toString()返回的是“对象的运行时类型 @ 对象的hashCode值的十六进制形式"

②通常是建议重写，如果在eclipse中，可以用Alt +Shift + S-->Generate toString()

③如果我们直接System.out.println(对象)，默认会自动调用这个对象的toString()

>   因为Java的引用数据类型的变量中存储的实际上时对象的内存地址，但是Java对程序员隐藏内存地址信息，所以不能直接将内存地址显示出来，所以当你打印对象时，JVM帮你调用了对象的toString()。

##### 例如自定义的Person类：

```java
public class Person {  
    private String name;
    private int age;
    @Override
    public String toString() {
        return "Person{" + "name='" + name + '\'' + ", age=" + age + '}';
    }
    // 省略构造器与Getter Setter
}
```

#### getClass()

public final Class<?> getClass()：获取对象的运行时类型

>   因为Java有多态现象，所以一个引用数据类型的变量的编译时类型与运行时类型可能不一致，因此如果需要查看这个变量实际指向的对象的类型，需要用getClass()方法

```java
public static void main(String[] args) {
    Object obj = new String();
    System.out.println(obj.getClass());//运行时类型
}
```

#### finalize()

protected void finalize()：用于最终清理内存的方法

```java
public class TestFinalize {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            MyData my = new MyData();
        }
        
        System.gc();//通知垃圾回收器来回收垃圾
        
        try {
            Thread.sleep(2000);//等待2秒再结束main，为了看效果
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
class MyData{
    @Override
    protected void finalize() throws Throwable {
        System.out.println("轻轻的我走了...");
    }
}
```

面试题：对finalize()的理解？

-   当对象被GC确定为要被回收的垃圾，在回收之前由GC帮你调用这个方法，不是由程序员手动调用。
-   这个方法与C语言的析构函数不同，C语言的析构函数被调用，那么对象一定被销毁，内存被回收，而finalize方法的调用不一定会销毁当前对象，因为可能在finalize()中出现了让当前对象“复活”的代码
-   每一个对象的finalize方法只会被调用一次。
-   子类可以选择重写，一般用于彻底释放一些资源对象，而且这些资源对象往往时通过C/C++等代码申请的资源内存

#### hashCode()

public int hashCode()：返回每个对象的hash值。

hashCode 的常规协定：

①如果两个对象的hash值是不同的，那么这两个对象一定不相等；

②如果两个对象的hash值是相同的，那么这两个对象不一定相等。

主要用于后面当对象存储到哈希表等容器中时，为了提高存储和查询性能用的。

```java
public static void main(String[] args) {
    System.out.println("Aa".hashCode());//2112
    System.out.println("BB".hashCode());//2112
}
```

#### equals()

public boolean equals(Object obj)：用于判断当前对象this与指定对象obj是否“相等”

①默认情况下，equals方法的实现等价于与“==”，比较的是对象的地址值

②我们可以选择重写，重写有些要求：

-   如果重写equals，那么一定要一起重写hashCode()方法，因为规定：

-   -   如果两个对象调用equals返回true，那么要求这两个对象的hashCode值一定是相等的；
    -   如果两个对象的hashCode值不同的，那么要求这个两个对象调用equals方法一定是false；
    -   如果两个对象的hashCode值相同的，那么这个两个对象调用equals可能是true，也可能是false

-   如果重写equals，那么一定要遵循如下几个原则：

-   -   自反性：x.equals(x)返回true
    -   传递性：x.equals(y)为true, y.equals(z)为true，然后x.equals(z)也应该为true
    -   一致性：只要参与equals比较的属性值没有修改，那么无论何时调用结果应该一致
    -   对称性：x.equals(y)与y.equals(x)结果应该一样
    -   非空对象与null的equals一定是false