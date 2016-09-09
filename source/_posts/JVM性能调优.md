---
title: JVM性能调优
date: 2016-07-18 10:25:47
tags:
- JVM
categories:
- Java

---
> 转自：http://my.oschina.net/chape/blog/200790
http://www.cnblogs.com/gw811/archive/2012/10/19/2730258.html

### 1) Heap Size

-Xmx ---最大Heap Size，即上图的Total size（包括Eden+form+to，Tenured，不包含Perm，见上图），限制了年轻代和年老代的可分配最大值；

-Xms ---初始化分配的Heap Size

生产环境中ms一般设置成跟mx相等，因为若ms不等于mx那么在某些场景下JVM可能需要对Heap Size进行频繁的扩展和收缩，增加处理时间；
### 2）New/Young Generation Size

-Xmn ---最大年轻代大小，即上图中的Eden+S0+S1+Virtual

-XX:NewSize ---初始化年轻代大小，即上图中的Eden+S0+S1，在只设置了-Xmn不设置-XX:NewSize的情况下，NewSize等于mn。

生产环境中一般只需设置-Xmn或者设置mn和NewSize相等，理由和HeapSize的设置一样，避免容量震荡消耗资源；

### 3）Old Generation Size （Tenured）

-XX:NewRatio --- Old Size/New Size，通过年老代和年轻代的比例和Heap Size就可以算出年老代的大小。一般默认为8，若Heap Size为1024m，则 NewSize=HeapSize/(NewRatio+1)=114m，OldSize=HeapSize-NewSize=910m；

注意：-Xmn的优先级比-XX:NewRatio高，若-Xmn已指定，则OldSize=HeapSize-NewSize，无需再按比例计算。生产环境中一般只需指定-Xmn就足够了。

### 4）Eden和S0、S1

-XX:SurvivorRatio --- Eden/S0，即 Eden区和S0的比例，默认为8，若NewSize为114m，则S0=NewSize/(SurvivorRatio+2)=11.4m;

S0==S1，S0、S1的职能是一模一样的，又叫做From space和To space，在每一次minor gc后角色会交换。

注意：-XX类型的选项在不同的JDK版本或实现中定义可能有所区别，在近日的实践中发现，

在Linux jdk_1_5_0_10_x86版本中，SurvivorRatio=(YoungSize/S0)，而Linux jdk_1_5_0_20_x64版本中，SurvivorRatio=(Eden/S0)

所以，我们在实际的工程实践中还是应该用jmap -heap输出的jvm内存结构信息为准，不要想当然。

### 5）Permanent Generation Size

-XX:MaxPermSize ---最大持久代大小，默认为64m；

-XX:PermSize ---初始化持久代大小，默认为16m；

生产环境中一般设置MaxPermSize和PermSize相等，理由和HeapSize的设置一样，避免容量震荡消耗资源；

当应用引用的类比较多或者应用了一些动态类生产技术时应该加大该区的值，一般256m对服务器程序都很足够了。


### 6）Thread Stack Size

-Xss ---线程堆栈大小，一般用于存放方法入口参数和返回值，以及原子类型的本地变量（即方法内部变量）；

一般可设置为128k.

### 7）Direct Memory Size

-XX:MaxDirectMemorySize ---direct byte buffer用到的本地内存，在本blog的一篇《xsocket内存泄漏》文章中介绍过该参数的作用。默认跟mx相等，所以生产环境中一般不设置mx大于物理内存的一半。

### GC过程  

在讲述GC过程前我先解释一下JVM的两个控制参数：

-XX:TargetSurvivorRatio --- Survivor Space最大使用率，若存放对象的总大小超过该值，将引起对象向Old区迁移；

-XX:MaxTenuringThreshold --- Young区对象的最大任期阀值，即可经历minor gc的次数，超过该次数的对象直接迁移到Old区；

实际的TenuringThreshold由JVM通过Survivor Space的占用率和TargetSurvivorRatio动态计算，详情请查看参考资料。

### 引用计数算法

　　很多教科书判断对象是否存活的算法是这样的：给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器都为0的对象就是不可能再被使用的。笔者面试过很多的应届生和一些有多年工作经验的开发人员，他们对于这个问题给予的都是这个答案。

　　客观地说，引用计数算法（Reference Counting）的实现简单，判定效率也很高，在大部分情况下它都是一个不错的算法，也有一些比较著名的应用案例，例如微软的COM（Component Object Model）技术、使用ActionScript 3的FlashPlayer、Python语言以及在游戏脚本领域中被广泛应用的Squirrel中都使用了引用计数算法进行内存管理。但是，Java语言中没有选用引用计数算法来管理内存，其中最主要的原因是它很难解决对象之间的相互循环引用的问题。
　　
### 根搜索算法

　　在主流的商用程序语言中（Java和C#，甚至包括前面提到的古老的Lisp），都是使用根搜索算法（GC Roots Tracing）判定对象是否存活的。这个算法的基本思路就是通过一系列的名为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连（用图论的话来说就是从GC Roots到这个对象不可达）时，则证明此对象是不可用的。如图3-1所示，对象object 5、object 6、object7虽然互相有关联，但是它们到GC Roots是不可达的，所以它们将会被判定为是可回收的对象。

　　在Java语言里，可作为GC Roots的对象包括下面几种：

　　　　虚拟机栈（栈帧中的本地变量表）中的引用的对象。

　　　　方法区中的类静态属性引用的对象。

　　　　方法区中的常量引用的对象。

　　　　本地方法栈中JNI（即一般说的Native方法）的引用的对象。
　　　　
### 引用

　　无论是通过引用计数算法判断对象的引用数量，还是通过根搜索算法判断对象的引用链是否可达，判定对象是否存活都与“引用”有关。在JDK 1.2之前，Java中的引用的定义很传统：如果reference类型的数据中存储的数值代表的是另外一块内存的起始地址，就称这块内存代表着一个引用。这种定义很纯粹，但是太过狭隘，一个对象在这种定义下只有被引用或者没有被引用两种状态，对于如何描述一些“食之无味，弃之可惜”的对象就显得无能为力。我们希望能描述这样一类对象：当内存空间还足够时，则能保留在内存之中；如果内存在进行垃圾收集后还是非常紧张，则可以抛弃这些对象。很多系统的缓存功能都符合这样的应用场景。

　　在JDK 1.2之后，Java对引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（WeakReference）、虚引用（Phantom Reference）四种，这四种引用强度依次逐渐减弱。

　　强引用就是指在程序代码之中普遍存在的，类似“Object obj = new Object()”这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。

　　软引用用来描述一些还有用，但并非必需的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中并进行第二次回收。如果这次回收还是没有足够的内存，才会抛出内存溢出异常。在JDK 1.2之后，提供了SoftReference类来实现软引用。

　　弱引用也是用来描述非必需对象的，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK 1.2之后，提供了WeakReference类来实现弱引用。

　　虚引用也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是希望能在这个对象被收集器回收时收到一个系统通知。在JDK 1.2之后，提供了PhantomReference类来实现虚引用。
　　
