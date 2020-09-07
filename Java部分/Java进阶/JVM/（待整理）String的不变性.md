# String的不变性

#### String不变性演示

```java
public class StringTest01 {
   
   @Test
   public void test() {
      String str = "Hello";
      System.out.println("未传参前str的hashCode：" + str.hashCode());
      exchange(str);
      System.out.println("传参后str的hashCode：" + str.hashCode());
   }
   
   public void exchange(String str) {
      System.out.println("传递到方法后，str被修改前的hashCode：" + str.hashCode());
      str = "World";
      System.out.println("传递到方法后，str被修改后的hashCode：" + str.hashCode());
   }
}

/*执行结果
未传参前str的hashCode：69609650
传递到方法后，str被修改前的hashCode：69609650
传递到方法后，str被修改后的hashCode：83766130
传参后str的hashCode：69609650
*/
```

解释：

两个前提：

   1. String类型传参为值传递
   2. String在常量池中只保留一份

因此就算是值传递，由于常量池只会保留一份相同的字符串，因此方法内被修改的String依旧指向原来的那个String对象，地址值不变。

由于代码修改String，因此str指向新创建出来的字符串，地址值改变。

由于是值传递，因此方法外作为实参的String对象存储的地址值不变。