---
title: MyBatis入门
date: 2016-06-23 15:35:40
tags:
- Mybatis
- Java Web
categories:
- Java

---

### 配置
[Mybatis在线使用文档](http://www.mybatis.org/mybatis-3/zh/getting-started.html "使用文档")
#### settings

useColumnLabel	使用列标签代替列名。不同的驱动在这方面会有不同的表现， 具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。

useGeneratedKeys	允许 JDBC 支持自动生成主键，需要驱动兼容。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作（比如 Derby）。

mapUnderscoreToCamelCase	是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。

### sqlMap

#### select
 resultType	 从这条语句中返回的期望类型的类的完全限定名或别名。注意如果是集合情形，那应该是集合可以包含的类型，而不能是集合本身。使用 resultType 或 resultMap，但不能同时使用。
 
#### resultMap

constructor - 类在实例化时,用来注入结果到构造方法中
	idArg - ID 参数;标记结果作为 ID 可以帮助提高整体效能
	arg - 注入到构造方法的一个普通结果
id – 一个 ID 结果;标记结果作为 ID 可以帮助提高整体效能
result – 注入到字段或 JavaBean 属性的普通结果
association – 一个复杂的类型关联;许多结果将包成这种类型
嵌入结果映射 – 结果映射自身的关联,或者参考一个
collection – 复杂类型的集
嵌入结果映射 – 结果映射自身的集,或者参考一个
discriminator – 使用结果值来决定使用哪个结果映射
	case – 基于某些值的结果映射
	嵌入结果映射 – 这种情形结果也映射它本身,因此可以包含很多相同的	元素,或者它可以参照一个外部的结果映射。
 
#### 自动映射
当自动映射查询结果时，MyBatis会获取sql返回的列名并在java类中查找相同名字的属性（忽略大小写）。 这意味着如果Mybatis发现了ID列和id属性，Mybatis会将ID的值赋给id。

通常数据库列使用大写单词命名，单词间用下划线分隔；而java属性一般遵循驼峰命名法。 为了在这两种命名方式之间启用自动映射，需要将 mapUnderscoreToCamelCase设置为true。



<resultMap id="authorResult" type="Author">
  <result property="username" column="author_username"/>
</resultMap>
With this result map both Blog and Author will be auto-mapped. But note that Author has an id property and there is a column named id in the ResultSet so Author's id will be filled with Blog's id, and that is not what you were expecting. So use the FULL option with caution.

Regardless of the auto-mapping level configured you can enable or disable the automapping for an specific ResultMap by adding the attribute autoMapping to it:

<resultMap id="userResultMap" type="User" autoMapping="false">
  <result property="password" column="hashed_password"/>
</resultMap>
#### 缓存
MyBatis 包含一个非常强大的查询缓存特性,它可以非常方便地配置和定制。MyBatis 3 中的缓存实现的很多改进都已经实现了,使得它更加强大而且易于配置。

默认情况下是没有开启缓存的,除了局部的 session 缓存,可以增强变现而且处理循环 依赖也是必须的。要开启二级缓存,你需要在你的 SQL 映射文件中添加一行:

	<cache/>
字面上看就是这样。这个简单语句的效果如下:

映射语句文件中的所有 select 语句将会被缓存。
映射语句文件中的所有 insert,update 和 delete 语句会刷新缓存。
缓存会使用 Least Recently Used(LRU,最近最少使用的)算法来收回。
根据时间表(比如 no Flush Interval,没有刷新间隔), 缓存不会以任何时间顺序 来刷新。
缓存会存储列表集合或对象(无论查询方法返回什么)的 1024 个引用。
缓存会被视为是 read/write(可读/可写)的缓存,意味着对象检索不是共享的,而 且可以安全地被调用者修改,而不干扰其他调用者或线程所做的潜在修改。