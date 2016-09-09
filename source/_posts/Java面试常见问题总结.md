---
title: Java面试常见问题总结
date: 2016-07-03 16:39:15
tags:
- 面试
categories:
- Java

---


![](/uploads/interview/1.jpg "")

1. 自我介绍
	让对方在最短的时间里了解你的教育背景，实习经历，工作经验，兴趣爱好，职业规划，性格特征，以及你的优势等等
2. 项目
   项目简要介绍，你的主要工作，主要技术，难点，解决方案，心得体会
3. 运行时多态的解释：a.运行时多态是指程序中定义的引用变量所指向的具体类型和b.通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误；
	```java
	class Father{
	     void say(){
	     	System.out.println("father");
	     }
	}
	
	class Son extends Father{
	  void say(){
	  	System.out.println("son");
	  }
	  void run(){
	    ...;
	  }
	}
	
	Father son=new Son();
	son.run();//编译出错
	```
	
![](/uploads/interview/2.jpg "")
![](/uploads/interview/3.jpg "")
![](/uploads/interview/4.jpg "")
![](/uploads/interview/5.jpg "")
![](/uploads/interview/6.jpg "")
![](/uploads/interview/7.jpg "")
![](/uploads/interview/8.jpg "")
![](/uploads/interview/9.jpg "")
![](/uploads/interview/10.jpg "")
![](/uploads/interview/11.jpg "")
![](/uploads/interview/12.jpg "")
![](/uploads/interview/13.jpg "")
![](/uploads/interview/14.jpg "")
![](/uploads/interview/15.jpg "")
![](/uploads/interview/16.jpg "")
![](/uploads/interview/17.jpg "")
![](/uploads/interview/18.jpg "")
![](/uploads/interview/19.jpg "")
![](/uploads/interview/20.jpg "")
![](/uploads/interview/21.jpg "")
![](/uploads/interview/22.jpg "")
![](/uploads/interview/23.jpg "")
![](/uploads/interview/24.jpg "")
![](/uploads/interview/25.jpg "")


4. TCP的三次握手和四次挥手的补充
   三次握手
		* 第一次握手：建立连接时，客户端A发送SYN包（SYN=j）到服务器B，并进入SYN_SEND状态，等待服务器B确认。
		* 第二次握手：服务器B收到SYN包，必须确认客户A的SYN（ACK=j+1），同时自己也发送一个SYN包（SYN=k），即SYN+ACK包，此时服务器B进入SYN_RECV状态。
		* 第三次握手：客户端A收到服务器B的SYN＋ACK包，向服务器B发送确认包ACK（ACK=k+1），此包发送完毕，客户端A和服务器B进入ESTABLISHED状态，完成三次握手。
完成三次握手，客户端与服务器开始传送数据。
        确认号：其数值等于发送方的发送序号 +1(即接收方期望接收的下一个序列号)。
   四次挥手
		* 客户端A发送一个FIN，用来关闭客户A到服务器B的数据传送。 
		* 服务器B收到这个FIN，它发回一个ACK，确认序号为收到的序号加1。和SYN一样，一个FIN将占用一个序号。 
		* 服务器B关闭与客户端A的连接，发送一个FIN给客户端A。 
		* 客户端A发回ACK报文确认，并将确认序号设置为收到序号加1。 

	CLOSE_WAIT
	  > 发起TCP连接关闭的一方称为client，被动关闭的一方称为server。被动关闭的server收到FIN后，但未发出ACK的TCP状态是CLOSE_WAIT。出现这种状况一般都是由于server端代码的问题，如果你的服务器上出现大量CLOSE_WAIT，应该要考虑检查代码。
   
   TIME_WAIT
	  > 根据TCP协议定义的3次握手断开连接规定,发起socket主动关闭的一方 socket将进入TIME_WAIT状态。TIME_WAIT状态将持续2个MSL(Max Segment Lifetime),在Windows下默认为4分钟，即240秒。TIME_WAIT状态下的socket不能被回收使用. 具体现象是对于一个处理大量短连接的服务器,如果是由服务器主动关闭客户端的连接，将导致服务器端存在大量的处于TIME_WAIT状态的socket， 甚至比处于Established状态下的socket多的多,严重影响服务器的处理能力，甚至耗尽可用的socket，停止服务。

5. 为什么需要TIME_WAIT？
   > TIME_WAIT是TCP协议用以保证被重新分配的socket不会受到之前残留的延迟重发报文影响的机制,是必要的逻辑保证。
6. 主动发起关闭连接的操作的一方将达到TIME_WAIT状态，而且这个状态要保持Maximum Segment Lifetime的两倍时间。为什么要这样做而不是直接进入CLOSED状态？

	原因有二：
	> 一、保证TCP协议的全双工连接能够可靠关闭
	> 二、保证这次连接的重复数据段从网络中消失

	先说第一点，如果Client直接CLOSED了，那么由于IP协议的不可靠性或者是其它网络原因，导致Server没有收到Client最后回复的ACK。那么Server就会在超时之后继续发送FIN，此时由于Client已经CLOSED了，就找不到与重发的FIN对应的连接，最后Server就会收到RST而不是ACK，Server就会以为是连接错误把问题报告给高层。这样的情况虽然不会造成数据丢失，但是却导致TCP协议不符合可靠连接的要求。所以，Client不是直接进入CLOSED，而是要保持TIME_WAIT，当再次收到FIN的时候，能够保证对方收到ACK，最后正确的关闭连接。

	再说第二点，如果Client直接CLOSED，然后又再向Server发起一个新连接，我们不能保证这个新连接与刚关闭的连接的端口号是不同的。也就是说有可能新连接和老连接的端口号是相同的。一般来说不会发生什么问题，但是还是有特殊情况出现：假设新连接和已经关闭的老连接端口号是一样的，如果前一次连接的某些数据仍然滞留在网络中，这些延迟数据在建立新连接之后才到达Server，由于新连接和老连接的端口号是一样的，又因为TCP协议判断不同连接的依据是socket pair，于是，TCP协议就认为那个延迟的数据是属于新连接的，这样就和真正的新连接的数据包发生混淆了。所以TCP连接还要在TIME_WAIT状态等待2倍MSL，这样可以保证本次连接的所有数据都从网络中消失。
	[参考文献:TCP的三次握手和四次挥手](http://www.cnblogs.com/Jessy/p/3535612.html "参考文献:TCP的三次握手和四次挥手")
 
![](/uploads/interview/26.jpg "")
![](/uploads/interview/27.jpg "")
![](/uploads/interview/28.jpg "")
	
4. java类加载过程：
	首先是加载：

    这一块虚拟机要完成3件事：

        通过一个类的全限定名来获取定义此类的二进制字节流。

        将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。

        在java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口。 
    然后是链接（验证，准备，解释），初始化
 检验的目的：确保class文件的字节流信息符合jvm的口味，不会让jvm感到不舒服。假如class文件是由纯粹的java代码编译过来的，自然不会出现类似于数组越界、跳转到不存在的代码块等不健康的问题，因为一旦出现这种现象，编译器就会拒绝编译了。但是，跟之前说的一样，Class文件流不一定是从java源码编译过来的，也可能是从网络或者其他地方过来的，甚至你可以自己用16进制写，假如jvm不对这些数据进行校验的话，可能一些有害的字节流会让jvm完全崩溃。

检验主要经历几个步骤：文件格式验证->元数据验证->字节码验证->符号引用验证 
用Class.forName(String className);来加载类的时候，也会执行初始化动作。注意:ClassLoader的loadClass(String className);方法只会加载并编译某类，并不会对其执行初始化。

8. Java序列化和transient

	

拦截器是基于java的反射机制的，而过滤器是基于函数回调。　　
拦截器不依赖与servlet容器，过滤器依赖与servlet容器。　　
拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。　
拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。
拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

#### 有关return和finally
```java
package jinyuanyao.com;

/**
 * Hello world!
 */
public class App {
    static String result = "";

    public static void main(String[] args) {
        System.out.println("Hello World!");

        System.out.println(method(1));
//        the output is 2
        method(0);
        System.out.print(result);
//        the output is 2334
    }

    public static String method(int i) {
        try {
            if (i == 1) {
                throw new Exception();
            }
        } catch (Exception e) {
            result += "2";
            return result;

        } finally {
            result += "3";
        }
        result += "4";
        return "";


    }


}

```
总结：
1、不管有木有出现异常，finally块中代码都会执行；
2、当try和catch中有return时，finally仍然会执行；
3、finally是在return后面的表达式运算后执行的（此时并没有返回运算后的值，而是先把要返回的值保存起来，管finally中的代码怎么样，返回的值都不会改变，任然是之前保存的值），所以函数返回值是在finally执行前确定的；
4、finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值。
