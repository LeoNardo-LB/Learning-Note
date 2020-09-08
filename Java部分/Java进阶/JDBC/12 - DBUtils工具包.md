# DBUtils工具包

#### DBUtils的作用

DBUtils工具包封装了底层 增删改查的操作,简化代码开发,我们仅需要提供数据库连接即可

#### DBUtils工具包的使用

1.  导入jar包

![image.png](_images/1599124603296-8363da9b-e96c-40e2-87f3-134da88126c8.png)

2.  编写代码

    

## 编写代码

##### 使用现成的jar中的QueryRunner测试增、删、改的操作:

```java
//测试插入
@Test
public void testInsert() {
    Connection conn = null;
    try {
        QueryRunner runner = new QueryRunner();
        conn = JdbcUtils.getConnection3();
        String sql = "insert into customers(name,email,birth)values(?,?,?)";
        int insertCount = runner.update(conn, sql, "蔡徐坤","caixukun@126.com","1997-09-08");
        System.out.println("添加了" + insertCount + "条记录");
    } catch (SQLException e) {
        e.printStackTrace();
    }finally{
        JdbcUtils.closeResource(conn, null);
    }

}
```

##### 使用现成的jar中的QueryRunner测试查询的操作:

```java
//测试查询
/*
     * BeanHander:是ResultSetHandler接口的实现类，用于封装表中的一条记录。
     */
@Test
public void testQuery1(){
    Connection conn = null;
    try {
        QueryRunner runner = new QueryRunner();
        conn = JdbcUtils.getConnection3();
        String sql = "select id,name,email,birth from customers where id = ?";
        BeanHandler<Customer> handler = new BeanHandler<>(Customer.class);
        Customer customer = runner.query(conn, sql, handler, 23);
        System.out.println(customer);
    } catch (SQLException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }finally{
        JdbcUtils.closeResource(conn, null);

    }

}

/*
     * BeanListHandler:是ResultSetHandler接口的实现类，用于封装表中的多条记录构成的集合。
     */
@Test
public void testQuery2() {
    Connection conn = null;
    try {
        QueryRunner runner = new QueryRunner();
        conn = JdbcUtils.getConnection3();
        String sql = "select id,name,email,birth from customers where id < ?";

        BeanListHandler<Customer>  handler = new BeanListHandler<>(Customer.class);

        List<Customer> list = runner.query(conn, sql, handler, 23);
        list.forEach(System.out::println);
    } catch (SQLException e) {
        e.printStackTrace();
    }finally{

        JdbcUtils.closeResource(conn, null);
    }

}

/*
     * MapHander:是ResultSetHandler接口的实现类，对应表中的一条记录。
     * 将字段及相应字段的值作为map中的key和value
     */
@Test
public void testQuery3(){
    Connection conn = null;
    try {
        QueryRunner runner = new QueryRunner();
        conn = JdbcUtils.getConnection3();
        String sql = "select id,name,email,birth from customers where id = ?";
        MapHandler handler = new MapHandler();
        Map<String, Object> map = runner.query(conn, sql, handler, 23);
        System.out.println(map);
    } catch (SQLException e) {
        e.printStackTrace();
    }finally{
        JdbcUtils.closeResource(conn, null);

    }

}

/*
     * MapListHander:是ResultSetHandler接口的实现类，对应表中的多条记录。
     * 将字段及相应字段的值作为map中的key和value。将这些map添加到List中
     */
@Test
public void testQuery4(){
    Connection conn = null;
    try {
        QueryRunner runner = new QueryRunner();
        conn = JdbcUtils.getConnection3();
        String sql = "select id,name,email,birth from customers where id < ?";

        MapListHandler handler = new MapListHandler();
        List<Map<String, Object>> list = runner.query(conn, sql, handler, 23);
        list.forEach(System.out::println);
    } catch (SQLException e) {
        e.printStackTrace();
    }finally{
        JdbcUtils.closeResource(conn, null);

    }

}

/*
     * ScalarHandler: ScalarHandler 的参数为空或null时，返回第一行第一列的数据 
     */
@Test
public void testQuery6(){
    Connection conn = null;
    try {
        QueryRunner runner = new QueryRunner();
        conn = JdbcUtils.getConnection3();

        String sql = "select max(birth) from customers";

        ScalarHandler handler = new ScalarHandler();
        Date maxBirth = (Date) runner.query(conn, sql, handler);
        System.out.println(maxBirth);
    } catch (SQLException e) {
        e.printStackTrace();
    }finally{
        JdbcUtils.closeResource(conn, null);

    }

}

/*
     * 自定义ResultSetHandler的实现类
     */
@Test
public void testQuery7(){
    Connection conn = null;
    try {
        QueryRunner runner = new QueryRunner();
        conn = JdbcUtils.getConnection3();

        String sql = "select id,name,email,birth from customers where id = ?";
        ResultSetHandler<Customer> handler = new ResultSetHandler<Customer>(){

            @Override
            public Customer handle(ResultSet rs) throws SQLException {
                //      System.out.println("handle");
                //      return null;
                //      return new Customer(12, "成龙", "Jacky@126.com", new Date(234324234324L));

                if(rs.next()){
                    int id = rs.getInt("id");
                    String name = rs.getString("name");
                    String email = rs.getString("email");
                    Date birth = rs.getDate("birth");
                    Customer customer = new Customer(id, name, email, birth);
                    return customer;
                }
                return null;

            }

        };
        Customer customer = runner.query(conn, sql, handler,23);
        System.out.println(customer);
    } catch (SQLException e) {
        e.printStackTrace();
    }finally{
        JdbcUtils.closeResource(conn, null);
    }
}
```

##### 使用dbutils.jar包中的DbUtils工具类实现连接等资源的关闭：

```java
/**
     * 
     * @Description 使用dbutils.jar中提供的DbUtils工具类，实现资源的关闭
     * @author shkstart
     * @date 下午4:53:09
     * @param conn
     * @param ps
     * @param rs
     */
public static void closeResource1(Connection conn,Statement ps,ResultSet rs){
    //      try {
    //          DbUtils.close(conn);
    //      } catch (SQLException e) {
    //          e.printStackTrace();
    //      }
    //      try {
    //          DbUtils.close(ps);
    //      } catch (SQLException e) {
    //          e.printStackTrace();
    //      }
    //      try {
    //          DbUtils.close(rs);
    //      } catch (SQLException e) {
    //          e.printStackTrace();
    //      }

    DbUtils.closeQuietly(conn);
    DbUtils.closeQuietly(ps);
    DbUtils.closeQuietly(rs);
}
```



## 个人笔记:

使用QueryRunner(外部类)  前提:导入第三方类,格式和之前自己写的一致,

1.  建立连接和sql语句:JDBCUtils.getConnection(); String sql = "....";
2.  建立QueryRunner对象:QueryRunner runner = new QueryRunner();
3.  如果是查询,则需要创建ResultSetHandler实现类对象,用于指定runner的返回值类型,如果是增删改,则不需要(增删改默认返回值为int,表示影响行数)
4.  调用QueryRunner对象:

-   -   增删改:runner.update(Connection connection,String sql,Object ...args) 返回值为影响的行数(int)
    -   查询:runner.query(Connection connection,String sql,HandlerType handler,Object ...args) 返回值由ResultSetHandler决定

HandlerType(返回值类型):

BeanHandler<Customers> handler = new BeanHandler<>(Customers.class);   返回一条Customers的记录

BeanListHandler<Customers> handler = new BeanListHandler<>(Customers.class);  返回Customers类的集合

MapHandler mapHandler = new MapHandler();   以键值对储存在map中返回(返回Map<K,V>的对象)

ScalarHandler sh = new ScalarHandler();  读取结果集中的第一条记录,以Object的对象返回

##### 自定义返回结果:

```java
ResultSetHandler<T> resultSetHandler = new ResultSetHandler<T>() {
    @Override
    public T handle(ResultSet resultSet) throws SQLException {
        // 自定义操作 // 
        return null;
    }
}
```