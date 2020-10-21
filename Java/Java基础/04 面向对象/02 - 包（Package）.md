# 包（Package）

## 包的作用  

1.  可以避免类重名：有了包之后，类的全名称就变为：包.类名
2.  分类组织管理众多的类

    例如：

-   java.lang----包含一些Java语言的核心类，如String、Math、Integer、 System和Thread等，提供常用功能
-   java.net----包含执行与网络相关的操作的类和接口。
-   java.io ----包含能提供多种输入/输出功能的类。
-   java.util----包含一些实用工具类，如集合框架类、日期时间、数组工具类Arrays，文本扫描仪
-   Scanner，随机值产生工具Random。
-   java.text----包含了一些java格式化相关的类
-   java.sql和javax.sql----包含了java进行JDBC数据库编程的相关类/接口
-   java.awt和java.swing----包含了构成抽象窗口工具集（abstract window toolkits）的多个类，这些类被用来构建和管理应用程序的图形用户界面(GUI)。

3.  可以控制某些类型或成员的可见范围

    如果某个类型或者成员的权限修饰缺省的话，那么就仅限于本包使用



## 声明包的语法格式

```java
package 包名;
```

>   注意：
>
>   (1) 必须在源文件的代码首行
>
>   (2) 一个源文件只能有一个声明包的语句

包的命名规范和习惯： 

（1）所有单词都小写，每一个单词之间使用.分割 

（2）习惯用公司的域名倒置 例如：com.atguigu.xxx;



## 如何跨包使用类

前提：被使用的类或成员的权限修饰符是>缺省的，即可见的

1.  使用类型的全名称

    例如：java.util.Scanner input = new java.util.Scanner(System.in);

2.  使用import 语句之后，代码中使用简名称

    import语句告诉编译器到哪里去寻找类。

    import语句的语法格式：

```java
import 包.类名;
import 包.*;
import static 包.类名.静态成员; //后面再讲
```

>   注意：
>
>   1.  使用java.lang包下的类，不需要import语句，就直接可以使用简名称
>   2.  import语句必须在package下面，class的上面
>   3.  当使用两个不同包的同名类时，例如：java.util.Date和java.sql.Date。一个使用全名称，一个使用简名称