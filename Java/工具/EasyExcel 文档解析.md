# EasyExcel 文档解析

## EasyExcel简介

github网址：https://github.com/alibaba/easyexcel

快速开始（非官方）：https://www.yuque.com/easyexcel/doc/easyexcel

EasyExcel特点：

-   Java领域解析、生成Excel比较有名的框架有Apache poi、jxl等。但他们都存在一个严重的问题就是非常的耗内存。如果你的系统并发量不大的话可能还行，但是一旦并发上来后一定会OOM或者JVM频繁的full gc。
-   EasyExcel是阿里巴巴开源的一个excel处理框架，**以使用简单、节省内存著称**。EasyExcel能大大减少占用内存的主要原因是在解析Excel时没有将文件数据一次性全部加载到内存中，而是从磁盘上一行行读取数据，逐个解析。
-   EasyExcel采用一行一行的解析模式，并将一行的解析结果以观察者的模式通知处理（AnalysisEventListener）。

相关依赖：

 ```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>2.1.7</version>
</dependency>
 ```



## 写入Excel

1、编写实体映射类：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Weapon {

    @ExcelProperty(value = "枪械名称",index = 0)
    private String name;

    @ExcelProperty(value = "价格",index = 4)
    private Integer price;

    @ExcelProperty(value = "伤害",index = 1)
    private Double damage;

    @ExcelProperty("重量")
    private Double weight;

    @ExcelProperty(value = "弹匣容量",index = 2)
    private Integer magazine;

    @ExcelProperty(value = "生产日期",index = 6)
    @DateTimeFormat("yyyy-MM-dd")   // 格式化日期
    private Date productDate;

    @ExcelIgnore    // 忽略该字段
    private Integer frequency;

}
```

2、准备写入的数据：

```java
private List<Weapon> data() {
    ArrayList<Weapon> weapons = new ArrayList<>();
    weapons.add(new Weapon("Ak47", 2700, 38.0, 3200.0, 30, new Date(), null));
    weapons.add(new Weapon("M4A1", 3100, 32.0, 2800.0, 30, new Date(), null));
    weapons.add(new Weapon("Deagle", 700, 55.0, 2000.0, 7, new Date(), null));
    weapons.add(new Weapon("AWP", 4750, 120.0, 3500.0, 10, new Date(), null));
    weapons.add(new Weapon("p250", 300, 41.0, 1800.0, 13, new Date(), null));
    weapons.add(new Weapon("m249", 5100, 30.0, 3800.0, 100, new Date(), null));
    return weapons;
}
```

3、执行写操作：

```java
@Test
public void testWrite() {
    EasyExcel.write("F:/test easyexcel/excel01.xlsx", Weapon.class)
            .sheet(0)
            .doWrite(data());
    System.out.println("写好了");
}
```



## 读取Excel

1、编写实体映射类：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Weapon {

    @ExcelProperty(value = "枪械名称",index = 0)
    private String name;

    @ExcelProperty(value = "价格",index = 4)
    private Integer price;

    @ExcelProperty(value = "伤害",index = 1)
    private Double damage;

    @ExcelProperty("重量")
    private Double weight;

    @ExcelProperty(value = "弹匣容量",index = 2)
    private Integer magazine;

    @ExcelProperty(value = "生产日期",index = 6)
    @DateTimeFormat("yyyy-MM-dd")   // 格式化日期
    private Date productDate;

    @ExcelIgnore    // 忽略该字段
    private Integer frequency;

}
```

2、编写读取监听器（实现 `AnalysisEventListener` 接口，用于监听读取的文件。

```java
/**
 * Author: Administrator
 * Create: 2020/12/22
 **/
public class WeaponExcelDataListener extends AnalysisEventListener<Weapon> {

    List<Weapon> list = new ArrayList<Weapon>();

    // 每读取一条记录都会调用
    @Override
    public void invoke(Weapon weapon, AnalysisContext analysisContext) {
        list.add(weapon);
    }

    // 读取完毕之后调用
    @Override
    public void doAfterAllAnalysed(AnalysisContext analysisContext) {
        System.out.println("数据读取完毕！");
        list.forEach(System.out::println);
    }

}
```

3、执行读取excel

```java
@Test
public void testRead(){
    EasyExcel.read("F:\\test easyexcel\\excel01.xlsx", Weapon.class,new WeaponExcelDataListener())
        .doReadAll();
}
```