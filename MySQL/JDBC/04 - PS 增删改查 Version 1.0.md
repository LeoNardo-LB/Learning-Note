# PS 增删改查 Version 1.0



查询时需要传入 Class类型以指定返回值类型的版本

#### PreparedStatement的理解：

① PreparedStatement 是Statement的子接口

② An object that represents a precompiled SQL statement. 

③ 可以解决Statement的sql注入问题，拼串问题

#### 两种技术

1.  使用结果集的元数据：ResultSetMetaData

>   getColumnCount():获取列数
>
>   getColumnLabel():获取列的别名
>
>   说明：如果sql中没给字段其别名，getColumnLabel()获取的就是列名

2.  反射的使用（①创建对应的运行时类的对象 ② 在运行时，动态的调用指定的运行时类的属性、方法）

##### 使用PreparedStatement实现通用的增、删、改的方法：version 1.0

```java
//通用的增删改操作
public void update(String sql,Object ...args){
    Connection conn = null;
    PreparedStatement ps = null;
    try {
        //1.获取数据库的连接
        conn = JdbcUtils.getConnection();
        //2.预编译sql语句，返回PreparedStatement的实例
        ps = conn.prepareStatement(sql);
        //3.填充占位符
        for(int i = 0;i < args.length;i++){
            ps.setObject(i + 1, args[i]);//小心参数声明错误！！
        }
        //4.执行
        ps.execute();
    } catch (Exception e) {
        e.printStackTrace();
    }finally{
        //5.资源的关闭
        JdbcUtils.closeResource(conn, ps);

    }
}
```

##### 使用PreparedStatement实现通用的查询操作 version 1.0

```java
// 针对于不同的表的通用的查询操作，返回表中的一条记录
public <T> T getInstance(Class<T> clazz,String sql, Object... args) {
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        conn = JdbcUtils.getConnection();

        ps = conn.prepareStatement(sql);
        for (int i = 0; i < args.length; i++) {
            ps.setObject(i + 1, args[i]);
        }

        rs = ps.executeQuery();
        // 获取结果集的元数据 :ResultSetMetaData
        ResultSetMetaData rsmd = rs.getMetaData();
        // 通过ResultSetMetaData获取结果集中的列数
        int columnCount = rsmd.getColumnCount();

        if (rs.next()) {
            T t = clazz.newInstance();
            // 处理结果集一行数据中的每一个列
            for (int i = 0; i < columnCount; i++) {
                // 获取列值
                Object columValue = rs.getObject(i + 1);

                // 获取每个列的列名
                // String columnName = rsmd.getColumnName(i + 1);
                String columnLabel = rsmd.getColumnLabel(i + 1);

                // 给t对象指定的columnName属性，赋值为columValue：通过反射
                Field field = clazz.getDeclaredField(columnLabel);
                field.setAccessible(true);
                field.set(t, columValue);
            }
            return t;
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        JdbcUtils.closeResource(conn, ps, rs);

    }

    return null;
}

// 返回多条记录
public <T> List<T> getForList(Class<T> clazz,String sql, Object... args){
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        conn = JdbcUtils.getConnection();

        ps = conn.prepareStatement(sql);
        for (int i = 0; i < args.length; i++) {
            ps.setObject(i + 1, args[i]);
        }

        rs = ps.executeQuery();
        // 获取结果集的元数据 :ResultSetMetaData
        ResultSetMetaData rsmd = rs.getMetaData();
        // 通过ResultSetMetaData获取结果集中的列数
        int columnCount = rsmd.getColumnCount();
        //创建集合对象
        ArrayList<T> list = new ArrayList<T>();
        while (rs.next()) {
            T t = clazz.newInstance();
            // 处理结果集一行数据中的每一个列:给t对象指定的属性赋值
            for (int i = 0; i < columnCount; i++) {
                // 获取列值
                Object columValue = rs.getObject(i + 1);

                // 获取每个列的列名
                String columnLabel = rsmd.getColumnLabel(i + 1);

                // 给t对象指定的columnName属性，赋值为columValue：通过反射
                Field field = clazz.getDeclaredField(columnLabel);
                field.setAccessible(true);
                field.set(t, columValue);
            }
            list.add(t);
        }

        return list;
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        JdbcUtils.closeResource(conn, ps, rs);

    }

    return null;
}
```

##### 查询特殊值

```java
    //用于查询特殊值的通用的方法
    public <E> E getValue(String sql,Object...args){
        Connection conn = JdbcUtils.getConnection();
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(sql);
            for(int i = 0;i < args.length;i++){
                ps.setObject(i + 1, args[i]);
                
            }
            
            rs = ps.executeQuery();
            if(rs.next()){
                return (E) rs.getObject(1);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally{
            JdbcUtils.closeResource(null, ps, rs);
            
        }
        return null;
}
```

##### 获取自增长键值

```java
/*
 * 我们通过JDBC往数据库的表格中添加一条记录，其中有一个字段是自增的，那么在JDBC这边怎么在添加之后直接获取到这个自增的值
 * PreparedStatement是Statement的子接口。
 * Statement接口中有一些常量值：
 * （1）Statement.RETURN_GENERATED_KEYS
 * 
 * 要先添加后获取到自增的key值：
 * （1）PreparedStatement pst = conn.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS);
 * （2）添加sql执行完成后,通过PreparedStatement的对象调用getGeneratedKeys()方法来获取自增长键值，遍历结果集
 * 		ResultSet rs = pst.getGeneratedKeys();
 */
public class TestAutoIncrement {
    public static void main(String[] args) throws Exception{
        //1、注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        //2、获取连接
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "123456");

        //3、执行sql
        String sql = "insert into t_department values(null,?,?)";
        /*
		 * 这里在创建PreparedStatement对象时，传入第二个参数的作用，就是告知服务器端
		 * 当执行完sql后，把自增的key值返回来。
		 */
        PreparedStatement pst = conn.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS);

        //设置？的值
        pst.setObject(1, "测试部");
        pst.setObject(2, "测试项目数据");

        //执行sql
        int len = pst.executeUpdate();//返回影响的记录数
        if(len>0){
            //从pst中获取到服务器端返回的键值
            ResultSet rs = pst.getGeneratedKeys();
            //因为这里的key值可能多个，因为insert语句可以同时添加多行，所以用ResultSet封装
            //这里因为只添加一条，所以用if判断
            if(rs.next()){
                Object key = rs.getObject(1);
                System.out.println("自增的key值did =" + key);
            }
        }

        //4、关闭
        pst.close();
        conn.close();
    }
}
```

## 