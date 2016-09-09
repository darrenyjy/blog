---
title: String池化及intern方法的作用
date: 2016-05-28 19:16:57
tags:
- 面试
categories:
- Java

---
## String池化
>String池 ：String是不可改变的，为了提高效率Java引用了字符串池的概念，例如new String("abc");首先会在String池中创建一个对象“abc”，因为有NEW的存在所以会分配地址空间copy String池的内容。当出现的String对象在String池中不存在时即在String池中创建该对象。

字符串对象的创建方式有两种如下:

	String s1 = new String("");//第一种不会入池
	
	String s2 = "";//第二种看情况而定(等号右边如果是常量则入池,非常量则不入池)

例:

	String s3 = "a" + "b"; //"a"是常量,"b"是常量,常量+常量=常量,所以会入池.

	String s4 = s1 + "b";   //s1是变量,"b"是常量,变量+常量!=常量,所以不会入池.

一旦入池的话,就会先查找池中有无此对象.如果有此对象,则让对象引用指向此对象;如果无此对象,则先创建此对象,再让对象引用指向此对象.

## intern()

> 存在于.class文件中的常量池，在运行期被JVM装载，并且可以扩充。String的intern()方法就是扩充常量池的一个方法；当一个String实例str调用intern()方法时，Java查找常量池中是否有相同Unicode的字符串常量，如果有，则返回其的引用，如果没有，则在常量池中增加一个Unicode等于str的字符串并返回它的引用；

例3：

    String s0= “kvill”;  
    String s1=new String(”kvill”);  
    String s2=new String(“kvill”);  
    System.out.println( s0==s1 );  
    System.out.println( “**********” );  
    s1.intern();  
    s2=s2.intern(); //把常量池中“kvill”的引用赋给s2  
    System.out.println( s0==s1);  
    System.out.println( s0==s1.intern() );  
    System.out.println( s0==s2 );  

结果为：

    false  
    **********  
    false //虽然执行了s1.intern(),但它的返回值没有赋给s1  
    true //说明s1.intern()返回的是常量池中”kvill”的引用  
    true  

最后我再破除一个错误的理解：

有人说，"使用String.intern()方法则可以将一个String类的保存到一个全局String表中，如果具有相同值的Unicode字符串已经在这个表中，那么该方法返回表中已有字符串的地址，如果在表中没有相同值的字符串，则将自己的地址注册到表中"。如果我把他说的这个全局的String表理解为常量池的话，他的最后一句话，“**如果在表中没有相同值的字符串，则将自己的地址注册到表中**”是错的：

看例4：

    String s1=new String("kvill");  
    String s2=s1.intern();  
    System.out.println( s1==s1.intern() );  
    System.out.println( s1+" "+s2 );  
    System.out.println( s2==s1.intern() );  

结果：

    false 
    kvill kvill  
    true  

在这个类中我们没有声名一个”kvill”常量，所以常量池中一开始是没有”kvill”的，当我们调用s1.intern()后就在常量池中新添加了一个”kvill”常量，原来的不在常量池中的”kvill”仍然存在，也就不是“将自己的地址注册到常量池中”了。
