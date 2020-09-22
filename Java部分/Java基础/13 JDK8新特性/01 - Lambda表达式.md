# Lambda表达式

Lambda表达式为了提高开发效率而生~

## 函数式编程思想

在数学中，函数就是有输入量、输出量的一套计算方案，也就是“拿什么东西做什么事情”。编程中的函数，也有类似的概念，你调用我的时候，给我实参为形参赋值，然后通过运行方法体，给你返回一个结果。对于调用者来做，关注这个方法具备什么样的功能。相对而言，面向对象过分强调“必须通过对象的形式来做事情”，而函数式思想则尽量忽略面向对象的复杂语法——**强调做什么，而不是以什么形式做。**

-   面向对象的思想:

    做一件事情,找一个能解决这个事情的对象,调用对象的方法,完成事情.

-   函数式编程思想:

    只要能获取到结果,谁去做的,怎么做的都不重要,重视的是结果,不重视过程

Java8引入了Lambda表达式之后，Java也开始支持函数式编程。

### Lambda引入：冗余的匿名内部类

当需要启动一个线程去完成任务时，通常会通过 java.lang.Runnable 接口来定义任务内容，并使用java.lang.Thread 类来启动该线程。代码如下：

```java
public class Demo01Runnable {
    public static void main(String[] args) {
        // 匿名内部类
        Runnable task = new Runnable() {
            @Override
            public void run() { // 覆盖重写抽象方法
                System.out.println("多线程任务执行！");
            }
        };
        new Thread(task).start(); // 启动线程
    }
}
```

#### 体验Lambda的更优写法

借助Java 8的全新语法，上述 Runnable 接口的匿名内部类写法可以通过更简单的Lambda表达式达到等效：

```java
public class Demo02LambdaRunnable {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("多线程任务执行！")).start(); // 启动线程
    }
}

```

这段代码和刚才的执行效果是完全一样的，可以在1.8或更高的编译级别下通过。从代码的语义中可以看出：我们启动了一个线程，而线程任务的内容以一种更加简洁的形式被指定。



## 函数式接口

lambda表达式其实就是实现SAM接口的语法糖，所谓SAM接口就是Single Abstract Method，即**该接口中只有一个抽象方法需要实现**，当然该接口可以包含其他非抽象方法。

其实只要满足“SAM”特征的接口都可以称为函数式接口，都可以使用Lambda表达式，但是如果要更明确一点，最好在声明接口时，加上@FunctionalInterface。一旦使用该注解来定义接口，编译器将会强制检查该接口是否确实有且仅有一个抽象方法，否则将会报错。

之前学过的SAM接口中，标记了@FunctionalInterface的函数式接口的有：Runnable，Comparator，FileFilter。

Java8在java.util.function新增了很多函数式接口：主要分为四大类，消费型、供给型、判断型、功能型。基本可以满足我们的开发需求。当然你也可以定义自己的函数式接口。

#### 自定义函数式接口

只要确保接口中有且仅有一个抽象方法即可：

```java
修饰符 interface 接口名称 {
    public abstract 返回值类型 方法名称(可选参数信息);
    // 其他非抽象方法内容
}
```

>接口中抽象方法的 public abstract 可以省略

**自定义函数式接口实例**

声明一个计算器 Calculator 接口，内含抽象方法 calc 可以对两个int数字进行计算，并返回结果。你可以根据自己所需要的情况自行实现计算规则。

```java
public interface Calculator {
    int calc(int a, int b);
}
```

### Java内置的函数式接口

#### 1、消费型接口

消费型接口抽象方法的特点：有形参，无返回值

| 接口名 | 抽象方法 | 描述 |
| ------ | -------- | ---- |
|Consumer |void accept(T t) |接收一个对象用于完成功能|
|BiConsumer<T,U> |void accept(T t, U u) |接收两个对象用于完成功能|
|DoubleConsumer |void accept(double value) |接收一个double值|
|IntConsumer| void accept(int value) |接收一个int值|
|LongConsumer |void accept(long value) |接收一个long值|
|ObjDoubleConsumer |void accept(T t, double value) |接收一个对象和一个double值|
|ObjIntConsumer| void accept(T t, int value)|接收一个对象和一个int值|
|ObjLongConsumer| void accept(T t, long value) |接收一个对象和一个long值|

#### 2、供给型接口

供给型接口抽象方法的特点：无参，有返回值

|接口名 | 抽象方法|  描述 |
| ------ | -------- | ---- |
|Supplier| T get() |返回一个对象|
|BooleanSupplier| boolean getAsBoolean() |返回一个boolean值|
|DoubleSupplier| double getAsDouble() |返回一个double值|
|IntSupplier| int getAsInt() |返回一个int值|
|LongSupplier| long getAsLong() |返回一个long值|

#### 3、判断型接口

判断型接口抽象方法的特点：有参，返回值类型为boolean

|接口名 |抽象方法 |描述|
| ------ | -------- | ---- |
|Predicate| boolean test(T t) |接收一个对象|
|BiPredicate<T,U>| boolean test(T t, U u) |接收两个对象|
|DoublePredicate| boolean test(double value) |接收一个double值|
|IntPredicate| boolean test(int value) |接收一个int值|
|LongPredicate| boolean test(long value) |接收一个long值|

#### 4、功能型接口

功能型接口抽象方法的特点：有形参，有返回值

| 接口名 | 抽象方法 | 描述 |
| ------ | -------- | ---- |
|Function<T,R>| R apply(T t) |接收一个T类型对象，返回一个R类型对象结果|
|UnaryOperator| T apply(T t) |接收一个T类型对象，返回一个T类型对象结果|
|DoubleFunction| R apply(double value) |接收一个double值，返回一个R类型对象|
|IntFunction| R apply(int value) |接收一个int值，返回一个R类型对象|
|LongFunction| R apply(long value) |接收一个long值，返回一个R类型对象|
|ToDoubleFunction| double applyAsDouble(T value) |接收一个T类型对象，返回一个double|
|ToIntFunction| int applyAsInt(T value) |接收一个T类型对象，返回一个int|
|ToLongFunction| long applyAsLong(T value) |接收一个T类型对象，返回一个long|
|DoubleToIntFunction| int applyAsInt(double value) |接收一个double值，返回一个int 结果|
|DoubleToLongFunction| long applyAsLong(double value) |接收一个double值，返回一个long结果|
|IntToDoubleFunction| double applyAsDouble(int value)|接收一个int值，返回一个double结果|
|IntToLongFunction|long applyAsLong(int value)|接收一个int值，返回一个long结果|
|LongToDoubleFunction| double applyAsDouble(long value)|接收一个long值，返回一个double结果|
|LongToIntFunction| int applyAsInt(long value)| 接收一个long值，返回一个int结果|
|DoubleUnaryOperator |double applyAsDouble(double operand) |接收一个double值，返回一个 double|
|IntUnaryOperator| int applyAsInt(int operand) |接收一个int值，返回一个int结果|
|LongUnaryOperator|long applyAsLong(long operand)|接收一个long值，返回一个long结果|
|BiFunction<T,U,R> |R apply(T t, U u) |接收一个T类型和一个U类型对象，返回一个R类型对象结果|
|BinaryOperator| T apply(T t, T u) |接收两个T类型对象，返回一个T类型对象结果|
|ToDoubleBiFunction<T,U> |double applyAsDouble(T t, U u)|接收一个T类型和一个U类型对象，返回一个double|
|ToIntBiFunction<T,U>| int applyAsInt(T t, U u) |接收一个T类型和一个U类型对象，返回一个int|
|ToLongBiFunction<T,U>| long applyAsLong(T t, U u) |接收一个T类型和一个U类型对象，返回一个long|
|DoubleBinaryOperator| double applyAsDouble(double left, double right)|接收两个double值，返回一个double结果|
|IntBinaryOperator| int applyAsInt(int left, int right) |接收两个int值，返回一个int结果|
|LongBinaryOperator|long applyAsLong(long left, long right)|接收两个long值，返回一个long结果|



## Lambda表达式语法

Lambda表达式是用来给【函数式接口】的变量或形参赋值用的。

其实本质上，Lambda表达式是用于实现【函数式接口】的“抽象方法”

**Lambda表达式语法格式**

```java
(形参列表) -> {Lambda体}
```

| 语法格式 | 代码示例 |
| -------- | -------- |
| 语法格式一：无参，无返回值                                   | `Runnable r1 = () -> {System.out.println("Hello Lambda");};` |
| 语法格式二：Lambda需要一个参数，但是没有返回值               | `Consumer<String> con = (String s) -> {System.out.println(s);};` |
| 语法格式三：数据类型可以省略，因为可由编译器推断得出，称为“类型推断” | `Consumer<String> con = (s) -> {System.out.println(s);};` |
| 语法格式四：Lambda若只需要一个参数，参数的小括号可以省略     | `Consumer<String> con = s -> {System.out.println(s);};` |
| 语法格式五：Lambda需要两个以上的参数，多条执行语句，并且可以有返回值 | `Consumer<Integer> con = (x,y) -> {System.out.println(x);` |
| 语法格式六：当Lambda体只有一条语句时，return与大括号         | `Comparator<Integer> com = (o1, o2) -> o1.compareTo(o2); ` |

```java
// 语法格式一：无参，无返回值
@Test
public void TestPOne(){
    Runnable r1 = new Runnable() {
        @Override
        public void run() {
            System.out.println(" I'm one");
        }
    };
    r1.run();

    System.out.println("=======lambda======");
    Runnable r2 = () -> System.out.println("I'm lambda'One");
    r2.run();
}


// 语法格式二：Lambda 需要一个参数，但是没有返回值。
@Test
public void TestTwo(){
    Consumer<String> con1 = new Consumer<String>() {
        @Override
        public void accept(String s) {
            System.out.println(s);
        }
    };
    con1.accept("老王");

    System.out.println("=======lambda======");
    Consumer<String> con2 = (String s1) -> {System.out.println(s1);};
    con2.accept("我是lambda老王");
}


//语法格式三：数据类型可以省略，因为可由编译器推断得出，称为“类型推断”
// 类型推断想我们之前遇到的泛型和数组一样
/* Consumer<String> con = new Consumer<>();  // 省略后面的String类型
        int [] len = new int [] {1,2,3}; 可以 写成 int [] len = {1,2,3}
     */
@Test
public void TestThree(){
    Consumer<String> con = new Consumer<String>() {
        @Override
        public void accept(String s) {
            System.out.println(s);
        }
    };
    con.accept("老王头");

    System.out.println("=======lambda======");
    Consumer<String> con1 = (s) -> { System.out.println(s); };
    con1.accept("吾乃lambda老王头");
}


//语法格式四：Lambda 若只需要一个参数时，参数的小括号可以省略
@Test
public void TestFour(){
    Consumer<String> con = new Consumer<String>() {
        @Override
        public void accept(String s) {
            System.out.println(s);
        }
    };
    con.accept("老刘");
    System.out.println("=======lambda======");
    Consumer<String> con1 = s -> {System.out.println(s);};
    con1.accept("吾乃lambda老刘");

}


//语法格式五：Lambda 需要两个或以上的参数，多条执行语句，并且可以有返回值
@Test
public void TestFive(){
    Comparator<Integer> com = new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            System.out.println(o1);
            System.out.println(o2);
            return o1.compareTo(o2);
        }
    };
    System.out.println(com.compare(13, 14));

    System.out.println("=======lambda======");
    Comparator<Integer> com1 = (l1,l2) -> {
        System.out.println(l1);
        System.out.println(l2);
        return l1.compareTo(l2);
    };
    System.out.println(com1.compare(11, 10));
}


//语法格式六：当 Lambda 体只有一条语句时，return 与大括号若有，都可以省略
@Test
public void TestSix(){

    // 原有
    Comparator<Integer> com = (o1, o2) -> {return o1.compareTo(o2);};
    System.out.println(com.compare(10, 11));

    //优化后
    Comparator<Integer> com1 = (l1, l2) -> l1.compareTo(l2);
    System.out.println(com1.compare(20, 10));

}
```

说明：

-   (形参列表)它就是你要赋值的函数式接口的抽象方法的(形参列表)，照抄
-   {Lambda体}就是实现这个抽象方法的方法体
-   ->称为Lambda操作符（减号和大于号中间不能有空格，而且必须是英文状态下半角输入方式）

优化：Lambda表达式可以精简

-   当{Lambda体}中只有一句语句时，可以省略{}和{;}
-   当{Lambda体}中只有一句语句时，并且这个语句还是一个return语句，那么return也可以省略，但是如果{;}没有省略的话，return是不能省略的
-   (形参列表)的类型可以省略
-   当(形参列表)的形参个数只有一个，那么可以把数据类型和()一起省略，但是形参名不能省略
-   当(形参列表)是空参时，()不能省略



## 方法引用与构造器引用

### 方法引用

类名或对象名，后面跟 `::` ，然后跟方法名称。

```java
interface Callable { // [1]
    void call(String s);
}

class Describe {
    void show(String msg) { // [2]
        System.out.println(msg);
    }
}

public class MethodReferences {
    static void hello(String name) { // [3]
        System.out.println("Hello, " + name);
    }
    static class Description {
        String about;
        Description(String desc) { about = desc; }
        void help(String msg) { // [4]
            System.out.println(about + " " + msg);
        }
    }
    static class Helper {
        static void assist(String msg) { // [5]
            System.out.println(msg);
        }
    }
    public static void main(String[] args) {
        Describe d = new Describe();
        Callable c = d::show; // [6]
        c.call("call()"); // [7]

        c = MethodReferences::hello; // [8]
        c.call("Bob");

        c = new Description("valuable")::help; // [9]
        c.call("information");

        c = Helper::assist; // [10]
        c.call("Help!");
    }
}
```

**[1]** 我们从单一方法接口开始（同样，你很快就会了解到这一点的重要性）。

**[2]** `show()` 的签名（参数类型和返回类型）符合 **Callable** 的 `call()` 的签名。

**[3]** `hello()` 也符合 `call()` 的签名。

**[4]** `help()` 也符合，它是静态内部类中的非静态方法。

**[5]** `assist()` 是静态内部类中的静态方法。

**[6]** 我们将 **Describe** 对象的方法引用赋值给 **Callable** ，它没有 `show()` 方法，而是 `call()` 方法。 但是，Java 似乎接受用这个看似奇怪的赋值，因为方法引用符合 **Callable** 的 `call()` 方法的签名。

**[7]** 我们现在可以通过调用 `call()` 来调用 `show()`，因为 Java 将 `call()` 映射到 `show()`。

**[8]** 这是一个**静态**方法引用。

**[9]** 这是 **[6]** 的另一个版本：对已实例化对象的方法的引用，有时称为*绑定方法引用*。

**[10]** 最后，获取静态内部类中静态方法的引用与 **[8]** 中通过外部类引用相似。

#### Runnable接口

Runnable接口的抽象方法 `run()` 不带参数，也没有返回值。因此可以使用Lambda表达式与方法引用作为Runnable

方法引用与 Runnable 接口的结合使用

```java
class Go {
    static void go() {
        System.out.println("Go::go()");
    }
}

public class RunnableMethodReference {
    public static void main(String[] args) {

        new Thread(new Runnable() {
            public void run() {
                System.out.println("Anonymous");
            }
        }).start();

        new Thread(
            () -> System.out.println("lambda")
        ).start();

        new Thread(Go::go).start();
    }
}

/*输出结果
Anonymous
lambda
Go::go()
*/
```

#### 未绑定的方法引用

关联对象的普通（非静态）方法。 使用未绑定的引用时，必须先提供对象

```java
// 没有方法引用的对象

class X {
    String f() { return "X::f()"; }
}

interface MakeString {
    String make();
}

interface TransformX {
    String transform(X x);
}

public class UnboundMethodReference {
    public static void main(String[] args) {
        // MakeString ms = X::f; // [1]
        TransformX sp = X::f;
        X x = new X();
        System.out.println(sp.transform(x)); // [2]
        System.out.println(x.f()); // 同等效果
    }
}

/*输出结果
X::f()
X::f()
*/
```

### 构造器引用

1.  当Lambda表达式是创建一个对象，并且满足Lambda表达式形参，正好是给创建这个对象的构造器的实参列表。
2.  当Lambda表达式是创建一个数组对象，并且满足Lambda表达式形参，正好是给创建这个数组对象的长度

构造器引用的语法格式：

-   类名::new
-   数组类型名::new