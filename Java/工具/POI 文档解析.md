# POI 文档解析

 开发中经常会设计到excel的处理，如导出Excel，导入Excel到数据库中，操作Excel目前有两个框架，一个是apache 的poi， 另一个是 Java Excel

 Apache POI 是用Java编写的免费开源的跨平台的 Java API，Apache POI提供API给Java程式对Microsoft Office（Excel、WORD、PowerPoint、Visio等）格式档案读和写的功能。

 官方主页： http://poi.apache.org/index.html
​ API文档： http://poi.apache.org/apidocs/index.html

maven坐标：

```xml
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi</artifactId>
  <version>3.14</version>
</dependency>

<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi-ooxml</artifactId>
  <version>3.14</version>
</dependency>
```



## Quick Start

读取一个Excel文件，步骤如下：

1.  创建操作excel的对象
2.  读取指定 sheet表
3.  遍历读取每个行
4.  遍历读取每一行的每一个单元格
5.  关闭资源

**代码示例**

```java
// 测试读取Excel
@Test
public void testRead() throws IOException {
    // 创建操作excel的对象
    XSSFWorkbook sheets = new XSSFWorkbook("C:\\Users\\Administrator\\Desktop\\testRead.xlsx");
    // 获取 sheet0 表
    XSSFSheet sheet0 = sheets.getSheetAt(0);

    // 从第一行遍历到最后一行，注意，需要取到最后一个行号
    for (int i = sheet0.getFirstRowNum(); i <= sheet0.getLastRowNum(); i++) {

        // 获取每一行中的列
        XSSFRow row = sheet0.getRow(i);
        for (int j = row.getFirstCellNum(); j < row.getLastCellNum(); j++) {
            // 获取每个单元格
            XSSFCell cell = row.getCell(j);
            // 取出单元格的String值，并打印之
            String value = cell.getStringCellValue();
            System.out.print(value + "\t\t");
        }
        // 换行
        System.out.println();
    }
    // 完成操作后释放资源
    sheets.close();
}
```

写入一个Excel文件，步骤如下；

1.  创建操作Excel的对象
2.  创建Sheet表
3.  创建每一行
4.  给每一行的单元格写入内容
5.  通过输出流保存到指定位置
6.  关闭资源

```java
// 测试写入Excel
@Test
public void testWrite() throws IOException, InvalidFormatException {

    // 创建操作excel的对象
    XSSFWorkbook sheets = new XSSFWorkbook();

    // 创建 sheet0 表
    XSSFSheet sheet0 = sheets.createSheet();

    // 根据行号创建行
    XSSFRow row0 = sheet0.createRow(0);
    XSSFRow row1 = sheet0.createRow(1);
    XSSFRow row2 = sheet0.createRow(2);

    // 给每一行赋值
    row0.createCell(0).setCellValue("人员");
    row0.createCell(1).setCellValue("配置");
    row0.createCell(2).setCellValue("状态");

    row1.createCell(0).setCellValue("Andy");
    row1.createCell(1).setCellValue("顶级装备");
    row1.createCell(2).setCellValue("受凉");

    row2.createCell(0).setCellValue("anrdo");
    row2.createCell(1).setCellValue("传说装备");
    row2.createCell(2).setCellValue("良好");

    sheets.write(new FileOutputStream("C:\\Users\\Administrator\\Desktop\\testWrite.xlsx"));

    // 完成操作后释放资源
    sheets.close();
}
```



## 通用的工具类

```java
public class POIUtils {
    private final static String xls = "xls";
    private final static String xlsx = "xlsx";
    private final static String DATE_FORMAT = "yyyy/MM/dd";
    /**
     * 读入excel文件，解析后返回
     * @param file
     * @throws IOException
     */
    public static List<String[]> readExcel(MultipartFile file) throws IOException {
        //检查文件
        checkFile(file);
        //获得Workbook工作薄对象
        Workbook workbook = getWorkBook(file);
        //创建返回对象，把每行中的值作为一个数组，所有行作为一个集合返回
        List<String[]> list = new ArrayList<String[]>();
        if(workbook != null){
            for(int sheetNum = 0;sheetNum < workbook.getNumberOfSheets();sheetNum++){
                //获得当前sheet工作表
                Sheet sheet = workbook.getSheetAt(sheetNum);
                if(sheet == null){
                    continue;
                }
                //获得当前sheet的开始行
                int firstRowNum  = sheet.getFirstRowNum();
                //获得当前sheet的结束行
                int lastRowNum = sheet.getLastRowNum();
                //循环除了第一行的所有行
                for(int rowNum = firstRowNum+1;rowNum <= lastRowNum;rowNum++){
                    //获得当前行
                    Row row = sheet.getRow(rowNum);
                    if(row == null){
                        continue;
                    }
                    //获得当前行的开始列
                    int firstCellNum = row.getFirstCellNum();
                    //获得当前行的列数
                    int lastCellNum = row.getPhysicalNumberOfCells();
                    String[] cells = new String[row.getPhysicalNumberOfCells()];
                    //循环当前行
                    for(int cellNum = firstCellNum; cellNum < lastCellNum;cellNum++){
                        Cell cell = row.getCell(cellNum);
                        cells[cellNum] = getCellValue(cell);
                    }
                    list.add(cells);
                }
            }
            workbook.close();
        }
        return list;
    }

    //校验文件是否合法
    public static void checkFile(MultipartFile file) throws IOException{
        //判断文件是否存在
        if(null == file){
            throw new FileNotFoundException("文件不存在！");
        }
        //获得文件名
        String fileName = file.getOriginalFilename();
        //判断文件是否是excel文件
        if(!fileName.endsWith(xls) && !fileName.endsWith(xlsx)){
            throw new IOException(fileName + "不是excel文件");
        }
    }
    public static Workbook getWorkBook(MultipartFile file) {
        //获得文件名
        String fileName = file.getOriginalFilename();
        //创建Workbook工作薄对象，表示整个excel
        Workbook workbook = null;
        try {
            //获取excel文件的io流
            InputStream is = file.getInputStream();
            //根据文件后缀名不同(xls和xlsx)获得不同的Workbook实现类对象
            if(fileName.endsWith(xls)){
                //2003
                workbook = new HSSFWorkbook(is);
            }else if(fileName.endsWith(xlsx)){
                //2007
                workbook = new XSSFWorkbook(is);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return workbook;
    }
    public static String getCellValue(Cell cell){
        String cellValue = "";
        if(cell == null){
            return cellValue;
        }
        //如果当前单元格内容为日期类型，需要特殊处理
        String dataFormatString = cell.getCellStyle().getDataFormatString();
        if(dataFormatString.equals("m/d/yy")){
            cellValue = new SimpleDateFormat(DATE_FORMAT).format(cell.getDateCellValue());
            return cellValue;
        }
        //把数字当成String来读，避免出现1读成1.0的情况
        if(cell.getCellType() == Cell.CELL_TYPE_NUMERIC){
            cell.setCellType(Cell.CELL_TYPE_STRING);
        }
        //判断数据的类型
        switch (cell.getCellType()){
            case Cell.CELL_TYPE_NUMERIC: //数字
                cellValue = String.valueOf(cell.getNumericCellValue());
                break;
            case Cell.CELL_TYPE_STRING: //字符串
                cellValue = String.valueOf(cell.getStringCellValue());
                break;
            case Cell.CELL_TYPE_BOOLEAN: //Boolean
                cellValue = String.valueOf(cell.getBooleanCellValue());
                break;
            case Cell.CELL_TYPE_FORMULA: //公式
                cellValue = String.valueOf(cell.getCellFormula());
                break;
            case Cell.CELL_TYPE_BLANK: //空值
                cellValue = "";
                break;
            case Cell.CELL_TYPE_ERROR: //故障
                cellValue = "非法字符";
                break;
            default:
                cellValue = "未知类型";
                break;
        }
        return cellValue;
    }
}
```

