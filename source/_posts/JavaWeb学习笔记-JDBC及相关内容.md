layout: '[layout]'
description: JDBC（Java Data Base Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。
title: JavaWeb-JDBC及相关内容
date: 2016-3-15 18:10:31 
tags: [Java, Javaweb]
---
# 1.什么是jdbc #
JDBC（Java Data Base Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。

----------

# 2.如何使用jdbc #

## 2.1得到Connection对象 ##

**以mysql为例：**

1. 导jar包。mysql-connector-java jar  
2. 加载驱动类:Class.forName(“类名”);  
3. 给出url、username、password等参数。  
4. 使用DriverManager类来的到Connection对象。  

**时刻牢记Jdbc中的四大参数:**

1. driverClassName:com.mysql.jdbc.Driver
1. url:jdbc:mysql://localhost:3306/数据库名
1. username:mysql用户名
1. password:mysql密码

Demo：

		public class Demo{
	    public void main(String[] args) throws ClassNotFoundException,SQLException{
	
	        Class.forName("com.mysql.jdbc.Driver");//加载驱动类。
	        String url="jdbc:mysql://localhost:3306/数据库名";
	        String username="root";
	        String password="123";
	
	        Connection con=DriverManager.getConnection(url,username,password);
	    }
	}

**代码分析:**

1. url的格式为–jdbc:厂商名称:子协议(由厂商自己来规定)。对于mysql而言，它的子协议结构的格式为://localhost:3306/数据库名。
2. 出现SQLException的原因:1.url username password 是否正确。2.检查是否打开了sql服务器
3. 出现ClassNotFoundException的原因:1.没导入驱动包。2.Class.forName()传入的字符串参数错误。

## 2.2 增删查改 ##
**增删改Demo：**

	public class Demo{
	    public void main(String[] args) throws ClassNotFoundException,SQLException{
	
	        Class.forName("com.mysql.jdbc.Driver");//加载驱动类。
	        String url="jdbc:mysql://localhost:3306/数据库名";
	        String username="root";
	        String password="123";
	
	        Connection con=DriverManager.getConnection(url,username,password);
	
	        Statement stmt=con.createStatement();//调用Connection的方法创建Statement对象，它是sql语句的发送器，功能就是向数据库发送sql语句
	        String sql＝"insert into stu values('...','...','...','...')";//右括号不需要打分号，打了就会出错，因为程序会自动帮我们加。
	        stmt.executeUpdate(sql);//调用此方法向数据库发送sql语句。该语句返回的值为改变数据库的行数。
	    }
	}

**查Demo：**

	public class Demo{
	    public void main(String[] args) throws ClassNotFoundException,SQLException{
	
	        Class.forName("com.mysql.jdbc.Driver");//加载驱动类。
	        String url="jdbc:mysql://localhost:3306/数据库名";
	        String username="root";
	        String password="123";
	
	        Connection con=DriverManager.getConnection(url,username,password);
	
	        Statement stmt=con.createStatement();
	
	        String sql="selece * from stu";
	
	        ResultSet rs=stmt.execute(sql);
	
	        /*
	         *解析ResultSet
	         *ResultSet提供了一系列的getXxx()方法
	         */
	         while(rs.next())//第一次调用next()方法是将光标移动到该表的第一行
	         {
	             rs.getInt(1);//通过列编号来获取该列的值
	             rs.getString("name");//通过列名称来获取该列的值。
	         }
	    }
	}

## 2.3关闭资源 ##

关闭资源时采用倒关的手法将对象进行处理:即先得到的对象后关，后得到的对象先关。

	rs.close();
	stmt.close();
	con.close();

为了注意代码的规范化:在try外给出引用的定义，在try内位对象实例化，在finally中对资源进行关闭。


## 2.4PreparedStatement(预处理) ##

### 2.4.1介绍 ###

PrepaerdStatement是Statement的子接口。下面通过一个例子学习PreparedStatement，注意与上述例子中Statement的区别。
	
	public class Demo{
	    public void main(String[] args) throws ClassNotFoundException,SQLException{
	
	        Class.forName("com.mysql.jdbc.Driver");//加载驱动类。
	        String url="jdbc:mysql://localhost:3306/数据库名";            String username="root";
	        String password="123";
	
	        Connection con=DriverManager.getConnection(url,username,password);
	
	        String sql="insert into stu where username=? and password=?";//定义sql模板，即参数以问号的形式给出
	
	        PreparedStatement pst=con.prepaerStatement(sql);
	        pst.setString(1,username);//数字1代表第一个问号
	        pst.setString(2.password);//数字2代表第二个问号
	        pst.executeUpdate();//向数据库发送sql语句
	    }
	}
### 2.4.2预处理的原理 ###

服务器的工作:

1. 检验sql语句的语法
2. 编译：一个与函数相似的东西
3. 执行：调用函数
PreparedStatement使用的前提就是:连接的数据库必须支持预处理。

以后我们要学的DAO模式就是写一个类，把访问数据库的代码封装起来，DAO在数据库与业务逻辑层之间。

## 2.5.Java中的时间类型和mysql中的时间类型转换 ##

### 2.5.1数据库类型与Java中类型的对应关系 ###

数据库中的DATE–>java.sql.Date–>java.util.Date;

数据库中的TIME–>java.sql.Time–>java.util.Date;

数据库中的TIMESTAMP–>java.sql.Timestamp–>java.util.Date;

**需要注意的是:**

1. 领域对象(domain)中的所有属性不能出现java.sql包下的东西，即不能使用java.sql.Date、java.sql.Time、java.sql.TimeStamp。
2. ResultSet的getDate()返回的是java.sql.Date()。
3. PreparedStatment的setDate(int,Date),其中第二个参数是sql包下的java.sql.Date()。为了在java中使用sql包下的时间类型，这是就出现了时间类型的转换。


### 2.5.2转换 ###

将util包下的Date转换为sql包下的Date、Time、Timestamp

步骤如下:

1. 把util的的Date转换成毫秒值。
2. 使用毫秒值创建sql的Date、Time、Timestamp


	java.util.Date date=new java.util.Date();
	
	long l=date.getTime();

	java.sql.Date sqlDate=new java.sql.Date(l);


将sql包下的Date、Time、Timestamp转换为util包下的Date

这一步不需要处理了，因为sql包下的Date、Time、和Timestamp继承自util包下的Date。所以可以直接用:

	new java.util.Date()=new java.sql.Date();或
	new java.util.Date()=new java.sql.Time();或
	new java.util.Date()=new java.sql.Timestamp();  

# 3.总结 #


因为连接是一个非常消耗资源的东西，所以每次SQL操作都需要建立和关闭连接，这势必会消耗大量的资源开销，这应该避免，可以采用连接池，对连接进行统一维护，不必每次都建立和关闭。事实上这是很多对JDBC进行封装的工具所采用的。也是下个部分的笔记重点。

