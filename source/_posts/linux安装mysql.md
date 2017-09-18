layout: '[layout]'
description: 最近打算自己实现一个简单的MySQL数据库的主从备份，因为不管是读写分离，还是高可用都要通过实现主从备份的数据库。
title: linux下mysql数据库安装
date: 2017-1-14 18:10:31 
tags: [MySQL, 经验总结]
---

# 下载MySQL #
下载链接 搜狐镜像

http://mirrors.sohu.com/mysql/MySQL-5.7/  
搜狐镜像下载速度还不错，所以可以在上面下载。首先需要提一下，**下载的时候需要注意自己linux系统的版本**，我以centos为例子，因为我用的是centos6.4.那么我就需要下载带有el6的mysql版本，下载el7肯定是不行的，el6指代centos6.x和红帽子6.x。大版本不改动的话底层库就不会变，如果下错了版本会导致安装失败。

# 安装MySQL #

1、rpm bundel包下载到 /usr/local/src中：   

	[root@localhost src]# ls
	mysql-5.7.11-1.el6.x86_64.rpm-bundle.tar



2解压缩bundle包

	[root@localhost src]# tar xf mysql-5.7.11-1.el6.x86_64.rpm-bundle.tar 
	[root@localhost src]# ls
	mysql-5.7.11-1.el6.x86_64.rpm-bundle.tar
	mysql-community-client-5.7.11-1.el6.x86_64.rpm
	mysql-community-common-5.7.11-1.el6.x86_64.rpm
	mysql-community-devel-5.7.11-1.el6.x86_64.rpm
	mysql-community-embedded-5.7.11-1.el6.x86_64.rpm
	mysql-community-embedded-devel-5.7.11-1.el6.x86_64.rpm
	mysql-community-libs-5.7.11-1.el6.x86_64.rpm
	mysql-community-libs-compat-5.7.11-1.el6.x86_64.rpm
	mysql-community-server-5.7.11-1.el6.x86_64.rpm
	mysql-community-test-5.7.11-1.el6.x86_64.rpm

安装必须组件：

	yum install numactl
	yum install libaio
	yum install perl-Time-HiRes per-devel


3.安装mysql-community-server之前，必须安装mysql-community-client和mysql-community-common  rpm包。而安装community-client和community-common包之前，必须删除mysql-lib(系统自带的版本过低)

	[root@localhost src]# rpm -qa | grep mysql
	mysql-libs-5.1.73-3.el6_5.x86_64

	[root@localhost src]# yum remove mysql-libs
	Loaded plugins: fastestmirror, refresh-packagekit, security
	Existing lock /var/run/yum.pid: another copy is running as pid 19475.
	Another app is currently holding the yum lock; waiting for it to exit...
	  The other application is: PackageKit
	    Memory :  39 M RSS (569 MB VSZ)
	
	
	Dependencies Resolved
	
	===========================================================================================
	 Package                              Arch                         Version                                Repositor
	===================================================================================================================
	Removing:
	 mysql-libs                           x86_64                       5.1.73-3.el6_5                         @anaconda
	Removing for dependencies:
	 cronie                               x86_64                       1.4.4-12.el6                           @anaconda
	 cronie-anacron                       x86_64                       1.4.4-12.el6                           @anaconda
	 crontabs                             noarch                       1.10-33.el6                            @anaconda
	 postfix                              x86_64                       2:2.6.6-6.el6_5                        @anaconda
	 sysstat                              x86_64                       9.0.4-27.el6                           @anaconda
	
	Transaction Summary
	===================================================================================================================
	Remove        6 Package(s)
	
	Installed size: 15 M
	Is this ok [y/N]: y
	Downloading Packages:
	Running rpm_check_debug
	Running Transaction Test
	Transaction Test Succeeded
	Running Transaction
	  Erasing    : sysstat-9.0.4-27.el6.x86_64                                                                         
	  Erasing    : cronie-1.4.4-12.el6.x86_64                                                                          
	  Erasing    : cronie-anacron-1.4.4-12.el6.x86_64                                                                  
	  Erasing    : crontabs-1.10-33.el6.noarch                                                                         
	  Erasing    : 2:postfix-2.6.6-6.el6_5.x86_64                                                                      
	  Erasing    : mysql-libs-5.1.73-3.el6_5.x86_64                                                                    
	  Verifying  : cronie-anacron-1.4.4-12.el6.x86_64                                                                  
	  Verifying  : mysql-libs-5.1.73-3.el6_5.x86_64                                                                    
	  Verifying  : sysstat-9.0.4-27.el6.x86_64                                                                         
	  Verifying  : crontabs-1.10-33.el6.noarch                                                                         
	  Verifying  : cronie-1.4.4-12.el6.x86_64                                                                          
	  Verifying  : 2:postfix-2.6.6-6.el6_5.x86_64                                                                      
	
	Removed:
	  mysql-libs.x86_64 0:5.1.73-3.el6_5                                                                               
	
	Dependency Removed:
	  cronie.x86_64 0:1.4.4-12.el6    cronie-anacron.x86_64 0:1.4.4-12.el6    crontabs.noarch 0:1.10-33.el6    postfix.
	
	Complete!



4.上面卸载了mysql-lib旧的包，现在安装mysql-lib最新版的文件，这个包在rpm-bundle里面都有，

直接安装即可

	[root@localhost src]# rpm -ivh mysql-community-common-5.7.11-1.el6.x86_64.rpm 
	warning: mysql-community-common-5.7.11-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
	Preparing...                ########################################### [100%]
	   1:mysql-community-common ########################################### [100%]
	
	[root@localhost src]# ls
	mysql-5.7.11-1.el6.x86_64.rpm-bundle.tar        mysql-community-embedded-5.7.11-1.el6.x86_64.rpm        mysql-commu
	mysql-community-client-5.7.11-1.el6.x86_64.rpm  mysql-community-embedded-devel-5.7.11-1.el6.x86_64.rpm  mysql-commu
	mysql-community-common-5.7.11-1.el6.x86_64.rpm  mysql-community-libs-5.7.11-1.el6.x86_64.rpm
	mysql-community-devel-5.7.11-1.el6.x86_64.rpm   mysql-community-libs-compat-5.7.11-1.el6.x86_64.rpm
	[root@localhost src]# rpm -ivh mysql-community-libs-5.7.11-1.el6.x86_64.rpm 
	warning: mysql-community-libs-5.7.11-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
	Preparing...                ########################################### [100%]
	   1:mysql-community-libs   ########################################### [100%]
	[root@localhost src]# rpm -ivh mysql-community-client-5.7.11-1.el6.x86_64.rpm 
	warning: mysql-community-client-5.7.11-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
	Preparing...                ########################################### [100%]
	   1:mysql-community-client ########################################### [100%]
	[root@localhost src]# rpm -ivh mysql-community-server-5.7.11-1.el6.x86_64.rpm 
	warning: mysql-community-server-5.7.11-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
	Preparing...                ########################################### [100%]
	   1:mysql-community-server ########################################### [100%]


5.启动mysql 服务

	[root@localhost src]# service mysql start
	mysql: unrecognized service
	[root@localhost src]# service mysqld start
	Initializing MySQL database:                               [  OK  ]
	Installing validate password plugin:                       [  OK  ]
	Starting mysqld:                                           [  OK  ]
	[root@localhost src]# netstat -tlunp
	Active Internet connections (only servers)
	Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
	tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1525/sshd           
	tcp        0      0 127.0.0.1:631               0.0.0.0:*                   LISTEN      1384/cupsd          
	tcp        0      0 :::22                       :::*                        LISTEN      1525/sshd           
	tcp        0      0 ::1:631                     :::*                        LISTEN      1384/cupsd          
	tcp        0      0 :::3306                     :::*                        LISTEN      19928/mysqld        
	udp        0      0 0.0.0.0:68                  0.0.0.0:*                               19346/dhclient      
	udp        0      0 0.0.0.0:631                 0.0.0.0:*                               1384/cupsd




6.rpm安装mysql后，会自动初始化一个密码，在日志中

	[root@localhost src]# cat /var/log/mysqld.log | more
	2016-03-11T01:44:19.913630Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --ex
	plicit_defaults_for_timestamp server option (see documentation for more details).
	2016-03-11T01:44:22.271099Z 0 [Warning] InnoDB: New log files created, LSN=45790
	2016-03-11T01:44:22.584299Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
	2016-03-11T01:44:23.050146Z 0 [Warning] No existing UUID has been found, so we assume that this is the first
	 time that this server has been started. Generating a new UUID: c7bf1a6a-e72a-11e5-8737-000c29a05942.
	2016-03-11T01:44:23.062259Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cann
	ot be opened.
	+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
	
	2016-03-11T01:44:23.093873Z 1 [Note] A temporary password is generated for root@localhost: 8%<rjn;+,11Y
	
	+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
	2016-03-11T01:44:28.470492Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --ex
	plicit_defaults_for_timestamp server option (see documentation for more details).
	2016-03-11T01:44:28.471726Z 0 [Note] /usr/sbin/mysqld (mysqld 5.7.11) starting as process 19672 ...
	2016-03-11T01:44:28.626135Z 0 [Note] InnoDB: PUNCH HOLE support available
	2016-03-11T01:44:28.626205Z 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
	2016-03-11T01:44:28.626220Z 0 [Note] InnoDB: Uses event mutexes
	2016-03-11T01:44:28.626233Z 0 [Note] InnoDB: GCC builtin __sync_synchronize() is used for memory barrier





7.修改 mysql root密码，由于最新的mysql版本对密码策略有要求，所以必须增加复杂程度才能通过

**注意，用刚才的随机密码登陆mysql.**

	[root@localhost src]# mysql -uroot -p
	Enter password: 此处输入刚才日志文件中的随机密码
	Welcome to the MySQL monitor.  Commands end with ; or g.
	Your MySQL connection id is 14
	Server version: 5.7.11
	
	Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or 'h' for help. Type 'c' to clear the current input statement.
	
	mysql> set password='Yes@126.com';
	Query OK, 0 rows affected (0.15 sec)
	
	mysql> quit;
	Bye
	
	[root@localhost src]# mysql -uroot -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or g.
	Your MySQL connection id is 15
	Server version: 5.7.11 MySQL Community Server (GPL)
	
	Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or 'h' for help. Type 'c' to clear the current input statement.


至此，安装完成。不是每个人都这么顺利的，事实上我也是安装了三次才成功。花了不少时间，希望有人能通过这个方法一次成功。

# 附 #

**很坑的SELinux**

什么是selinux以下引用百度百科:
> SELinux(Security-Enhanced Linux) 是美国国家安全局（NSA）对于强制访问控制的实现，是 Linux历史上最杰出的新安全子系统。NSA是在Linux社区的帮助下开发了一种访问控制体系，在这种访问控制体系的限制下，进程只能访问那些在他的任务中所需要文件。SELinux 默认安装在 Fedora 和 Red Hat Enterprise Linux 上，也可以作为其他发行版上容易安装的包得到。
> SELinux 是 2.6 版本的 Linux 内核中提供的强制访问控制(MAC）系统。对于目前可用的 Linux安全模块来说，SELinux 是功能最全面，而且测试最充分的，它是在 20 年的 MAC 研究基础上建立的。SELinux 在类型强制服务器中合并了多级安全性或一种可选的多类策略，并采用了基于角色的访问控制概念。[1]  


简单来说，如果你开启了这个东西，更改数据库配置文件很可能会出现权限不够的问题，导致更改失败，即使你给你的文件夹足够的读写权限。  

在red hat系列的linux中selinux对哪些daemon可以进行怎么样的操作是有限制的，mysql的select into outfile的命令是mysql的daemon来负责写文件操作的。写文件之前当然要具有写文件的权限。而selinux对这个权限做了限制。如果selinux是关闭的吧，这个命令执行是没有问题的  
 
	mysql> select user from user into outfile '/home/test.txt';  
	Query OK, 2 rows affected (0.02 sec)  

当时selinux开启时  
selinux对mysql的守护进程mysqld进行了限制。  

	mysql> select user from user into outfile '/home/test.txt';  
	ERROR 1 (HY000): Can't create/write to file '/home/test.txt' (Errcode: 13)   

出现了没有权限写的error。  
解决方法，可以直接关闭selinux。   

	setenforce 0          
最好的办法当时是配置文件里面去掉，可以在/etc/selinux中找到config  
修改SELINUX=disabled关闭selinux就可以了。

至于改他会不会带来什么大的问题，其实这个东西是鸡肋，食之无味，弃之可惜。所以不会带来什么问题，因为linux更多的是开源来集合大家的力量解决漏洞来处理安全问题的。

