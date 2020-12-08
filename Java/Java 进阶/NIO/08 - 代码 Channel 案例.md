# 代码: Channel 案例

##### 利用通道完成文件复制

```java
public void test1() {
    FileInputStream input = null;
    FileOutputStream output = null;
    FileChannel inChannel = null;
    FileChannel outChannel = null;

    try {
        //1.建立搬运流
        input = new FileInputStream("1.jpg");
        output = new FileOutputStream("2.jpg");

        //2.建立通道
        inChannel = input.getChannel();
        outChannel = output.getChannel();

        //3.建立缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);

        //4.读取数据 写入缓冲区
        while (inChannel.read(buf) != -1) {
            buf.flip(); //切换读取模式

            // 5.将缓冲区数据写入通道
            outChannel.write(buf);

            // 6.清空缓冲区
            buf.clear();
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (inChannel != null) {
            try {
                inChannel.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (outChannel != null) {
            try {
                outChannel.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (output != null) {
            try {
                output.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (input != null) {
            try {
                input.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

##### 内存映射(使用直接缓冲区完成文件的复制),只有ByteBuffer支持

```java
public void test2() throws IOException {
    //1.获取通道(定义通道和赋予权限:读取 写入 创建等)
    FileChannel inChannel = FileChannel.open(Paths.get("1.jpg"), StandardOpenOption.READ); //第一个数据是给一个path类的路径   后面的参数是给定的权限
    FileChannel outChannel = FileChannel.open(Paths.get("2.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE, StandardOpenOption.READ);

    //2.内存映射文件声明与定义(在物理内存中)
    MappedByteBuffer inMappedBuffer = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
    MappedByteBuffer outMappedBuffer = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());

    //3.直接对缓冲区进行数据的读写操作
    byte[] temp = new byte[inMappedBuffer.limit()];
    inMappedBuffer.get(temp);
    outMappedBuffer.put(temp);

    inChannel.close();
    outChannel.close();
}
```

##### 通道之间的数据传输

```java
public void test3() throws IOException {
    /*
            transferFrom(输入源,0,size())
            transferTo(0,size(),输出源)
         */

    //1.获取通道
    FileChannel inChannel = FileChannel.open(Paths.get("1.jpg"), StandardOpenOption.READ);
    FileChannel outChannel = FileChannel.open(Paths.get("2.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);

    //2.二选一
    inChannel.transferTo(0, inChannel.size(), outChannel);
    outChannel.transferFrom(inChannel,0,inChannel.size());

    inChannel.close();
    outChannel.close();
}
```

##### 分散和聚集

```java
public void test4() throws IOException {

    //1.获取通道
    FileChannel open1 = FileChannel.open(Paths.get("1.txt"), StandardOpenOption.READ);
    FileChannel open2 = FileChannel.open(Paths.get("2.txt"), StandardOpenOption.READ, StandardOpenOption.CREATE, StandardOpenOption.WRITE);

    //2.获取缓冲区
    ByteBuffer allocate1 = ByteBuffer.allocate(128);
    ByteBuffer allocate2 = ByteBuffer.allocate(512);
    ByteBuffer allocate3 = ByteBuffer.allocate(1024);
    ByteBuffer[] arr = new ByteBuffer[]{allocate1, allocate2, allocate3};

    //3.分散读取
    open1.read(arr);
    for (ByteBuffer b : arr) {
        b.flip();
    }
    System.out.println(new String(arr[0].array(), 0, arr[0].limit()));
    System.out.println(new String(arr[1].array(), 0, arr[1].limit()));
    System.out.println(new String(arr[2].array(), 0, arr[2].limit()));
    System.out.println();

    //4.聚集写入
    long write = open2.write(arr);
    System.out.println(write);

}
```

##### 字符集Chaeset

```java
public void test5() throws CharacterCodingException {

    Charset gbk = Charset.forName("GBK");

    // 获取编码器
    CharsetEncoder ce = gbk.newEncoder();
    // 获取解码器
    CharsetDecoder cd = gbk.newDecoder();

    // 编码和解码就是CharBuffer 和 ByteBuffer 之间的转换
    // 获取字符缓存的过程(可有多种方式)
    CharBuffer charBuffer = CharBuffer.allocate(1024);
    charBuffer.put("香港属于中国");
    charBuffer.flip();

    //编码
    ByteBuffer byteBuffer = ce.encode(charBuffer);  //字符流根据编码器的指示: CharBuffer->ByteBuffer

    //解码
    CharBuffer decode = cd.decode(byteBuffer);
    System.out.println(decode);
    System.out.println("-------------------------------------------------");

    //如果解码格式不一致,有可能出现乱码
    Charset cs2 = Charset.forName("UTF-8");
    byteBuffer.rewind();
    CharBuffer decode1 = cs2.decode(byteBuffer);
    System.out.println(decode1.toString());
}
```