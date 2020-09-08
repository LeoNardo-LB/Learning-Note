# tring的不变性

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

>   