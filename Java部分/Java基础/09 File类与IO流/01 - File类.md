# File类与流的概述

## File概述

File类位于java.io.包下，用于操作文件和目录。File类能新建、删除、重命名文件和目录。

在API中File的解释是文件和目录路径名的抽象表示形式，即File类是文件或目录的路径，而不是文件本
身，因此File类不能直接访问文件内容本身，如果需要访问文件内容本身，则需要使用输入/输出流。

### File类的构造方法

| 方法名 | 方法说明 |
| ------ | -------- |
|public File(String pathname) |通过将给定的路径名字符串转换为抽象路径名来创建新的File实例。|
|public File(String parent, String child) |从父路径名字符串和子路径名字符串创建新的 File实例。|
|public File(File parent, String child) |从父抽象路径名和子路径名字符串创建新的File实例。|

##### 代码示例，有4中创建File类的方式

```java
// 文件路径名
String pathname = "D:\\aaa.txt";
File file1 = new File(pathname);

// 文件路径名
String pathname2 = "D:\\aaa\\bbb.txt";
File file2 = new File(pathname2);

// 通过父路径和子路径字符串
String parent = "d:\\aaa";
String child = "bbb.txt";
File file3 = new File(parent, child);

// 通过父级File对象和子路径字符串
File parentDir = new File("d:\\aaa");
String child = "bbb.txt";
File file4 = new File(parentDir, child);
```

>   一个File对象代表硬盘中实际存在的一个文件或者目录。
>
>   无论该路径下是否存在文件或者目录，都不影响File对象的创建。



## 常用方法

#### 获取文件和目录基本信息的方法

| 方法名 | 方法说明 |
| ------ | -------- |
|public String getName() |返回由此File表示的文件或目录的名称。|
|public long length()|返回由此File表示的文件的长度。|
|public String getPath() |将此File转换为路径名字符串。|
|public long lastModified()|返回File对象对应的文件或目录的最后修改时间（毫秒值）|

```java
public class TestFile {
    public static void main(String[] args) {
        File f = new File("d:/aaa/bbb.txt");
        System.out.println("文件构造路径:"+f.getPath());
        System.out.println("文件名称:"+f.getName());
        System.out.println("文件长度:"+f.length()+"字节");

        File f2 = new File("d:/aaa");
        System.out.println("目录构造路径:"+f2.getPath());
        System.out.println("目录名称:"+f2.getName());
        System.out.println("目录长度:"+f2.length()+"字节");
    }
}

/*输出结果
文件构造路径:d:\aaa\bbb.java
文件名称:bbb.java
文件长度:636字节
文件最后修改时间：2019-07-23T22:01:32.065
目录构造路径:d:\aaa
目录名称:aaa
目录长度:4096字节
文件最后修改时间：2019-07-23T22:01:32.065
*/
```

>   API中说明：length()，表示文件的长度。但是File对象表示目录，则返回值未指定。

#### 各种路径问题

| 方法名 | 方法说明 |
| ------ | -------- |
|public String getPath() |将此File转换为路径名字符串。|
|public String getAbsolutePath() |返回此File的绝对路径名字符串。|
|String getCanonicalPath() |返回此File对象所对应的规范路径名。|

File类可以使用文件路径字符串来创建File实例，该文件路径字符串既可以是绝对路径，也可以是相对路径。
默认情况下，系统总是依据用户的工作路径来解释相对路径，这个路径由系统属性“user.dir”指定，通常也就是运行Java虚拟机时所作的路径。

-   绝对路径：从盘符开始的路径，这是一个完整的路径。
-   相对路径：相对于项目目录的路径，这是一个便捷的路径，开发中经常使用。
    -   IDEA中 单元测试的相对路径的根路径是 **模块**；main方法的相对路径是 **当前项目**
-   规范路径：所谓规范路径名，即对路径中的“..”等进行解析后的路径名

>   Windows中 路径的分隔符为 `\` ，因此再String字符串中使用 `\\` 转译成 `\`

#### 判断功能的方法
| 方法名 | 方法说明 |
| ------ | -------- |
|public boolean exists() |此File表示的文件或目录是否实际存在。|
|public boolean isDirectory() |此File表示的是否为目录。|
|public boolean isFile() |此File表示的是否为文件。|

>   如果文件或目录不存在，那么exists()、isFile()和isDirectory()都是返回true

#### 创建删除功能的方法
| 方法名 | 方法说明 |
| ------ | -------- |
|public boolean createNewFile() |当且仅当具有该名称的文件尚不存在时，创建一个新的空文件。|
|public boolean delete() |删除由此File表示的文件或目录。 只能删除空目录。|
|public boolean mkdir() |创建由此File表示的目录。|
|public boolean mkdirs() |创建由此File表示的目录，包括任何必需但不存在的父目录。|

>   API中说明：delete方法，如果此File表示目录，则目录必须为空才能删除。

#### 目录操作
| 方法名 | 方法说明 |
| ------ | -------- |
|public String[] list() |返回一个String数组，表示该File目录中的所有子文件或目录。|
|public File[] listFiles() |返回一个File数组，表示该File目录中的所有的子文件或目录。|
|public File[] listFiles(FileFilter filter) |返回所有满足指定过滤器的文件和目录。|

过滤器方法说明：

如果给定 filter 为 null，则接受所有路径名。否则，当且仅当在路径名上调用过滤器的FileFilter.accept(java.io.File) 方法返回 true 时，该路径名才满足过滤器。如果当前File对象不表示一个目录，或者发生 I/O 错误，则返回 null。

##### 输出文件夹下所有文件夹及其行数示例

```java
public class IOTest01 {

    @Test
    public void test1() throws IOException, ExecutionException, InterruptedException {

        //15683

        //		File file = new File("E:\\_Code_");//78719
        File file = new File("E:\\");//78719 78768
        long start = System.currentTimeMillis();
        int sum = getAllFile(file);
        long end = System.currentTimeMillis();
        System.out.println("总长度为：" + sum);
        System.out.println("耗时：" + (end - start));
    }

    public int getAllFile(File file) throws IOException, ExecutionException, InterruptedException {
        File[] files = file.listFiles();
        if (files == null) {
            return 0;
        }

        int sum = 0;

        for (int i = 0; i < files.length; i++) {
            if (files[i].isDirectory()) {

                sum += getAllFile(files[i]);
            } else {
                if (files[i].getAbsolutePath().endsWith(".java")) {
                    String path = files[i].getAbsolutePath();

                    //					sum += getLineCount(path);

                    GetLineCount count = new GetLineCount(path);
                    //					int lineCount = count.getLineCount();

                    FutureTask<Integer> futureTask = new FutureTask<>(count);
                    // 需要先run，然后再通过 get 获取结果
                    futureTask.run();
                    Integer lineCount = futureTask.get();

                    sum += lineCount;

                    System.out.println(path + "\t文件行数为：" + lineCount);
                }
            }
        }
        return sum;
    }

    public int getLineCount(String path) throws IOException {
        BufferedReader reader = new BufferedReader(new FileReader(path));
        int sum = 0;
        while (reader.readLine() != null) {
            sum += 1;
        }
        return sum;
    }
}


class GetLineCount implements Callable<Integer> {

    private String path;

    private Integer count = 0;

    public GetLineCount(String path) {
        this.path = path;
    }

    public int getLineCount() throws IOException {
        BufferedReader reader = new BufferedReader(new FileReader(path));
        while (reader.readLine() != null) {
            count += 1;
        }
        return count;
    }

    public Integer getCount() {
        return count;
    }

    @Override
    public Integer call() throws Exception {
        BufferedReader reader = new BufferedReader(new FileReader(path));
        while (reader.readLine() != null) {
            count += 1;
        }
        return count;
    }
}
```



## IO概述

### 流的分类

#### 根据数据的流向分为：输入流和输出流。

-   输入流 ：把数据从 其他设备 上读取到 内存 中的流。
    -   以InputStream,Reader结尾
-   输出流 ：把数据从 内存 中写出到 其他设备 上的流。
    -   以OutputStream、Writer结尾

#### 根据数据的类型分为：字节流和字符流。

-   字节流 ：以字节为单位，读写数据的流。
    -   以InputStream和OutputStream结尾
-   字符流 ：以字符为单位，读写数据的流。
    -   以Reader和Writer结尾

#### 根据IO流的角色不同分为：节点流和处理流（包装流）。

-   节点流：可以从或向一个特定的地方（节点）读写数据。如FileReader.
-   处理流：是对一个已存在的流进行连接和封装，通过所封装的流的功能调用实现数据读写。如BufferedReader.处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。

>   这种设计是装饰模式（Decorator Pattern）也称为包装模式（Wrapper Pattern），其使用一种对客户端透明的方式来动态地扩展对象的功能，它是通过继承扩展功能的替代方案之一。在现实生活中你也有很多装饰者的例子，例如：人需要各种各样的衣着，不管你穿着怎样，但是，对于你个人本质来说是不变的，充其量只是在外面加上了一些装饰，有，“遮羞的”、“保暖的”、“好看的”、“防雨的”....

**常用的节点流：**

-   文 件 FileInputStream FileOutputStrean FileReader FileWriter 文件进行处理的节点流。
-   字符串 StringReader StringWriter 对字符串进行处理的节点流。
-   数 组 ByteArrayInputStream ByteArrayOutputStream CharArrayReader CharArrayWriter 对数
-   组进行处理的节点流（对应的不再是文件，而是内存中的一个数组）。
-   管 道 PipedInputStream、PipedOutputStream、PipedReader、PipedWriter对管道进行处理的节点流。

**常用处理流：**

-   缓冲流：BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter---增加缓冲功能，避免频繁读写硬盘。
-   转换流：InputStreamReader、OutputStreamReader---实现字节流和字符流之间的转换。
-   数据流：DataInputStream、DataOutputStream -提供读写Java基础数据类型功能
-   对象流：ObjectInputStream、ObjectOutputStream--提供直接读写Java对象功能

### IO流的基类
| |输入流| 输出流|
|----| ------ | -------- |
|字节流 |字节输入流InputStream |字节输出流OutputStream|
|字符流 |字符输入流Reader |字符输出流Writer|
