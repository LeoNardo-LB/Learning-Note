# PS 操作Blob(二进制)类型

#### PreparedStatement可以操作Blob类型的变量。

写入操作的方法：setBlob(InputStream is);

##### 读取操作的方法：

```java
Blob blob = getBlob(int index);
InputStream is = blob.getBinaryStream();
```

##### 插入insert

```java
//向数据表customers中插入Blob类型的字段
    @Test
    public void testInsert() throws Exception{
        Connection conn = JdbcUtils.getConnection();
        String sql = "insert into customers(name,email,birth,photo)values(?,?,?,?)";
        
        PreparedStatement ps = conn.prepareStatement(sql);
        
        ps.setObject(1,"袁浩");
        ps.setObject(2, "yuan@qq.com");
        ps.setObject(3,"1992-09-08");
        FileInputStream is = new FileInputStream(new File("girl.jpg"));
        ps.setBlob(4, is);
        
        ps.execute();
        
        JdbcUtils.closeResource(conn, ps);
        
}
```

##### 查询query

```java
//查询数据表customers中Blob类型的字段
@Test
public void testQuery(){
    Connection conn = null;
    PreparedStatement ps = null;
    InputStream is = null;
    FileOutputStream fos = null;
    ResultSet rs = null;
    try {
        conn = JdbcUtils.getConnection();
        String sql = "select id,name,email,birth,photo from customers where id = ?";
        ps = conn.prepareStatement(sql);
        ps.setInt(1, 21);
        rs = ps.executeQuery();
        if(rs.next()){
            //          方式一：
            //          int id = rs.getInt(1);
            //          String name = rs.getString(2);
            //          String email = rs.getString(3);
            //          Date birth = rs.getDate(4);
            //方式二：
            int id = rs.getInt("id");
            String name = rs.getString("name");
            String email = rs.getString("email");
            Date birth = rs.getDate("birth");

            Customer cust = new Customer(id, name, email, birth);
            System.out.println(cust);

            //将Blob类型的字段下载下来，以文件的方式保存在本地
            Blob photo = rs.getBlob("photo");
            is = photo.getBinaryStream();
            fos = new FileOutputStream("zhangyuhao.jpg");
            byte[] buffer = new byte[1024];
            int len;
            while((len = is.read(buffer)) != -1){
                fos.write(buffer, 0, len);
            }

        }
    } catch (Exception e) {
        e.printStackTrace();
    }finally{

        try {
            if(is != null)
                is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        try {
            if(fos != null)
                fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        JdbcUtils.closeResource(conn, ps, rs);
    }


}
```

