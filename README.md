# JDBC基础
## **JDBC（Java DataBase Connectivity)**
### 基本概念：
 JDBC本质：其实是官方（sun公司）定义的一套操作所有关系型数据库的规则，即接口。各个数据库厂商去实现这套接口，提供数据库驱动jar包。我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动jar包中的实现类。  

### 基本操作方式：
 1. 到Oracle官网搜索数据库Mysql,找到Mysql Community,点进去Connect/java下载。  
 2. 项目中src下建一个libs，把下载的JDBC的jar包复制进去。  
 3. 右键libs，点Add As Library,新建javaClass,然后开始写代码。  

```java
//注册驱动
Class.forName("com.mysql.cj.jdbc.Driver");
//获取数据库连接对象
Connection conn = DriverManager.getConnection("jdbc:mysql:///数据库名?useUnicode=true & characterEncoding=UTF-8 & userSSL=false & serverTimezone=GMT%2B8", "数据库用户名", "密码");
//使用数据库SQL语句进行操作
String sql = "update 列名 set 表名 = 500 where id = 1";
//获取执行上面这句话的对象(Statement类)
Statement stmt = conn.createStatement();
//执行这句sql语句
int count = stmt.executeUpdate(sql);
//执行结果的输出
System.out.println(count);
//释放资源
stmt.close();
conn.close();
```

### **JDBC各个包详解**  
***DriverManager：驱动管理对象***  
* 注册驱动：告诉程序该使用哪一个数据库驱动jar  
 `static void registerDriver(Driver driver)` :注册与给定的驱动程序 DriverManager。
```java
static {
try {
java.sql.DriverManager.registerDriver(new Driver());
} catch (SQLException E) {
throw new RuntimeException("Can't register driver!");
   }
}
```
可以省略注册驱动步骤  

* 获取数据库连接  
`Connection conn = DriverManager.getConnection("jdbc:mysql:///数据库名?useUnicode=true & characterEncoding=UTF-8 & userSSL=false & serverTimezone=GMT%2B8", "数据库用户名", "密码");`  

如果连接的是本机mysql服务器，并且mysql服务默认端口是3306，则url可以简写为：`jdbc:mysql:///数据库名称?useUnicode=true & characterEncoding=UTF-8 & userSSL=false & serverTimezone=GMT%2B8  `1

***Connection：数据库连接对象***  
1. 获取执行sql的对象  
`Statement createStatement()`  
`PreparedStatement prepareStatement(String sql) `  

2. 管理事务  
* 开启事务：`setAutoCommit(boolean autoCommit)` ：调用该方法设置参数为false，即开启事务<br>
* 提交事务：`commit()`  
* 回滚事务：`rollback() `  

***Statement：执行sql的对象***
1. 执行sql  
* `boolean execute(String sql)` ：可以执行任意的sql  
* `int executeUpdate(String sql)` ：执行DML（insert、update、delete）语句、DDL(create，alter、drop)语句返回值：影响的行数，可以通过这个影响的行数判断DML语句是否执行成功 返回值>0的则执行成功，反之，则失败。  
* `ResultSet executeQuery(String sql) ` ：执行DQL（select)语句  

 ***ResultSet：结果集对象,封装查询结果***  
* `boolean next(): `游标向下移动一行，判断当前行是否是最后一行末尾(是否有数据)，如果是，则返回false，如果不是则返回true  
* `getXxx(参数)`:获取数据  
* Xxx：代表数据类型   如： `int getInt()` ,	`String getString()`  

*参数:*  
1. int：代表列的编号,从1开始   如： `getString(1)`  
2. String：代表列名称。 如： `getDouble("balance")`  
			
*注意:*  
使用步骤：  
1. 游标向下移动一行  
2. 判断是否有数据  
3. 获取数据  

```java
//循环判断游标是否是最后一行末尾。
while(rs.next()){
//获取数据
//6.2 获取数据
  int id = rs.getInt(1);
  String name = rs.getString("name");
  double balance = rs.getDouble(3);
  System.out.println(id + "---" + name + "---" + balance);
}
```  

***PreparedStatement：执行sql的对象***  
*SQL注入问题：在拼接sql时，有一些sql的特殊关键字参与字符串的拼接。会造成安全性问题*  
**例如**  
* *a' or 'a' = 'a  
* *sql：select * from user where username = 'fhdsjkf' and password = 'a' or 'a' = 'a'   

解决sql注入问题：使用PreparedStatement对象来解决  
select * from user where username = ? and password = ?;  

给？赋值：
* 方法： setXxx(参数1,参数2)
* 参数1：？的位置编号 从1 开始
* 参数2：？的值

预编译的SQL：参数使用?作为占位符  
```java
//2.定义sql
//2.1 张三 - 500
String sql1 = "update account set balance = balance - ? where id = ?";
//2.2 李四 + 500
String sql2 = "update account set balance = balance + ? where id = ?";
//3.获取执行sql对象
pstmt1 = conn.prepareStatement(sql1);
pstmt2 = conn.prepareStatement(sql2);
//4. 设置参数
pstmt1.setDouble(1,500);
pstmt1.setInt(2,1);
pstmt2.setDouble(1,500);
pstmt2.setInt(2,2);
//5.执行sql
pstmt1.executeUpdate();
```
      

注意：后期都会使用PreparedStatement来完成增删改查的所有操作  
1. 可以防止SQL注入  
2. 效率更高  

 
