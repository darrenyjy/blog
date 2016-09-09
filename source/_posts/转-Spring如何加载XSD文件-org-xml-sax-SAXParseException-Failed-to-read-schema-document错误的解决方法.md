---
title: >-
  转:Spring如何加载XSD文件(org.xml.sax.SAXParseException: Failed to read schema
  document错误的解决方法)
date: 2016-06-06 08:49:08
tags:
- Spring
- 转发
categories:
- Java

---
本文原文连接: http://blog.csdn.net/bluishglc/article/details/7596118 ,转载请注明出处！

有时候你会发现过去一直启动正常的系统，某天启动时会报出形如下面的错误：



    org.xml.sax.SAXParseException: schema_reference.4: Failed to read schema document 'http://www.springframework.org/schema/beans/spring-beans-2.0.xsd', because 1) could not find the document; 2) the document could not be read; 3) the root element of the document is not <xsd:schema>.  


很显然，spring xml配置文件中指定的xsd文件读取不到了，原因多是因为断网或spring的官网暂时无法连接导致的。 你可以通过在浏览器输入xsd文件的URL，如：http://www.springframework.org/schema/beans/spring-beans-2.0.xsd 进行确认。


关于这个问题，网上有两种常见的解决方法，第一种简单有效，但是工作量大，即：把所有spring配置文件中url形式的xsd路径转换成指向本地xsd文件的classpath形式的路径，例如：classpath:org/springframework/beans/factory/xml/spring-beans-2.5.xsd ,再有一种方法就是在本机搭建web服务器，按URL创建相应文件夹，放入对应xsd文件，在本机hosts文件中加入"127.0.0.1 www.springframework.org".实际上，这两种方法都属于“歪打正着”式的方法，直正弄明白这一问题还需要从spring的XSD文件加载机制谈起。


首先：你必须知道一点：spring在加载xsd文件时总是先试图在本地查找xsd文件(spring的jar包中已经包含了所有版本的xsd文件)，如果没有找到，才会转向去URL指定的路径下载。这是非常合理的做法，并不像看上去的那样，每次都是从站点下载的。事实上，假如你的所有配置是正确定的，你的工程完全可以在断网的情况下启动而不会报上面的错误。Spring加载xsd文件的类是PluggableSchemaResolver，你可以查看一下它的源码来验证上述说法。另外，你可以在log4j.xml文件中加入：


    <logger name="org.springframework.beans.factory.xml">  
        <level value="all" />  
    </logger>  


通过日志了解spring是何加载xsd文件的。


接下来，问题就是为什么spring在本地没有找到需要的文件，不得不转向网站下载。关于这个问题，其实也非常简单。在很多spring的jar包里，在META-INF目录下都有一个spring.schemas，这是一个property文件，其内容类似于下面：



    http\://www.springframework.org/schema/beans/spring-beans-2.0.xsd=org/springframework/beans/factory/xml/spring-beans-2.0.xsd  
    http\://www.springframework.org/schema/beans/spring-beans-2.5.xsd=org/springframework/beans/factory/xml/spring-beans-2.5.xsd  
    http\://www.springframework.org/schema/beans/spring-beans-3.0.xsd=org/springframework/beans/factory/xml/spring-beans-3.0.xsd  
    ....  


实际上，这个文件就是spring关于xsd文件在本地存放路径的映射，spring就是通过这个文件在本地（也就是spring的jar里）查找xsd文件的。那么，查找不到的原因排除URL输入有误之外，可能就是声明的xsd文件版本在本地不存在。一般来说，新版本的spring jar包会将过去所有版本(应该是自2.0以后)的xsd打包，并在spring.schemas文件中加入了对应项，出现问题的情况往往是声明使用了一个高版本的xsd文件，如3.0，但依赖的spring的jar包却是2.5之前的版本，由于2.5版本自然不可能包含3.0的xsd文件，此时就会导致spring去站点下载目标xsd文件，如遇断网或是目标站点不可用，上述问题就发生了。


但是，在实现开发中，出现上述错误的几率并不高，最常见的导致这一问题的原因其实与使用了一个名为“assembly”的maven打包插件有关。很多项目需要将工程连同其所依赖的所有jar包打包成一个jar包，maven的assembly插件就是用来完成这个任务的。但是由于工程往往依赖很多的jar包，而被依赖的jar又会依赖其他的jar包，这样，当工程中依赖到不同的版本的spring时，在使用assembly进行打包时，只能将某一个版本jar包下的spring.schemas文件放入最终打出的jar包里，这就有可能遗漏了一些版本的xsd的本地映射，进而出现了文章开始提到的错误。如果你的项目是打成单一jar的，你可以通过检查最终生成的jar里的spring.schemas文件来确认是不是这种情况。而关于这种情况，解决的方法一般是推荐使用另外一种打包插件shade,它确实是一款比assembly更加优秀的工具，在对spring.schemas文件处理上，shade能够将所有jar里的spring.schemas文件进行合并，在最终生成的单一jar包里，spring.schemas包含了所有出现过的版本的集合！


以上就是spring加载XSD文件的机制和出现问题的原因分析。实际上，我们应该让我们工程在启动时总是加载本地的xsd文件，而不是每次去站点下载，做到这一点就需要你结合上述提及的种种情况对你的工程进行一番检查。