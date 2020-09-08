# transfer 的使用

### transfer的作用

transfer的作用为将一个通道的数据直接映射到另一个通道

### API说明

-   transferFrom(输入源,0,size())：从输入源传输数据到调用方

-   transferTo(0,size(),输出源)：从调用方传输数据到输出源



```java
//1.获取通道
FileChannel inChannel = FileChannel.open(Paths.get("1.jpg"), StandardOpenOption.READ);
FileChannel outChannel = FileChannel.open(Paths.get("2.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);

//2.二选一
inChannel.transferTo(0, inChannel.size(), outChannel);
outChannel.transferFrom(inChannel,0,inChannel.size());

inChannel.close();
outChannel.close();
```