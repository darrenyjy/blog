---
title: Java动态代理和CGLib代理
date: 2016-08-24 16:38:15
tags:
- 面试
categories:
- Java

---
#### Java动态代理

```java
/**
 * HelloWorld.java
 */
public interface HelloWorld {
    public int say(String words);
}

/**
 * HelloWorldImplements.java
 */
public class HelloWorldImplements implements HelloWorld {
    public int say(String words) {
        System.out.println("I am saying:"+words);
        return 1;
    }
}

/**
 * HelloWorldHandler.java
 */
public class HelloWorldHandler implements InvocationHandler {

    Object target;

    public HelloWorldHandler(Object target) {
        super();
        this.target = target;
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before");
        Object result=method.invoke(target,args);
        System.out.println("after");
        return result;
    }

}


/**
 * Main.java
 */
public class Main {
    public static void main(String[] args) {
        HelleWorld helleWorld = new HelloWorldImplements();
        HelloWorldHandler helloWorldHandler = new HelloWorldHandler(helleWorld);
        //HelleWorld proxy =(HelleWorld) Proxy.newProxyInstance(HelleWorld.class.getClassLoader(), HelloWorldImplements.class.getInterfaces(), helloWorldHandler);
        HelleWorld proxy = (HelleWorld) Proxy.newProxyInstance(helleWorld.getClass().getClassLoader(), new Class[]{HelleWorld.class}, helloWorldHandler);
        proxy.say("fuck the world");

    }
```

#### JDK动态代理和CGLib动态代理的区别
* jdk动态代理的对象需要实现一个接口，不能对类直接进行代理
* CGLIB可以对类进行代理，基于继承去生成一个子类，在子类里对父类的方法进行覆盖，所以最好不要用final修饰方法
* CGLIB的性能和JDK有所区别