
layout: '[layout]'
title: MySQL入门和常用sql指令
description: 学习好一段时间MySQL了，到现在基本上也有些心得了，既然如此，那我就把它们做个简单的总结。
date: 2016-6-27 18:15:08
tags: [MySQL]  

---
# MySQL介绍 # 

MySQL 为关系型数据库(Relational Database Management System), 这种所谓的"关系型"可以理解为"表格"的概念, 一个关系型数据库由一个或数个表格组成,

    表头(header): 每一列的名称;
    列(row): 具有相同数据类型的数据的集合;
    行(col): 每一行用来描述某个人/物的具体信息;
    值(value): 行的具体信息, 每个值必须与该列的数据类型相同;
    键(key): 表中用来识别某个特定的人\物的方法, 键的值在当前列中具有唯一性。

## MySQL服务的启动、停止与卸载 ##

**本文是在window下的经验之谈，linux有稍微不同，本博客还有一篇linux下MySQL安装的文章可以参考----2017.1.7**

在 Windows 命令提示符下运行:

	启动: net start MySQL
	
	停止: net stop MySQL
	
	卸载: sc delete MySQL
## MySQL脚本的基本组成 ##

与常规的脚本语言类似, MySQL 也具有一套对字符、单词以及特殊符号的使用规定, MySQL 通过执行 SQL 脚本来完成对数据库的操作, 该脚本由一条或多条MySQL语句(SQL语句 + 扩展语句)组成, 保存时脚本文件后缀名一般为 .sql。在控制台下, MySQL 客户端也可以对语句进行单句的执行而不用保存为.sql文件。

**标识符**

	标识符用来命名一些对象, 如数据库、表、列、变量等, 以便在脚本中的其他地方引用。MySQL标识符命名规则稍微有点繁琐, 这里我们使用万能命名规则: 标识符由字母、数字或下划线(_)组成, 且第一个字符必须是字母或下划线。

**对于标识符是否区分大小写取决于当前的操作系统, Windows下是不敏感的, 但对于大多数 linux\unix 系统来说, 这些标识符大小写是敏感的。**

 
**关键字:**

	MySQL的关键字众多, 这里不一一列出。后续内容会有介绍，没有提到的也该在自己的学习中国慢慢掌握。另外，这些关键字有自己特定的含义, 尽量避免作为标识符。

 
**语句:**

	MySQL语句是组成MySQL脚本的基本单位, 每条语句能完成特定的操作, 他是由 SQL 标准语句 + MySQL 扩展语句组成。

 
**函数:**

	MySQL函数用来实现数据库操作的一些高级功能, 这些函数大致分为以下几类: 字符串函数、数学函数、日期时间函数、搜索函数、加密函数、信息函数。

## MySQL中的数据类型 ##

MySQL有三大类数据类型, 分别为数字、日期\时间、字符串, 这三大类中又更细致的划分了许多子类型:

    数字类型
        整数: tinyint、smallint、mediumint、int、bigint
        浮点数: float、double、real、decimal
    日期和时间: date、time、datetime、timestamp、year
    字符串类型
        字符串: char、varchar
        文本: tinytext、text、mediumtext、longtext
        二进制(可用来存储图片、音乐等): tinyblob、blob、mediumblob、longblob

**事实上MySQL实现主从备份就是通过一个二进制日志的传递来实现的----2017.1.7**

## 使用MySQL数据库 ##

**登录到MySQL**

当 MySQL 服务已经运行时, 我们可以通过MySQL自带的客户端工具登录到MySQL数据库中, 首先打开命令提示符, 输入以下格式的命名:

mysql -h 主机名 -u 用户名 -p

    -h : 该命令用于指定客户端所要登录的MySQL主机名, 登录当前机器该参数可以省略;
    -u : 所要登录的用户名;
    -p : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。

以登录刚刚安装在本机的MySQL数据库为例, 在命令行下输入 mysql -u root -p 按回车确认, 如果安装正确且MySQL正在运行, 会得到以下响应:

	Enter password:

若密码存在, 输入密码登录, 不存在则直接按回车登录, 按照本文中的安装方法, 默认 root 账号是无密码的。登录成功后你将会看到` Welecome to the MySQL monitor... `的提示语。

然后命令提示符会一直以 mysql> 加一个闪烁的光标等待命令的输入, 输入 exit 或 quit 退出登录。


**创建一个数据库**

使用 create database 语句可完成对数据库的创建, 创建命令的格式如下:

	create database 数据库名 [其他选项];

例如我们需要创建一个名为 samp_db 的数据库, 在命令行下执行以下命令:

	create database samp_db character set gbk;

为了便于在命令提示符下显示中文, 在创建时通过 character set utf8 将数据库字符编码指定为 utf8。创建成功时会得到  

	Query OK, 1 row affected(0.02 sec) 的响应。

注意: MySQL语句以分号(;)作为语句的结束, 若在语句结尾不添加分号时, 命令提示符会以 -> 提示你继续输入(有个别特例, 但加分号是一定不会错的);

提示: 可以使用 `show databases`; 命令查看已经创建了哪些数据库。
选择所要操作的数据库

要对一个数据库进行操作, 必须先选择该数据库, 否则会提示错误:

	ERROR 1046(3D000): No database selected

**数据库进行使用的选择:**

在登录后使用 use 语句指定, 命令: use 数据库名;

use 语句可以不加分号, 执行 use samp_db 来选择刚刚创建的数据库, 选择成功后会提示: `Database changed`

这里稍微说明一下，


**创建数据库表**

使用 create table 语句可完成对表的创建, create table 的常见形式:

	create table 表名称(列声明);

以创建 students 表为例, 表中将存放 学号(id)、姓名(name)、性别(sex)、年龄(age)、联系电话(tel) 这些内容:

	create table students
	（
		id int unsigned not null auto_increment primary key,
		name char(8) not null,
		sex char(4) not null,
		age tinyint unsigned not null,
		tel char(13) null default "-"
	);
				

对于一些较长的语句在命令提示符下可能容易输错, 因此我们可以通过任何文本编辑器将语句输入好后保存为 createtable.sql 的文件中, 通过命令提示符下的文件重定向执行执行该脚本。

打开命令提示符, 输入: 

	mysql -D samp_db -u root -p < createtable.sql

**1.如果连接远程主机请加上 -h 指令; 2. createtable.sql 文件若不在当前工作目录下需指定文件的完整路径。**


**增加新用户**

格式：grant select on 数据库.* to 用户名@登录主机 identified by “密码”

1、增加一个用户test1密码为abc，让他可以在任何主机上登录，并对所有数据库有查询、插入、修改、删除的权限。首先用root用户连入MYSQL，然后键入以下命令：

	grant select,insert,update,delete on *.* to test1@”%” Identified by “abc”;

但增加的用户是十分危险的，你想如某个人知道test1的密码，那么他就可以在internet上的任何一台电脑上登录你的mysql数据库并对你的数据可以为所欲为了，解决办法见2。

2、增加一个用户test2密码为abc,让他只可以在localhost上登录，并可以对数据库mydb进行查询、插入、修改、删除的操作（localhost指本地主机，即MYSQL数据库所在的那台主机），

这样用户即使用知道test2的密码，他也无法从internet上直接访问数据库，只能通过MYSQL主机上的web页来访问了。

	grant select,insert,update,delete on mydb.* to test2@localhost identified by “abc”;

如果你不想test2有密码，可以再打一个命令将密码消掉。

	grant select,insert,update,delete on mydb.* to test2@localhost identified by “”;

**关于MySQL权限控制，我有一篇详细描述的文章，可以查阅。**

## 常用指令总结 ##

显示命令

1、显示当前数据库服务器中的数据库列表：

	mysql> SHOW DATABASES;

注意：mysql库里面有MYSQL的系统信息，我们改密码和新增用户，实际上就是用这个库进行操作。

2、显示数据库中的数据表：

	mysql> USE 库名；
	mysql> SHOW TABLES;

3、显示数据表的结构：

	mysql> DESCRIBE 表名;

4、建立数据库：

	mysql> CREATE DATABASE 库名;

5、建立数据表：

	mysql> USE 库名;
	mysql> CREATE TABLE 表名 (字段名 VARCHAR(20), 字段名 CHAR(1));

6、删除数据库：

	mysql> DROP DATABASE 库名;

7、删除数据表：

	mysql> DROP TABLE 表名；

8、将表中记录清空：

	mysql> DELETE FROM 表名;

9、显示表中的记录：

	mysql> SELECT * FROM 表名;

10、往表中插入记录：

	mysql> INSERT INTO 表名 VALUES (”hyq”,”M”);

11、更新表中数据：

	mysql-> UPDATE 表名 SET 字段名1='a',字段名2='b' WHERE 字段名3='c';

12、用文本方式将数据装入数据表中：

	mysql> LOAD DATA LOCAL INFILE “D:/mysql.txt” INTO TABLE 表名;

13、导入.sql文件命令：

	mysql> USE 数据库名;
	mysql> SOURCE d:/mysql.sql;

14、命令行修改root密码：

	mysql> UPDATE mysql.user SET password=PASSWORD('新密码') WHERE User='root';
	mysql> FLUSH PRIVILEGES;

15、显示use的数据库名：

	mysql> SELECT DATABASE();

16、显示当前的user：

	mysql> SELECT USER();

17、向表中插入数据

	insert [into] 表名 [(列名1, 列名2, 列名3, ...)] values (值1, 值2, 值3, ...);

18、查询表中的数据

	select 列名称 from 表名称 [查询条件];

**查询其实才是数据库操作的重点，这里只是简单总结**

19、 更新表中的数据

	update 表名称 set 列名称=新值 where 更新条件;

20、 删除表中的数据

delete from 表名称 where 删除条件;


 **创建后表的修改**

alter table 语句用于创建后对表的修改, 基础用法如下:

21、 添加列

	alter table 表名 add 列名 列数据类型 [after 插入位置];

	示例:
	
	在表的最后追加列 address: alter table students add address char(60);
	
	在名为 age 的列后插入列 birthday: alter table students add birthday date after age;

22、 修改列

	基本形式: alter table 表名 change 列名称 列新名称 新数据类型;
	
	示例:
	
	将表 tel 列改名为 telphone: alter table students change tel telphone char(13) default "-";
	
	将 name 列的数据类型改为 char(16): alter table students change name name char(16) not null;

23、 删除列

	基本形式: alter table 表名 drop 列名称;
	
	示例:
	
	删除 birthday 列: alter table students drop birthday;
	重命名表
	
	基本形式: alter table 表名 rename 新表名;
	
	示例:
	
	重命名 students 表为 workmates: alter table students rename workmates;

24、 删除整张表

	基本形式: drop table 表名;
	
	示例: 删除 workmates 表: drop table workmates;
	删除整个数据库
	
	基本形式: drop database 数据库名;
	
	示例: 删除 samp_db 数据库: drop database samp_db;


