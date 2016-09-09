---
title: Spring IOC相关内容
date: 2016-05-30 13:02:07
tags:
- Spring
categories:
- Java

---

### BeanFactory和ApplicationContext

* BeanFactory在初始化容器时并未实例化Bean,而是在第一次访问才实例化目标，ApplicationContext在初始化应用上下文时就实例化了所有单实例的Bean。
* ApplicationContext:XMlApplicationContext(ClassPathXmlApplicationContext、FileSystemApplicationContext)
AnnotationConfigApplicationContext
WebApplicationContext.



### Bean的相关知识点

##### 1.Bean的作用域
* singleton:当一个bean的作用域为singleton, 那么Spring IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。

	注意：Singleton作用域是Spring中的缺省作用域。要在XML中将bean定义成singleton，可以这样配置： 

<bean id="empServiceImpl" class="cn.csdn.service.EmpServiceImpl" scope="singleton">

* prototype：一个bean定义对应多个对象实例。Prototype作用域的bean会导致在每次对该bean请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例。根据经验，对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。

* request：在一次HTTP请求中，一个bean定义对应一个实例；即每次HTTP请求将会有各自的bean实例， 它们依据某个bean定义创建而成。该作用域仅在基于web的Spring ApplicationContext情形下有效。

	考虑下面bean定义：
	
		<bean id="loginAction" class=cn.csdn.LoginAction" scope="request"/>

	针对每次HTTP请求，Spring容器会根据loginAction bean定义创建一个全新的LoginAction bean实例， 且该loginAction bean实例仅在当前HTTP request内有效，因此可以根据需要放心的更改所建实例的内部状态， 而其他请求中根据loginAction bean定义创建的实例，将不会看到这些特定于某个请求的状态变化。 当处理请求结束，request作用域的bean实例将被销毁。

* session：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

	考虑下面bean定义：

		<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>

	针对某个HTTP Session，Spring容器会根据userPreferences bean定义创建一个全新的userPreferences bean实例， 且该userPreferences bean仅在当前HTTP Session内有效。 与request作用域一样，你可以根据需要放心的更改所创建实例的内部状态，而别的HTTP Session中根据userPreferences创建的实例， 将不会看到这些特定于某个HTTP Session的状态变化。 当HTTP Session最终被废弃的时候，在该HTTP Session作用域内的bean也会被废弃掉。

* global session：在一个全局的HTTP Session中，一个bean定义对应一个实例。典型情况下，仅在使用portlet context的时候有效。该作用域仅在基于web的Spring ApplicationContext情形下有效。

	考虑下面bean定义：

		<bean id="userPreferences" class="com.foo.UserPreferences" scope="globalSession"/>

	global session作用域类似于标准的HTTP Session作用域，不过它仅仅在基于portlet的web应用中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个portlet web应用的各种不同的portlet所共享。在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内。

	请注意：假如你在编写一个标准的基于Servlet的web应用，并且定义了一个或多个具有global session作用域的bean，系统会使用标准的HTTP Session作用域，并且不会引起任何错误。

##### 2.Bean的命名

* Spring配置文件中不能出现两个相同的id的Bean，但是可以出现两个相同name的Bean,如果有多个相同name的Bean，通过getBean(BeanName)将返回最后申明的那个Bean.
* ![Bean的命名](/uploads/2.jpg "Bean的命名")

##### 3.Bean的注入方式
* Bean的注入方式: 1. 属性注入 2.构造函数注入（注意参数的顺序和类型） 
* ref属性：bean,local,parent

##### 4.Bean的自动装配
* ![过滤表达式](/uploads/5.spring.jpg "过滤表达式")

##### 5.注解

* Component、Service、Controller、Repository、Import、Configuration、Bean、Autowired、ImportResource、PostConstruct、PreDestory

##### 6.过滤表达式

* ![过滤表达式](/uploads/3.spring.jpg "过滤表达式")

##### 7.Bean的配置方式有哪几种
* 基于xml的配置
* 基于注解的配置
* 基于Java类的配置
![Bean不同配置方式的比较](/uploads/2.spring.jpg "Bean不同配置方式的比较")
![Bean不同配置方式的适用场合](/uploads/4.spring.jpg "Bean不同配置方式的适用场合")
