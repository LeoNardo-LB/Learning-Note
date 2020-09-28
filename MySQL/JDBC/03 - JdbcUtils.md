# JdbcUtils

##### JdbcUtils 封装了连接数据库及关闭数据库通道的操作

```java
package utils;

import java.io.FileInputStream;
import java.io.IOException;
import java.sql.*;
import java.util.Properties;


/**
 * @author: LeoNardo
 * @date: 2020/9/25 - 15:35
 */
public class JdbcUtils {
	
	private static Properties prop = new Properties();
	
	private static String username;
	
	private static String password;
	
	private static String url;
	
	private static String driverClass;
	
	// 在静态代码块中完成配置的读取
	static {
		
		try {
			prop.load(JdbcUtils.class.getClassLoader().getResourceAsStream("JdbcConnection.properties"));
			username = prop.getProperty("username");
			password = prop.getProperty("password");
			url = prop.getProperty("url");
			driverClass = prop.getProperty("driverClass");
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		Connection connection = getConnection();
		System.out.println("connection = " + connection);
	}
	
	/**
	 * 获取连接
	 * @return
	 */
	public static Connection getConnection() {
		Connection connection = null;
		try {
			
			Class.forName(driverClass);
			
			connection = DriverManager.getConnection(url, username, password);
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException throwables) {
			throwables.printStackTrace();
		}
		return connection;
	}
	
	/**
	 * 关闭资源
	 * 1. 连接
	 * 2. 命令发送器
	 * 3. 结果集
	 */
	public static void closeResource(Connection conn, PreparedStatement ps, ResultSet rs) {
		if (conn != null) {
			try {
				conn.setAutoCommit(true);
				conn.close();
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
		}
		
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
		}
		
		if (rs != null) {
			try {
				rs.close();
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
		}
	}
	
	public static void closeResource(Connection conn, PreparedStatement ps) {
		closeResource(conn, ps, null);
	}
}

```

#### 使用Druid数据库连接池的JdbcUtils

```java
package utils;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.IOException;
import java.sql.*;
import java.util.Properties;
import java.util.Random;


/**
 * @author: LeoNardo
 * @date: 2020/9/25 - 15:35
 */
public class PooledJdbcUtils {
	
	private static Properties prop = null;
	
	private static DataSource source = null;
	
	// 在静态代码块中完成连接池的配置
	static {
		
		try {
			prop = new Properties();
			
			prop.load(PooledJdbcUtils.class.getClassLoader().getResourceAsStream("druidConnection.properties"));
			
			source = DruidDataSourceFactory.createDataSource(prop);
		} catch (IOException e) {
			e.printStackTrace();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	// 测试数据库连接
	public static void main(String[] args) throws InterruptedException {
		
		for (int i = 0; i < 200; i++) {
			new Thread(() -> {
				
				Connection connection = getConnection();
				System.out.println(Thread.currentThread().getName() + ": " + connection);
				try {
					Thread.sleep(700 + new Random().nextInt(1000));
					connection.close();
				} catch (Exception e) {
					System.err.println(e.getMessage());
				}
			}).start();
			
			Thread.sleep(100);
		}
	}
	
	/**
	 * 获取连接
	 * @return
	 */
	public static Connection getConnection() {
		Connection connection = null;
		try {
			
			connection = source.getConnection();
		} catch (SQLException throwables) {
			System.err.println(throwables.getMessage());
		}
		return connection;
	}
	
	/**
	 * 关闭资源
	 * 1. 连接
	 * 2. 命令发送器
	 * 3. 结果集
	 */
	public static void closeResource(Connection conn, PreparedStatement ps, ResultSet rs) {
		if (conn != null) {
			try {
				conn.setAutoCommit(true);
				conn.close();
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
		}
		
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
		}
		
		if (rs != null) {
			try {
				rs.close();
			} catch (SQLException throwables) {
				throwables.printStackTrace();
			}
		}
	}
	
	public static void closeResource(Connection conn, PreparedStatement ps) {
		closeResource(conn, ps, null);
	}
}

```

