layout: '[layout]'
description: 在开发ssh项目过程中，遇到了failed to lazily initialize a collection of role XXXXXX, no session or session was closed 这个异常的麻烦，首先说说什么是延迟加载：
title: hibernate-有关延迟加载的问题及处理
date: 2016-8-15 18:10:31 
tags: [经验总结, hibernate]
---

# 什么是延迟加载 #
在开发ssh项目过程中，遇到了failed to lazily initialize a collection of role: XXXXXX, no session or session was closed 这个异常的麻烦，首先说说什么是延迟加载：

> 延迟加载，可以简单理解为，只有在使用的时候，才会发出sql语句进行查询。延迟加载的有效期是在session打开的情况下，当session关闭后，会报异常。当调用load方法加载对象时，返回代理对象，等到真正用到对象的内容时才发出sql语句。

首先说说我个人的看法，hibernate延迟加载是一项有用的技术，在数据量不大的情况下，数据库受到的压力或许并不大，但是一旦遇到了很高的并发请求，这项技术就会给数据库很好的减压，我认为它本身是很好的一个东西。但是比较尴尬的是，如果对数据库所有数据都进行这样的处理，有些时候会误伤一些我们需要的数据请求。举例如下：  

我们在开发的时候经常会遇到延迟加载问题，在实体映射时，多对一和多对多中，多的一样的属性默认是lazy="true"(即，默认是延迟加载)，

	<many-to-one name="parent" class="Department" column="parentId" lazy="true"/>

延迟加载表现在:比如：我们要查询id为2的部门数据，但是有许多用户数据的部门外键是id为2，我们在查询的时候，由于默认lazy="true"(懒加载)，所以是不会查询部门外键为2的用户数据的，但是我们在一次session中，不仅不要部门数据，而且还有可能需要该部门对应的用户数据，由于默认设置为lazy="true"，所以我们在一次session中是获取不到该用户数据了。


那么怎么解决这些问题呢，我自己查阅资料找到了这么些解决方法：

# 延迟加载问题的处理 #

1、是把对应一对多的那两个列lazy=true改为lazy＝false即可；

2、对于查询中如果用的是xxx.load（class，id）则改为xxx,get(class，id)；

3、在web.xml文件中加入： 

   	
      <filter-name>hibernateFilter</filter-name>
      <filter-class>

          org.springframework.orm.hibernate3.support.OpenSessionInViewFilter</filter-class>

          <init-param>
            <param-name>singleSession</param-name>
            <param-value>false</param-value>
          </init-param>

	<!--这个--   <init-param>一定要加不然很可能会报错：   org.springframework.dao.InvalidDataAccessApiUsageException:Write operations are not allowed in read-only mode (FlushMode.NEVER) - turn your Session into FlushMode.AUTO or remove 'readOnly' marker from transaction definition
	-->
     </filter>

     <filter-mapping>
         <filter-name>hibernateFilter</filter-name>
         <url-pattern>*.mmg</url-pattern>
     </filter-mapping>

对以上方法进行一一测试，到后来结果都是一样，出现同样的异常，后来有苦苦在网上找资料，并自己努力思考，后来也了解到spring能很好地解决这个问题，Spring框架为hibernate延迟加载与DAO模式的整合提供了一种方便的解决方法。对那些不熟悉Spring与Hibernate集成使用的人，我不会在这里讨论过多的细节，但是我建议你去了解Hibernate与Spring集成的数据访问。以一个Web应用为例，Spring提供了OpenSessionInViewFilter和OpenSessionInViewInterceptor。我们可以随意选择一个类来实现相同的功能。两种方法唯一的不同就在于interceptor在Spring容器中运行并被配置在web应用的上下文中，而Filter在Spring之前运行并被配置在web.xml中。不管用哪个，他们都在请求将当前会话与当前（数据库）线程绑定时打开Hibernate会话。一旦已绑定到线程，这个打开了的Hibernate会话可以在DAO实现类中透明地使用。这个会话会为延迟加载数据库中值对象的视图保持打开状态。一旦这个逻辑视图完成了，Hibernate会话会在Filter的doFilter方法或者Interceptor的postHandle方法中被关闭。用spring解决这个问题而且不用把lazy设置为false，提高性能。

方法是：在web.xml中加入以下配置：

	<filter>     
		<filter-name>hibernateFilter</filter-name>     
		<filter-class>     
		org.springframework.orm.hibernate3.support.OpenSessionInViewFilter      
		</filter-class>     
	</filter>      
		<filter-mapping>     
		<filter-name>hibernateFilter</filter-name>     
		<url-pattern>/*</url-pattern>    
	</filter-mapping>

开始时，把这个配置随意地加到web.xml的最后，发现还是不行，后来又通过网络了解到是过滤器顺序的问题，应该是：
OpenSessionInViewFilter
ActionContextCleanUp
FilterDispatcher
的顺序，最后调整过滤器的顺序，一切问题解决。


