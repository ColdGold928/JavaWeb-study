# 抽取JDBC工具类：JDBCUtils
简化一般JDBC操作过程

* 使用配置文件传递连接参数
* 使JDBC工具类具有通用性，修改配置时不用再改java代码
  
 1. 配置文件
  ```properties
  jdbc.properties
  url=jdbc:mysql://ip地址(域名):端口号/数据库名称
  user=数据库用户名
  password=数据库密码
  ```

  2. 在连接类编写静态代码块，读取配置文件
  ```java
  static{
	        //读取资源文件，获取值。
	
	        try {
	            //1. 创建Properties集合类。
	            Properties pro = new Properties();
	
	            //获取src路径下的文件的方式--->ClassLoader 类加载器
	            ClassLoader classLoader = JDBCUtils.class.getClassLoader();
	            URL res  = classLoader.getResource("jdbc.properties");
	            String path = res.getPath();
	            System.out.println(path);//文件存储的绝对路径
	            //2. 加载文件
	           // pro.load(new FileReader("文件存储的绝对路径"));
	            pro.load(new FileReader(path));
	
	            //3. 获取数据，赋值
	            url = pro.getProperty("url");//通过标签名获取数据
	            user = pro.getProperty("user");
	            password = pro.getProperty("password");
	            driver = pro.getProperty("driver");
	            //4. 注册驱动
	            Class.forName(driver);
	        } catch (IOException e) {
	            e.printStackTrace();
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	        }
	    }

  ```

3.获取连接数据，并进行连接(静态方法)
```java
public static Connection getConnection() throws SQLException {
	
	        return DriverManager.getConnection(url, user, password);
	    }
```

4.最后进行资源释放（两种释放类型)  
* **第一种**
```java
public static void close(Statement stmt,Connection conn){
	        if( stmt != null){
	            try {
	                stmt.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	            }
	        }
	
	        if( conn != null){
	            try {
	                conn.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	            }
	        }
	    }
```

* **第二种**
```java
public static void close(ResultSet rs,Statement stmt, Connection conn){
	        if( rs != null){
	            try {
	                rs.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	            }
	        }
	
	        if( stmt != null){
	            try {
	                stmt.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	            }
	        }
	
	        if( conn != null){
	            try {
	                conn.close();
	            } catch (SQLException e) {
	                e.printStackTrace();
	            }
	        }
	    }
	
	}
```

