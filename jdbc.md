## JDBC
   ### 1.什么是JDBC？
    JDBC（Java DataBase Connectivity）是Java和数据库之间的一个桥梁，是一个规范,一种接口。而不是一个实现，能够执行SQL语句。它由一组用Java语言编写的类和接口组成。具体的实现由不同的生产产商决定。
    JDBC驱动程序一共有四种类型：
    类型1-JDBC-ODBC桥

    类型2-本地API驱动

    类型3-网络协议驱动

    类型4-本地协议驱动
    我们常用的是类型4-本地协议驱动，这种类型的驱动使用socket链接，直接在客户端和数据库之间进行通信。
    优点是1-访问速度快 2-最直接，最纯粹的JAVA实现。
    缺点是1-需要每个数据库厂商提供自己的JDBC驱动。2-需要针对不同的数据库使用不同的驱动程序。

  ### 2.JDBC的实现步骤
* 加载驱动
```
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
      Class.forName把驱动类com.mysql.jdbc.Driver加载到JVM中，完成驱动的初始化的相关工作。

```
* 建立JDBC和数据库之间的Connection连接
```
  Connection c = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/exam?characterEncoding=UTF-8", "root", "admin");
```
* 创建Statement或者PreparedStatement接口，执行SQL语句
```
  在Statement中使用字符串拼接的方式，该方式存在句法复杂，容易犯错等缺点，具体在下文中的对比中介绍。所以Statement在实际过程中使用的非常的少。
  使用PreparedStatement接口
  与 Statement一样，PreparedStatement也是用来执行sql语句的与创建Statement不同的是，需要根据sql语句创建PreparedStatement。除此之外，还能够通过设置参数，指定相应的值，而不是Statement那样使用字符串拼接。
  String sql = "insert into t_course(course_name) values(?)"; 
  给占位符赋值：preparedStatement.setString(1, courseName); 

```
* 执行sql
```
  preparedStatement.executeUpdate();	
```
* 关闭连接

### 3.Preparedstatement与Statement的区别
     PreparedStatement执行的效率比高，PreparedStatement能对SQL进行预编译，预编译的SQL存储在PreparedStatement对象中，因为使用了占位符，所以每次的Sql就是一样的，数据库就不会再次编译，这样能够显著提高性能。如果基本数据库和驱动程序在语句提交之后仍保持这些语句的打开状态，则同一个 PreparedStatement 可执行多次。如果这一点不成立，那么试图通过使用PreparedStatement 对象代替 Statement 对象来提高性能是没有意义的。
     PreparedStatement其能够有效防止SQL注入攻击。其使用参数设置，可读性好，不易记错。在statement中使用字符串拼接，可读性和维护性比较差。
     Statement对象编译SQL语句时，如果SQL语句有变量，就需要使用分隔符来隔开，如果变量非常多，就会使SQL变得非常复杂。PreparedStatement可以使用占位符，简化sql的编写。

### 4. 可滚动和可更新
    ResultSet表示数据库结果集的数据表，通常通过执行查询数据库的语句生成。默认的 ResultSet 对象不可更新，仅有一个向前移动的光标。因此，只能迭代它一次，并且只能按从第一行到最后一行的顺序进行。
    但是可以通过设置ResultSet的属性来生成可滚动和可更新的结果集。可滚动简单说就是设置结果集可更新resultSet目前的游标值。可更新就是可以更新结果集里面的增删改查。可更新简单说，就是获取数据集ResultSet以后改动更加灵活。
    可滚动示例：
```
import java.sql.*;
 
public class TestScroll {
	public static void main(String args[]) {
 
		try {
			new oracle.jdbc.driver.OracleDriver();
			String url = "jdbc:oracle:thin:@127.0.0.1:1521:orcl";
			Connection conn = DriverManager
					.getConnection(url, "scott", "tiger");
			Statement stmt = conn.createStatement(
					ResultSet.TYPE_SCROLL_INSENSITIVE,
					ResultSet.CONCUR_READ_ONLY);  //设置结果集的状态
			ResultSet rs = stmt
					.executeQuery("select * from emp order by sal");
			rs.next();//将光标前移一行
			System.out.println(rs.getInt(1));
			rs.last(); //光标移到最后一行
			System.out.println(rs.getString(1));
			System.out.println(rs.isLast());
			System.out.println(rs.isAfterLast());
			System.out.println(rs.getRow());
			rs.previous(); //将光标移到该resultset对象的上一行
			System.out.println(rs.getString(1));
			rs.absolute(6); //将光标移动到该编号的resultSet对象
			System.out.println(rs.getString(1));
			rs.close();
			stmt.close();
			conn.close();
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}
```
可更新示例：
```
import java.sql.*;
public class TestUpdataRs {
    public static void main(String args[]){
	
	try{
	    new oracle.jdbc.driver.OracleDriver();
	    String url="jdbc:oracle:thin:@192.168.0.1:1521:SXT";
	    Connection conn=DriverManager.getConnection(url,"scott","tiger");
	    Statement stmt=conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE,ResultSet.CONCUR_UPDATABLE);
	    
	    ResultSet rs=stmt.executeQuery("select * from emp2");
	    
	    rs.next();
	    //更新一行数据
	    rs.updateString("ename","AAAA");
	    rs.updateRow();
 
	    //插入新行
	    rs.moveToInsertRow();  //将光标移动到插入行
	    rs.updateInt(1, 9999);
	    rs.updateString("ename","AAAA");
	    rs.updateInt("mgr", 7839);
	    rs.updateDouble("sal", 99.99);
	    rs.insertRow(); //将插入行的内容插入到此resultset对象和数据库中
	    //将光标移动到新建的行
	    rs.moveToCurrentRow();
 
	    //删除行（光标不位于插入行上时，不能调用此方法）
	    rs.absolute(5);
	    rs.deleteRow();
 
	    //取消更新
	    //rs.cancelRowUpdates();
 
	  }catch(SQLException e){
	    e.printStackTrace();
	  }
    }
}
```
    滚动特性：
    next()，此方法是使游标向下一条记录移动。

　　previous() ，此方法可以使游标上一条记录移动，前提前面还有记录。

　　absolute(int row)，可以使用此方法跳到指定的记录位置。定位成功返回true，不成功返回false，返回值为false，则游标不会移动。

　　afterLast() ，游标跳到最后一条记录之后。

　　beforeFirst() ，游标跳到第一条记录之前。（跳到游标初始位）

　　first()，游标指向第一条记录。

　　last()，游标指向最后一条记录。

　　relative(int rows) ，相对定位方法，参数值可正可负，参数为正，游标从当前位置向下移动指定值，参数为负，游标从当前位置向上移动指定值。

    
### 5.处理处理大文本和二进制数据
   使用clob用于存储大文本，使用blob用于存储二进制数据.
   MySQL存储大文本是用Test【代替clob】，Test又分为4类;
    TINYTEXT
    TEXT
    MEDIUMTEXT
    LONGTEXT
  ```
  /*
*用JDBC操作MySQL数据库去操作大文本数据
*
*setCharacterStream(int parameterIndex,java.io.Reader reader,long length)
*第二个参数接收的是一个流对象，因为大文本不应该用String来接收，String太大会导致内存溢出
*第三个参数接收的是文件的大小
*
* */
public class Demo5 {

    @Test
    public void add() {

        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;

        try {
            connection = JdbcUtils.getConnection();
            String sql = "INSERT INTO test2 (bigTest) VALUES(?) ";
            preparedStatement = connection.prepareStatement(sql);

            //获取到文件的路径
            String path = Demo5.class.getClassLoader().getResource("BigTest").getPath();
            File file = new File(path);
            FileReader fileReader = new FileReader(file);

            //第三个参数，由于测试的Mysql版本过低，所以只能用int类型的。高版本的不需要进行强转
            preparedStatement.setCharacterStream(1, fileReader, (int) file.length());

            if (preparedStatement.executeUpdate() > 0) {
                System.out.println("插入成功");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(connection, preparedStatement, null);
        }


    }

    /*
    * 读取大文本数据，通过ResultSet中的getCharacterStream()获取流对象数据
    * 
    * */
    @Test
    public void read() {

        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            connection = JdbcUtils.getConnection();
            String sql = "SELECT * FROM test2";
            preparedStatement = connection.prepareStatement(sql);
            resultSet = preparedStatement.executeQuery();

            if (resultSet.next()) {

                Reader reader = resultSet.getCharacterStream("bigTest");

                FileWriter fileWriter = new FileWriter("d:\\abc.txt");
                char[] chars = new char[1024];
                int len = 0;
                while ((len = reader.read(chars)) != -1) {
                    fileWriter.write(chars, 0, len);
                    fileWriter.flush();
                }
                fileWriter.close();
                reader.close();

            }
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(connection, preparedStatement, resultSet);
        }

    }
  ```
使用JDBC连接MYsql数据库操作二进制数据:
```
public class Demo6 {

    @Test
    public void add() {


        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;


        try {
            connection = JdbcUtils.getConnection();
            String sql = "INSERT INTO test3 (blobtest) VALUES(?)";
            preparedStatement = connection.prepareStatement(sql);

            //获取文件的路径和文件对象
            String path = Demo6.class.getClassLoader().getResource("1.wmv").getPath();
            File file = new File(path);

            //调用方法
            preparedStatement.setBinaryStream(1, new FileInputStream(path), (int)file.length());

            if (preparedStatement.executeUpdate() > 0) {

                System.out.println("添加成功");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(connection, preparedStatement, null);
        }

    }

    @Test
    public void read() {


        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;


        try {
            connection = JdbcUtils.getConnection();
            String sql = "SELECT * FROM test3";
            preparedStatement = connection.prepareStatement(sql);

            resultSet = preparedStatement.executeQuery();


            //如果读取到数据，就把数据写到磁盘下
            if (resultSet.next()) {
                InputStream inputStream = resultSet.getBinaryStream("blobtest");
                FileOutputStream fileOutputStream = new FileOutputStream("d:\\aa.jpg");

                int len = 0;
                byte[] bytes = new byte[1024];
                while ((len = inputStream.read(bytes)) > 0) {

                    fileOutputStream.write(bytes, 0, len);

                }
                fileOutputStream.close();
                inputStream.close();

            }

        } catch (SQLException e) {
            e.printStackTrace();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(connection, preparedStatement, null);
        }

    }
```

### 6.获取数据库的自动主键列
    比如说有一张老师表，一张学生表。现在来了一个新的老师，学生要跟着新老师上课。我首先要知道老师的id编号是多少，学生才能知道跟着哪个老师学习【学生外键参照老师主键】。
```
@Test
public void test() {

    Connection connection = null;
    PreparedStatement preparedStatement = null;
    ResultSet resultSet = null;

    try {
        connection = JdbcUtils.getConnection();

        String sql = "INSERT INTO test(name) VALUES(?)";
        preparedStatement = connection.prepareStatement(sql);

        preparedStatement.setString(1, "ouzicheng");

        if (preparedStatement.executeUpdate() > 0) {

            //获取到自动主键列的值
            resultSet = preparedStatement.getGeneratedKeys();

            if (resultSet.next()) {
                int id = resultSet.getInt(1);
                System.out.println(id);
            }
        }

    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JdbcUtils.release(connection, preparedStatement, null);
    }
}    
```
### 8.调用数据库的存储过程
调用存储过程的语法：{call <procedure-name>[(<arg1>,<arg2>, ...)]}
调用函数的语法：    {?= call <procedure-name>[(<arg1>,<arg2>, ...)]}
* 定义一个过程，获取users表总记录数，将10设置到变量count中
create procedure simpleproc(out count int)
begin
    select count(id) into count from users;
end
java代码示例：
```
 String sql = "{call simpleproc(?)}";
    Connection conn = JdbcUtil.getConnection();
    CallableStatement cstmt = conn.prepareCall(sql);
    cstmt.registerOutParameter(1,Types.INTEGER);
    cstmt.execute();
    Integer count = cstmt.getInt(1);
    System.out.println("共有" + count + "人");
```
* 定义一个函数，完成字符串拼接
create function hello( s char(20) ) returns char(50) 
return concat('hello，',s,'!');
java代码示例：
```
String sql = "{? = call hello(?)}";
    Connection conn = JdbcUtil.getConnection();
    CallableStatement cstmt = conn.prepareCall(sql);
    cstmt.registerOutParameter(1,Types.VARCHAR);
    cstmt.setString(2,"zhaojun");
    cstmt.execute();
    String value = cstmt.getString(1);
    System.out.println(value);
    JdbcUtil.close(cstmt);
    JdbcUtil.close(conn);
```


### 7.事务和批处理
    事务有什么特性或者说有什么作用？
    原子性：最小的单元，如果一个是失败了，则一切的操作将全部失败。
    一致性：如果事务出现错误，则回到最原始的状态
    隔离性：多个事务之间无法访问，只有当事务完成后才可以看到结果
    持久性：当一个系统崩溃时，一个事务依然可以提交，当事务完成后，操作结果保存在磁盘中，不会被回滚

保存点可以更细粒度地控制回滚操作，而不用每次都退回到初始点。
    什么又是批量更新？批量更新包括批量增删改，当我们一次性要插入很多条数据的时候，假设我们每次提交一次又获取数据库连接一次，然后又关闭数据库连接，而且数据库连接是一个耗时操作，这样会大大降低性能，当然可以使用连接池来解决这个问题，批量更新呢，则先把数据放入一个队列里，并没有真正存入数据库中，当调用commit()方法的时候，队列的数据的操作一次性收集和提交。
    通过executeBath()方法批量处理执行SQL语句，返回一个int[]数组，该数组代表各句SQL的返回值.
     示例：
```
  public class Client {
    public static void main(String [] args) throws SQLException {
        long time=System.currentTimeMillis();
        Connection connection=null;
        PreparedStatement pStatement=null;
        boolean autoCommit=false;
        Savepoint savepoint=null;
        try {
            connection=JDBCUtil.getConnection();
            autoCommit=connection.getAutoCommit();
            connection.setAutoCommit(false);
            String sql="insert into user(loginName,userName,password,sex)values(?,?,?,?)";
            pStatement=connection.prepareStatement(sql);
            //设置保存点
            savepoint=connection.setSavepoint("savePoint");
            for(int i=0;i<1000;i++){
                pStatement.setString(1,"tony"+i);
                pStatement.setString(2,"user"+i);
                pStatement.setString(3,i+"");
                pStatement.setInt(4,i);
                //添加到队列
                pStatement.addBatch();
            }
            //批量执行
            pStatement.executeBatch();
            connection.commit();

        } catch (SQLException e) {
            e.printStackTrace();
            //回滚到保存点
            connection.rollback(savepoint);
        }finally {
            //把事务提交设置为最初设置
            connection.setAutoCommit(autoCommit);
        }
        long temp=System.currentTimeMillis()-time;
        System.out.println(temp+"ms");
    }
}
```    

### 6.连接池
* 为什么使用连接池？
  用户每次请求都需要向数据库获得链接，而数据库创建连接通常需要消耗相对较大的资源，创建时间也较长。假设网站一天10万访问量，数据库服务器就需要创建10万次连接，极大的浪费数据库的资源，并且极易造成数据库服务器内存溢出、拓机。
  其次Drivermanager.getConnection(jdbcurl)连接数据库，并不能满足高并发情况。因为connection不是线程安全的，一个connection对应的是一个事物。如果线程里的多个dao操作，用的不是是同一个connection，就无法保证事务。
  开源组织提供了数据源的独立实现：
* DBCP 数据库连接池
* C3P0 数据库连接池






    




