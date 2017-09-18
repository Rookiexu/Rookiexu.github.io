layout: '[layout]'
title: JavaWeb学习笔记-连接池
description: 数据库连接是一种关键的、有限的、昂贵的资源，这一点在多用户的网页应用程序中体现得尤为突出。对数据库连接的管理能显著影响到整个应用程序的伸缩性和健壮性，影响到程序的性能指标。数据库连接池正是针对这个问题提出来的。
date: 2016-3-16 18:09:02
tags: [Java, Javaweb]
---
# 数据库连接池 #

用户每次请求都需要向数据库获得链接，而数据库创建连接通常需要消耗相对较大的资源，创建时间也较长。假设网站一天10万访问量，数据库服务器就需要创建10万次连接，极大的浪费数据库的资源，并且极易造成数据库服务器内存溢出、拓机。如下图所示:
![](http://ohendw9u7.bkt.clouddn.com/blog/image/jpg/20161205_01.jpg)

数据库连接是一种关键的有限的昂贵的资源,这一点在多用户的网页应用程序中体现的尤为突出.。对数据库连接的管理能显著影响到整个应用程序的伸缩性和健壮性，影响到程序的性能指标。数据库连接池正式针对这个问题提出来的。数据库连接池负责分配，管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个。如下图所示:
![](http://ohendw9u7.bkt.clouddn.com/blog/image/jpg/20161205_02.jpg)

数据库连接池在初始化时将创建一定数量的数据库连接放到连接池中, 这些数据库连接的数量是由最小数据库连接数来设定的.无论这些数据库连接是否被使用,连接池都将一直保证至少拥有这么多的连接数量.连接池的最大数据库连接数量限定了这个连接池能占有的最大连接数,当应用程序向连接池请求的连接数超过最大连接数量时,这些请求将被加入到等待队列中。

**数据库连接池的最小连接数和最大连接数的设置要考虑到以下几个因素:**

1. 最小连接数(MinActive):是连接池一直保持的数据库连接,所以如果应用程序对数据库连接的使用量不大,将会有大量的数据库连接资源被浪费。

2. 最大连接数(MaxActive):是连接池能申请的最大连接数,如果数据库连接请求超过次数,后面的数据库连接请求将被加入到等待队列中,这会影响以后的数据库操作。

3. 如果最小连接数与最大连接数相差很大:那么最先连接请求将会获利,之后超过最小连接数量的连接请求等价于建立一个新的数据库连接.不过,这些大于最小连接数的数据库连接在使用完不会马上被释放,他将被放到连接池中等待重复使用或是空间超时后被释放。

##  池参数(所有池参数都有默认值) ##

	设置初始化大小:connection.setInitialSize();
	设置最小空闲连接数:connection.setMinIdle();
	设置最大空闲连接数:connection.setMaxIdle();
	设置最小连接数:connection.setMinActive();
	设置最大连接数:connection.setMaxActive();
	设置增量:一次创建的最小单位。
	设置最大的等待时间:connection.setMaxWait();

##   四大连接参数 ##

连接池也是使用Jdbc中的四大连接参数和驱动jar包来完成创建连接对象。

##  实现的接口 ##

连接池必须实现`javax.sql.DataSource`接口。

从连接池返回的Connection对象，它的close()方法与众不同。调用它的close()方法不是关闭，而是把连接归还给池。

## DBCP数据库连接池 ##

DBCP是Apache软件基金组织下的开源连接池实现，要使用DBCP数据源，需要应用程序应在系统中增加如下两个jar文件:

	Commons-dbcp.jar：连接池的实现
	Commons-pool.jar：连接池实现的依赖库

**Demo:**

	public class Demo{
	    public static void main(String[] args)
	    {
	        /*
	         *1.创建连接池对象
	         *2.配置四大参数
	         *3.配置池参数
	         *4.得到连接对象
	         */
	        BasicDataSource dataSource=new BasicDataSource();
	
	        dataSource.setDriverClassName(“com.mysql.jdbc.Driver”);
	        dataSource.setUrl(“jdbc:mysql://localhost:3306/mydb”);
	        dataSource.setUsername(“root”);
	        dataSource.setPassword(123);
	
	        dataSource.setMaxActive(20);
	        dataSource.setMinIdle(3);
	        dataSource.setMaxWait(1000);
	
	        Connection con=dataSource.getConnection();
	        System.out.println(con);
	
	      /*
	        *连接池内部使用四大参数创建了连接对象，即mysql驱动提供的Connection
	        *连接池使用mysql的连接对象进行了装饰，只对close()方法进行了增强！
	        *装饰之后的Connection的close()方法，用来把当前连接归还给池
	        */
	        con.close();//把连接归还给池。
	    }
	}

## c3p0数据库连接池 ##

c3p0,全名叫ComboPooledDataSource;

需要导入的jar包:
	
	连接池的实现:c3p0-0.9.5.2.jar
	依赖库:mchange－commons.jar

**Demo:**
	
		public class Demo{
	    public static void main(String[] args)
	    {
	        //在创建连接池对象时，这个对象就会自动加载配置文件，不用我们来指定。
	        ComboPooledDataSource data=new comboPooledDataSource();
	        Connection con=data.getConnection();
	        System.out.println(con);
	    }
	}

### c3p0配置文件的使用 ###

配置文件要求:

	文件名称:必须叫c3p0-config.xml。
	文件的位置:必须在src下。

c3p0配置文件:

	<?xml version="1.0" encoding="UTF-8"?>
	<c3p0-config>
	  <default-config>
		<property name="driverClass">com.mysql.jdbc.Driver</property>
		<property name="jdbcUrl">jdbc:mysql://localhost:3306/day13</property>
		<property name="user">root</property>
		<property name="password">abc</property>
	    <property name="initialPoolSize">10</property>
	    <property name="maxIdleTime">30</property>
	    <property name="maxPoolSize">100</property>
	    <property name="minPoolSize">10</property>
	  </default-config>
	</c3p0-config>

