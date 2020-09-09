# StringBuilder&StringBuffer

## 与String区别 

因为String对象是不可变对象，虽然可以共享常量对象，但是对于频繁字符串的修改和拼接操作，效率极低。因此，JDK又在java.lang包提供了可变字符序列StringBuilder和StringBuffer类型。

-   StringBuffer：老的，线程安全的（因为它的方法有synchronized修饰）
-   StringBuilder：线程不安全的

## 常用API

常用的API，StringBuilder、StringBuffer的API是完全一致的

| 方法名 | 说明 |
| ------ | ---- |
|StringBuffer append(xx)|拼接，追加|
|StringBuffer insert(int index, xx)|在[index]位置插入xx|
|StringBuffer delete(int start, int end)|删除[start,end)之间字符|
|StringBuffer deleteCharAt(int index)|删除[index]位置字符|
|void setCharAt(int index, xx)|替换[index]位置字符|
|StringBuffer reverse()|反转|
|void setLength(int newLength) |设置当前字符序列长度为newLength|
|StringBuffer replace(int start, int end, String str)|替换[start,end)范围的字符序列为str|
|int indexOf(String str)|在当前字符序列中查询str的第一次出现下标|
|int indexOf(String str, int fromIndex)|在当前字符序列[fromIndex,最后]中查询str的第一次出现下标|
|int lastIndexOf(String str)|在当前字符序列中查询str的最后一次出现下标|
|int lastIndexOf(String str, int fromIndex)|在当前字符序列[fromIndex,最后]中查询str的最后一次出现下标|
|String substring(int start)|截取当前字符序列[start,最后]|
|String substring(int start, int end)|截取当前字符序列[start,end)|
|String toString()|返回此序列中数据的字符串表示形式|

#### 测试

```java
@Test
public void test6(){
    StringBuilder s = new StringBuilder("helloworld");
    s.setLength(30);
    System.out.println(s);
}

@Test
public void test5(){
    StringBuilder s = new StringBuilder("helloworld");
    s.setCharAt(2, 'a');
    System.out.println(s);
}

@Test
public void test4(){
    StringBuilder s = new StringBuilder("helloworld");
    s.reverse();
    System.out.println(s);
}

@Test
public void test3(){
    StringBuilder s = new StringBuilder("helloworld");
    s.delete(1, 3);
    s.deleteCharAt(4);
    System.out.println(s);
}

@Test
public void test2(){
    StringBuilder s = new StringBuilder("helloworld");
    s.insert(5, "java");
    s.insert(5, "chailinyan");
    System.out.println(s);
}

@Test
public void test1(){
    StringBuilder s = new StringBuilder();
    s.append("hello").append(true).append('a').append(12).append("atguigu");
    System.out.println(s);
    System.out.println(s.length());
}
```

