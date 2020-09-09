# 字符串String

## String的特点

1.  字符串不可变，内部存储结构为 ` private final char value[]` 

    >   在JDK9及其之后，内部使用byte[]数组

2.  字符串分为字面量存储与String对象形式存储

    -   字面量存储在字符串常量池中
    -   String对象形式存储在堆中

3.  字符串类String被声明为final，不可继承

## 字符串的常用方法

### 常用方法

| 方法名                                | 说明                                                  |
| ----------------- | -------------- |
| boolean isEmpty() | 字符串是否为空 |
|int length()|返回字符串的长度|
|String concat(xx)|拼接，等价于+|
|boolean equals(Object obj)|比较字符串是否相等，区分大小写|
|boolean equalsIgnoreCase(Object obj|比较字符串是否相等，区分大小写|
|int compareTo(String other)|比较字符串大小，区分大小写，按照Unicode编码值比较大小|
|int compareToIgnoreCase(String other)|比较字符串大小，不区分大小写|
|String toLowerCase()|将字符串中大写字母转为小写|
|String toUpperCase()|将字符串中小写字母转为大写|
|String trim()|去掉字符串前后空白符|

### 查找方法

| 方法名               | 说明                                                         |
| ----------------- | -------------- |
|boolean contains(xx)|是否包含xx|
|int indexOf(xx)|从前往后找当前字符串中xx，即如果有返回第一次出现的下标，要是没有返回-1|
|int lastIndexOf(xx)|从后往前找当前字符串中xx，即如果有返回最后一次出现的下标，要是没有返回-1|

### 字符串截取方法
| 方法名                                         | 说明                                                         |
| ----------------- | -------------- |
| String substring(int beginIndex) | 返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。 |
|String substring(int beginIndex, int endIndex)|返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。|

### 字符相关方法

| 方法名                                                       | 说明                                    |
| ------------------ | --------------------- |
| char charAt(index) | 返回[index]位置的字符 |
|char[] toCharArray()| 将此字符串转换为一个新的字符数组返回|
|String(char[] value)|返回指定数组中表示该字符序列的 String。|
|String(char[] value, int offset, int count)|返回指定数组中表示该字符序列的 String。|
|static String copyValueOf(char[] data)| 返回指定数组中表示该字符序列的 String|
|static String copyValueOf(char[] data, int offset, int count)|返回指定数组中表示该字符序列的 String|
|static String valueOf(char[] data, int offset, int count) |返回指定数组中表示该字符序列的String|
|static String valueOf(char[] data) |返回指定数组中表示该字符序列的 String|

### 编码与解码方法

| 方法名                                                       | 说明                                                       |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| byte[] getBytes()                                            | 编码，把字符串变为字节数组，按照平台默认的字符编码进行编码 |
| byte[] getBytes(字符编码方式)                                | 按照指定的编码方式进行编码                                 |
| new String(byte[] ) 或 new String(byte[], int, int)          | 解码，按照平台默认的字符编码进行解码                       |
| new String(byte[]，字符编码方式 ) 或 new String(byte[], int, int，字符编码方式) | 解码，按照指定的<br/>编码方式进行解码                      |

### 开头与结尾

| 方法名                 | 说明         |
| ---------------------- | ------------ |
| boolean startsWith(xx) | 是否以xx开头 |
| boolean endsWith(xx)   | 是否以xx结尾 |

### 替换

| 方法名 | 说明 |
| ------ | ---- |
|String replace(xx,xx)|不支持正则|
|String replaceFirst(正则，value)|替换第一个匹配部分|
|String repalceAll(正则， value)|替换所有匹配部分|

### 拆分

| 方法名               | 说明                 |
| -------------------- | -------------------- |
| String[] split(正则) | 按照某种规则进行拆分 |

### 正则匹配

```
字符类
[abc] ： a 、 b  或 c （简单类）
[^abc] ：任何字符，除了 a 、 b  或 c （否定）
[a-zA-Z] ： a  到 z  或 A  到 Z ，两头的字母包括在内（范围）
预定义字符类
. ：任何字符（与行结束符可能匹配也可能不匹配）
\d ：数字： [0-9]
\D ：非数字： [^0-9]
\s ：空白字符： [ \t\n\x0B\f\r]
\S ：非空白字符： [^\s]
\w ：单词字符： [a-zA-Z_0-9]
\W ：非单词字符： [^\w]
POSIX 字符类（仅 US-ASCII）
\p{Lower} 小写字母字符：[a-z]
\p{Upper} 大写字母字符：[A-Z]
\p{ASCII} 所有 ASCII：[\x00-\x7F]
\p{Alpha} 字母字符：[\p{Lower}\p{Upper}]
\p{Digit}  十进制数字：[0-9]
\p{Alnum} 字母数字字符：[\p{Alpha}\p{Digit}]
\p{Punct}  标点符号：!"#$%&'()*+,-./:;<=>?@[]^_`{|}~
\p{Blank}  空格或制表符：[ \t]
边界匹配器
^ ：行的开头
$ ：行的结尾
Greedy 数量词
X ? ：X，一次或一次也没有
X * ：X，零次或多次
X + ：X，一次或多次
X { n } ：X，恰好 n 次
X { n ,} ：X，至少 n 次
X { n , m } ：X，至少 n 次，但是不超过 m 次
Logical 运算符
XY：X 后跟 Y
X | Y：X 或 Y
( X ) ：X，作为捕获组
特殊构造（非捕获）
(?:X) X，作为非捕获组
(?=X) X，通过零宽度的正 lookahead
(?!X) X，通过零宽度的负 lookahead
(?<=X) X，通过零宽度的正 lookbehind
(?<!X) X，通过零宽度的负 lookbehind
(?>X) X，作为独立的非捕获组
```



## 字符串的内存分析

1.  字面量直接存储在字符串常量池中
2.  String对象存储在堆中
3.  常量+常量：结果是常量池
4.  常量与变量 或 变量与变量：结果是堆
5.  intern方法有jvm版本的区别，这里不再深入分析，jdk8中执行原理是如果字符串常量池有内容相同的字符串则直接返回，否则把堆中创建的字符串引用放入字符串常量池，返回此引用，总之所有版本都是通过字符串常量池返回的内容。

```java
@Test
public void test1(){
    String s1 = "hello";
    String s2 = "hello";
    String s3 = new String("hello");

}

@Test
public void test02(){
    String s1 = "hello";
    String s2 = "world";
    String s3 = "helloworld";

    String s4 = s1 + "world";//s4字符串内容也helloworld，s1是变量，"world"常量，变量 + 常量的结果在堆中
    String s5 = s1 + s2;//s5字符串内容也helloworld，s1和s2都是变量，变量 + 变量的结果在堆中
    String s6 = "hello" + "world";//常量+常量，编译期经过优化，跟s3完全相同的情况。

    System.out.println(s3 == s4);//false
    System.out.println(s3 == s5);//false
    System.out.println(s3 == s6);//true
}

@Test
public void test03(){
    final String s1 = "hello";
    final String s2 = "world";
    String s3 = "helloworld";

    String s4 = s1 + "world";//s4字符串内容也helloworld，s1是常量，"world"常量，常量+ 常量 结果在常量池中
    String s5 = s1 + s2;//s5字符串内容也helloworld，s1和s2都是常量，常量+ 常量 结果在常量池中
    String s6 = "hello" + "world";//常量+ 常量 结果在常量池中，因为编译期间就可以确定结果

    System.out.println(s3 == s4);//true
    System.out.println(s3 == s5);//true
    System.out.println(s3 == s6);//true
}

@Test
public void test04(){
    String s1 = "hello";
    String s2 = "world";
    String s3 = "helloworld";

    String s4 = (s1 + "world").intern();//如果常量池已经有“helloworld”，直接返回，否则把拼接结果的引用放到常量池中
    String s5 = (s1 + s2).intern();

    System.out.println(s3 == s4);//true
    System.out.println(s3 == s5);//true
}
```

