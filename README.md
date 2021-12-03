# JavaWeb-study
Study notes
## 2021/12/3
### 开始用github写笔记

##黑马Web(cover 黑马程序员笔记)
## **JDBC（Java DataBase Connectivity)**
### 基本概念：
> JDBC本质：其实是官方（sun公司）定义的一套操作所有关系型数据库的规则，即接口。各个数据库厂商去实现这套接口，提供数据库驱动jar包。我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动jar包中的实现类。

### 基本操作方式：
> 1.到Oracle官网搜索数据库Mysql,找到Mysql Community,点进去Connect/java下载。<br>
> 2.项目中src下建一个libs，把下载的JDBC的jar包复制进去。<br>
> 3.右键libs，点Add As Library,新建javaClass,然后开始写代码。<br>
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
### JDBC各个包详解
>DriverManager：驱动管理对象<br>
> 1.注册驱动：告诉程序该使用哪一个数据库驱动jar <br>
> `static void registerDriver(Driver driver)` :注册与给定的驱动程序 DriverManager。
```static {
try {
					            java.sql.DriverManager.registerDriver(new Driver());
					        } catch (SQLException E) {
					            throw new RuntimeException("Can't register driver!");
					        }
    					}
```
>可以省略注册驱动步骤

>2.获取数据库连接<br>
`Connection conn = DriverManager.getConnection("jdbc:mysql:///数据库名?useUnicode=true & characterEncoding=UTF-8 & userSSL=false & serverTimezone=GMT%2B8", "数据库用户名", "密码");`<br>
>如果连接的是本机mysql服务器，并且mysql服务默认端口是3306，则url可以简写为：jdbc:mysql:///数据库名称?useUnicode=true & characterEncoding=UTF-8 & userSSL=false & serverTimezone=GMT%2B8<br>



>Connection：数据库连接对象<br>
>1.获取执行sql的对象<br>
>`Statement createStatement()`<br>
>`PreparedStatement prepareStatement(String sql) `<br>

>2.管理事务<br>
>开启事务：`setAutoCommit(boolean autoCommit)` ：调用该方法设置参数为false，即开启事务<br>
>提交事务：`commit()`<br>
>回滚事务：`rollback() `<br>




>Statement：执行sql的对象<br>
>1. 执行sql<br>
>`boolean execute(String sql)` ：可以执行任意的sql 了解 <br>
> `int executeUpdate(String sql)` ：执行DML（insert、update、delete）语句、DDL(create，alter、drop)语句返回值：影响的行数，可以通过这个影响的行数判断DML语句是否执行成功 返回值>0的则执行<br>成功，反之，则失败。<br>
3. ResultSet executeQuery(String sql)  ：执行DQL（select)语句<br>

> 例子：<br>
> account表 添加一条记录<br>
> account表 修改记录<br>
> account表 删除一条记录<br>
```java
Statement stmt = null;
Connection conn = null;
try {
  //1. 注册驱动
  Class.forName("com.mysql.jdbc.Driver");
  //2. 定义sql
  String sql = "insert into account values(null,'王五',3000)";
  //3.获取Connection对象
  conn = DriverManager.getConnection("jdbc:mysql:///db3", "root", "root");
  //4.获取执行sql的对象 Statement
  stmt = conn.createStatement();
  //5.执行sql
  int count = stmt.executeUpdate(sql);//影响的行数
  //6.处理结果
  System.out.println(count);
  if(count > 0){
  System.out.println("添加成功！");
  }else{
  System.out.println("添加失败！");
 }
			
 } catch (ClassNotFoundException e) {
  e.printStackTrace();
 } catch (SQLException e) {
  e.printStackTrace();
 }finally {
 //stmt.close();
 //7. 释放资源
 //避免空指针异常
 if(stmt != null){
 try {
    stmt.close();
 } catch (SQLException e) {
  e.printStackTrace();
 }
}
			
if(conn != null){
try {
  conn.close();
} catch (SQLException e) {
   e.printStackTrace();
   }
 }
}
 ```
 
 
 
> ResultSet：结果集对象,封装查询结果<br>
> `boolean next(): `游标向下移动一行，判断当前行是否是最后一行末尾(是否有数据)，如果是，则返回false，如果不是则返回true<br>
>`getXxx(参数)`:获取数据<br>
>Xxx：代表数据类型   如： `int getInt()` ,	`String getString()`<br>
>参数：<br>
>1. int：代表列的编号,从1开始   如： `getString(1)`<br>
>2. String：代表列名称。 如： `getDouble("balance")`<br>
			
>注意：<br>
> *使用步骤：<br>
>1. 游标向下移动一行<br>
>2. 判断是否有数据<br>
>3. 获取数据<br>

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


						
>PreparedStatement：执行sql的对象<br>
>SQL注入问题：在拼接sql时，有一些sql的特殊关键字参与字符串的拼接。会造成安全性问题<br>
>1.输入用户随便，输入密码：a' or 'a' = 'a<br>
>2. sql：select * from user where username = 'fhdsjkf' and password = 'a' or 'a' = 'a' <br>
>*解决sql注入问题：使用PreparedStatement对象来解决<br>
>*预编译的SQL：参数使用?作为占位符<br>
      

>注意：后期都会使用PreparedStatement来完成增删改查的所有操作<br>
>1. 可以防止SQL注入<br>
>2. 效率更高<br>








 
