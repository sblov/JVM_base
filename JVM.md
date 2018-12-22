#### 1）内存溢出（简单实现）

```java
List<Object> list = new ArrayList<>();
while (true) {
	list.add(new Object());
}
```

​	通过在Run Configurations中设置VM arguments：`-XX:+HeapDumpOnOutOfMemoryError -Xms20m -Xmx20m`，时运行时内存为20m，并对堆内存溢出错误生成错误分析文件（java_pid12068.hprof，位于项目根目录下）

```java
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid12068.hprof ...
Heap dump file created [27984093 bytes in 0.102 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Unknown Source)
	at java.util.Arrays.copyOf(Unknown Source)
	at java.util.ArrayList.grow(Unknown Source)
	at java.util.ArrayList.ensureExplicitCapacity(Unknown Source)
	at java.util.ArrayList.ensureCapacityInternal(Unknown Source)
	at java.util.ArrayList.add(Unknown Source)
	at com.lov.jvm.JvmTest.main(JvmTest.java:13)
```

​	该文件需要通过内存分析工具进行查看分析（eclipse memory analysis）

![eclipse meomory analysis](C:\Users\Administrator\Desktop\JVM\溢出分析.png)

#### 2）JVM监控工具

​	在jdk包中包含了对JVM的监控工具：`jconsole`

![jconsole](C:\Users\Administrator\Desktop\JVM\jconsole.png)



---

# **Java虚拟机**

### 一、**了解历史**

#### 	1.Java之父

​		**Java之父**——[詹姆斯·高斯林](https://baike.baidu.com/item/%E8%A9%B9%E5%A7%86%E6%96%AF%C2%B7%E9%AB%98%E6%96%AF%E6%9E%97/9902700)出生于加拿大，是一位计算机编程天才。在[卡内基·梅隆大学](https://baike.baidu.com/item/%E5%8D%A1%E5%86%85%E5%9F%BA%C2%B7%E6%A2%85%E9%9A%86%E5%A4%A7%E5%AD%A6/514335)攻读计算机博士学位时，他编写了多处理器版本的Unix操作系统，是JAVA编程语言的创始人。 

#### 	2.java

- 1995      Oak -> java1.0	
- 1996      jdk1.0 -> jvm(Sun Classic VM)

- 1996      首届JavaOne大会

- 1997      jdk1.1 -> 内部类、反射、jar文件格式、jdbc、javabeans、rmi

- 1998      jdk1.2 -> j2SE、J2EE、J2ME（swing、jit[Just In Time Compiler ]、HotSpot VM）

- 2000      jdk1.3 -> Timer、java2d，从此高速发展，并规定两年发布一个新版本

- 2002      jdk1.4 -> struts、hibernate、spring1.x、正则表达式、NIO、日志、XML解析器，API逐渐完善

- 2004      jdk1.5 -> 自动装箱拆箱、泛型、注解、枚举、变长参数、增强for循环、spring2.x

- 2006      jdk1.6 ->javaEE、javaSE、javaME、jdk6、提供脚本语言支持（动态语言）、提供编译api以及http服务器api、宣布java开源

- 2011      jdk1.7 -> Oracle收购Sun（74亿）

- 2014      jdk1.8 -> StreamAPI、Lambda、新时间API、函数式接口

- 2017      jdk1.9

#### 3.Java技术体系

- Java程序设计语言

- 各硬件平台上的Java虚拟机

- Class文件格式

- JavaAPI

- 第三方的Java类库

#### 4.虚拟机

​	***Sun Classic VM***：世界第一款商用Java虚拟机，现在基本不用，只能使用纯解释器的方式来执行java代码

​	***Exact VM***：Exact Memory Management准确式内存管理；编译器与解释器混合工作及两级即时编译器；只在Solaris平台发布过

​	***HotSpot VM***：Sun公司收购在的项目，取代前两个VM，并在开源后oracle与open的jdk共用的VM

​	***KVM***：kilobyte形式，简单、轻量、高度可移植；在手机平台运行

​	***JRockit***：最初为BEA开发，之后BEA在2008年被Sun收购；专注于服务器端应用；有很好的垃圾收集器与独特的MissionControl服务套件（寻找生产环境中的系统的内存泄漏）

​	***J9***：IBM专用，与HotSpot VM类似

​	***Dalvik***：android的虚拟机，不算java虚拟机；执行dex文件（dalvik executable）

​	***Microsoft JVM***：被Sun公司指控被告，最终停止里Mirosoft JVM发展

​	***Azul VM、Liquid VM***：高性能的java虚拟机

​	***TaoBao VM***

### 二、**内存结构**

​	![java虚拟机内存管理图](C:\Users\Administrator\Desktop\JVM\内存管理.png)

#### 1.程序计数器

- 程序计数器是一块比较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器
- 程序计数器处于线程独占区
- 如果线程执行的是java方法，这个计数器记录的正在执行的虚拟机字节码指令的地址。如果正在执行的是native方法，这个计数器的值为undefined
- 此区域是唯一一个在java虚拟机规范中没有规定任何OutOfMemoryError情况的区域

#### 2.Java虚拟机栈

- 虚拟机栈描述的是Java方法执行的动态内存模型（为虚拟机执行Java方法服务）

- 栈帧：每个方法执行，都会创建一个栈帧，伴随这方法从创建到执行完成。用于存储局部变量表，操作数栈，动态链接，方法出口等

- 局部变量表：1）存放编译期可知的各种基本数据类型，引用类型，returnAddress类型；2）局部变量表的内存空间在编译期完成分配，当进入一个方法时，这个方法需要在帧分配多少内存时固定的，在方法运行期间是不会改变局部变量表的大小

  *简单模拟栈溢出（当不限制栈的大小，会产生内存溢出）*

  ```java
  public class StackTest {
  
  	public static void main(String[] args) {
  		
  		new StackTest().stack();
  		
  	}
  
  	public void stack() {
  		
  		System.out.println("stack in----");
  		stack();
  	}
  	
  }
  ```

  ![StackOverFlowError](C:\Users\Administrator\Desktop\JVM\StackOverFlowError.png)

#### 3.本地方法栈：

​	为虚拟机执行native方法服务（在HotSpot VM中与虚拟机栈合二为一）

#### 4.Java堆：

​	1）存放对象实例；

​	2）垃圾收集器管理主要区域；

​	3）最大内存分配

#### 5.方法区

- 存储虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码等数据（类的版本、字段、方法、接口）
- 方法区和永久区
- 对常量垃圾回收
- 异常的定义：如在申请内存时，如果不足则抛出OutOfMemoryError

1. 运行时常量池（方法区一部分）：如String str = new String("abc");   str.intern();（对于new的对象会直接在Java堆中分配新的内存地址及实例，如果直接用常量赋值，则在方法区中的常量池中进行常量分配）

2. 直接内存：基本由NIO使用

---

#### 6.对象

1. **对象的创建**

   只有new对象与调用对象的<init>方法可以检测到，中间的步骤都是虚拟机内部执行

   ![对象的创建](C:\Users\Administrator\Desktop\JVM\对象的创建.png)

   给对象分配内存：

      	1.	指针碰撞（堆内存按空闲内存指针定位分配）
      	2.	空闲列表	（将分散的空闲空间以表的方式记录，在分配时进行查找使用）

   在对象创建的内存分配时会存在线程问题

   1. 线程同步
   2. 本地线程分配缓存（TLAB）（在堆中为不同的线程进行内存分配，当某个线程空间满时，使用同步策略扩容）

2. **对象的结构**

   1. Header

      - 自身运行时数据（Mark Word）：哈希值；GC分代年龄；锁状态标志；线程持有的锁；偏向线程ID；偏向时间戳
      - 类型指针

      ![对象头](C:\Users\Administrator\Desktop\JVM\对象头.png)

   2. InstanceData（内部的数据存储策略：将宽度相同类型的数据存放在一块）

   3. Padding（填充内存）

3. **对象的访问定位**

   1. 使用句柄

      在堆中会单独有一块存储实例对象句柄的句柄池，当指向对象的引用要获取堆内存中的实例时，会先去句柄池获取句柄，在通过句柄获取对象

   2. 使用指针（HotSpot使用该方式）

      引用直接指向对象堆内存地址

      *无论那种方式，都需要存储到对象实例数据的指针与到对象类型数据的指针*

### **三、垃圾回收机制**

#### 1.判定对象为垃圾对象

##### 	1.引用计数法

​		当对象中添加一个引用计数器，当有地方引用这个对象时，引用计数器的值就+1，当引用失效时，就-1（当内部的对象间仍有相互的引用，该方法无法检测到）

​		以下jvm运行参数，打印垃圾收集情况：

​		*-verbose:gc -XX:+PrintGCDetails*

![GCDetails](C:\Users\Administrator\Desktop\JVM\GCDetails.png)

##### 	2.可达性分析法

​		以GCRoot为起始点，通过相应的引用链进行查询，当某条引用链或被引用的对象内存没有相对应的GCRoot，则进行垃圾判断（大多数虚拟机采用垃圾判断方法）

​		GCRoots：虚拟机栈引用的对象；方法去的类属性引用的对象；方法区中常量引用的对象；本地方法栈中引用的对象

![可达性分析法](C:\Users\Administrator\Desktop\JVM\可达性分析法.png)

#### 2.垃圾回收

##### 	1.回收策略

```java
		|-内存结构细分：
			|-堆
				|-新生代
					|-Eden
					|-Survivor
					|-Tenured Gen
				|-老年代
		|-方法区
		|-栈	-本地方法栈	-程序计数器

```



###### 		1）**标记-清除算法**

​			以下算法的基础，将被判断为垃圾的对象内存进行标记，在回收时清除被标记垃圾内存；但是这种方法会有两个问题-1）空间问题（当回收后空间都是分散的）2）效率问题（空间问题导致为大对象分配内存没有足够连续空间，从而再进行一次垃圾回收）

###### 		2）**复制算法**

​			将原有的内存空间分为两块相同的存储空间，每次只使用一块，在垃圾回收时，将正在使用的内存块中存活对象复制到未使用的那一块内存空间中，之后清除正在使用的内存块中的所有对象，完成垃圾回收。 

​			**浪费空间：**

​			![](C:\Users\Administrator\Desktop\JVM\复制算法.png)

​			在java中的新生代串行垃圾回收器中，使用了复制算法的思想，新生代分为eden空间、from空间和to空间3个部分，其中from和to空间可以看做用于复制的两块大小相同、可互换角色的内存空间块（同一时间只能有一个被当做当前内存空间使用，另一个在垃圾回收时才发挥作用），from和to空间也称为survivor空间，用于存放未被回收的对象。

​			**新生代对象**：存放年轻对象的堆空间，年轻对象指刚刚创建，或者经历垃圾回收次数不多的对象。

​			**老年代对象**：存放老年对象的堆空间。即为经历多次垃圾回收依然存活的对象。

​			*在垃圾回收时，eden空间中存活的对象会被复制到未使用的survivor空间中（图中的to），正在使用的survivor空间（图中的from）中的年轻对象也会被复制到to空间中（大对象或者老年对象会直接进入老年代，如果to空间已满，则对象也会进入老年代）。此时eden和from空间中剩余对象就是垃圾对象，直接清空，to空间则存放此次回收后存活下来的对象。* 

​			**保证了内存空间的连续性，又避免了大量的空间浪费：** 

​	

![复制算法2](C:\Users\Administrator\Desktop\JVM\复制算法2.png)

###### 		3）**标记-整理算法**（标记清除压缩算法 ）

​			复制算法的高效性是建立在存活对象少、垃圾对象多的情况下，这种情况在新生代比较常见，

但是在老年代中，大部分对象都是存活的对象，如果还是有复制算法的话，成本会比较高。因此，基于**老年代**这种特性，应该使用其他的回收算法。

​			将所有的存活对象压缩到内存空间的一端，之后，清理边界外所有的空间。这样做避免的碎片的产生，又不需要两块相同的内存空间，因此性价比高

​			![](C:\Users\Administrator\Desktop\JVM\标记整理.png)

###### 		4）**分代收集算法**

​			将内存空间根据对象的特点不同进行划分，选择合适的垃圾回收算法，以提高垃圾回收的效率。 

​			![](C:\Users\Administrator\Desktop\JVM\分代算法.png)

###### 		5)**分区算法** 

​			将整个堆空间划分为连续的不同小区间， 每一个小区间都独立使用，独立回收。 

​			可以控制一次回收多少个小区间 

​			通常，相同的条件下，堆空间越大，一次GC所需的时间就越长，从而产生的停顿时间就越长。为了更好的控制GC产生的停顿时间，将一块大的内存区域分割成多个小块，根据目标的停顿时间，每次合理的回收若干个小区间，而不是整个堆空间，从而减少一个GC的停顿时间。 

​			![](C:\Users\Administrator\Desktop\JVM\分区算法.png)

##### 	2.垃圾回收器

###### 		1）Serial

​			最早的收集器，发展历史最悠久；属于单线程垃圾收集器；更多对于桌面应用

​			Serial收集器是一个新生代收集器，单线程执行，使用复制算法。它在进行垃圾收集时，它不仅只会使用一个CPU或者一条收集线程去完成垃圾收集作，而且必须暂停其他所有的工作线程(用户线程),直到它收集完成。

​			是Jvm **client模式**下默认的**新生代收集器**。对于限定单个CPU的环境来说，简单高效，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率，因此是运行在Client模式下的虚拟机的不错选择（比如桌面应用场景）。 

​		![](C:\Users\Administrator\Desktop\JVM\serial收集器.jpg) 

---



​		**Serial Old(串行GC)收集器**:Serial Old是Serial收集器的老年代版本，它同样使用一个单线程执行收集，使用“标记-整理”算法。主要使用在Client模式下的虚拟机。 

​		如果在Service模式下使用：1.一种是在JDK1.5以及之前的版本中与Parallel Scavenge收集器搭配使用，因为那时还没有Parallel  Old老年代收集器搭配；2.另一种就是作为CMS收集器的后备预案，在并发收集发生Concurrent Model Failure时使用 

​			![](C:\Users\Administrator\Desktop\JVM\Serial Old.jpg)

###### 		2）Parnew

​			ParNew收集器其实就是serial收集器的多线程版本，使用复制算法。除了使用多条线程进行垃圾收集之外，其余行为与Serial收集器一样。是运行在**Service模式下虚拟机**中首选的**新生代收集器**，其中一个与性能无关的原因就是除了Serial收集器外，目前只有ParNew收集器能与CMS收集器配合工作。

　　		ParNew收集器在单CPU环境中绝对没有Serial的效果好，由于存在线程交互的开销，该收集器在超线程技术实现的双CPU中都不能一定超过Serial收集器。默认开启的垃圾收集器线程数就是CPU数量，可通过-XX：parallelGCThreads参数来限制收集器线程数

​			![](C:\Users\Administrator\Desktop\JVM\ParNew收集器.jpg)

###### 		3）Parallel Scavenge

​			Parallel Scavenge收集器也是一个新生代收集器，它也是使用复制算法的收集器，又是并行多线程收集器。parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而parallel Scavenge收集器的目标则是达到一个可控制的吞吐量。**吞吐量= 程序运行时间/(程序运行时间 + 垃圾收集时间)**，虚拟机总共运行了100分钟。其中垃圾收集花掉1分钟，那吞吐量就是99%。 

​			短停顿时间适合和用户交互的程序，体验好。高吞吐量适合高效利用CPU，主要用于后台运算不需要太多交互。 

​			提供了两个参数来精确控制吞吐量：1.最大垃圾收集器停顿时间（-XX：MaxGCPauseMillis    大于0的毫秒数，停顿时间小了就要牺牲相应的吞吐量和新生代空间），2.设置吞吐量大小（-XX：GCTimeRatio    大于0小于100的整数，默认99，也就是允许最大1%的垃圾回收时间）。

　　		还有一个参数表示自适应调节策略（GC Ergonomics）（-XX：UseAdaptiveSizePolicy）。就不用手动设置新生代大小（-Xmn）、Eden和Survivor区的比例（-XX：SurvivorRatio）今生老年代对象大小（-XX：PretenureSizeThreshold），会根据当前系统的运行情况手机监控信息，动态调整停顿时间和吞吐量大小。也是其与PreNew收集器的一个重要区别，也是其无法与CMS收集器搭配使用的原因（CMS收集器尽可能地缩短垃圾收集时用户线程的停顿时间，以提升交互体验）。

​			![](C:\Users\Administrator\Desktop\JVM\Parallel Scavenge收集器.jpg)

---

​			**Parallel Old(并行GC)收集器:**Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法，JDK1.6才提供。 由于之前有一个Parallel Scavenge新生代收集器，但是却无老年代收集器与之完美结合，只能采用Serial Old老年代收集器，但是由于Serial Old收集器在服务端应用性能上低下（毕竟单线程，多CPU浪费了），其吞吐量反而不一定有ParNew+CMS组合。 

​			![](C:\Users\Administrator\Desktop\JVM\Parallel Old.jpg)

###### 		4）Cms

> CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器。CMS收集器是HotSpot虚拟机中的一款真正意义上的并发收集器，第一次实现了让垃圾回收线程和用户线程（基本上）同时工作。用CMS收集老年代的时候，新生代只能选择Serial或者ParNew收集器。
>
> 　　CMS收集器是基于“标记-清除”算法实现的，整个收集过程大致分为4个步骤：

> > ①.初始标记(CMS initial mark)
>
> > ②.并发标记(CMS concurrenr mark)
>
> > ③.重新标记(CMS remark)
>
> > ④.并发清除(CMS concurrent sweep)
>
> > ​     其中初始标记、重新标记这两个步骤任然需要停顿其他用户线程（Stop The World）。初始标记仅仅只是标记出GC ROOTS能直接关联到的对象，速度很快，并发标记阶段是进行GC ROOTS 根搜索算法阶段，会判定对象是否存活。而重新标记阶段则是为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间会被初始标记阶段稍长，但比并发标记阶段要短。
> >
> > ​     由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以整体来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。
> >
> > CMS收集器的优点：并发收集、低停顿，但是CMS还远远达不到完美，器主要有三个显著缺点：
> >
> > 　　1.CMS收集器对CPU资源非常敏感。在并发（并发标记、并发清除）阶段，虽然不会导致用户线程停顿，但是会占用CPU资源而导致应用程序变慢，总吞吐量下降。CMS默认启动的回收线程数是：(CPU数量+3) / 4。收集器线程所占用的CPU数量为：（CPU+3）/4=0.25+3/（4*CPU）。因此这时垃圾收集器始终不会占用少于25%的CPU，因此当进行并发阶段时，虽然用户线程可以跑，但是很缓慢，特别是双核CPU的时候，已经占用了5/8的CPU，吞吐量会很低。为了解决这种情况，产生了“增量式并发收集器”（Incremental Concurrent Mark Sweep/i-CMS）。就是采用抢占方式来模拟多任务机制，就是在并发（并发标记、并发清除）阶段，让GC线程、用户线程交替执行，尽量减少GC线程独占CPU，这样垃圾收集过程更长，但是对用户程序影响小一些。实际上i-CMS效果很一般，目前已经被声明为“deprecated”。
> >
> > 　　2.CMS收集器无法处理浮动垃圾，可能出现“Concurrent Mode Failure“，失败后而导致另一次Full  GC的产生。由于CMS并发清理阶段用户线程还在运行，伴随程序的运行自热会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法在本次收集中处理它们，只好留待下一次GC时将其清理掉。这一部分垃圾称为“浮动垃圾”。也是由于在垃圾收集阶段用户线程还需要运行，即需要预留足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分内存空间提供并发收集时的程序运作使用。在默认设置下，CMS收集器在老年代使用了68%的空间时就会被激活，也可以通过参数-XX:CMSInitiatingOccupancyFraction的值来提高触发百分比，以降低内存回收次数提高性能。JDK1.6中，CMS收集器的启动阈值已经提升到92%。要是CMS运行期间预留的内存无法满足程序其他线程需要，就会出现“Concurrent Mode Failure”失败，这时候虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。所以说参数-XX:CMSInitiatingOccupancyFraction设置的过高将会很容易导致“Concurrent Mode Failure”失败，性能反而降低。
> >
> > 　　3.最后一个缺点，CMS是基于“标记-清除”算法实现的收集器，使用“标记-清除”算法收集后，会产生大量碎片。空间碎片太多时，将会给对象分配带来很多麻烦，比如说大对象，内存空间找不到连续的空间来分配不得不提前触发一次Full  GC。为了解决这个问题，CMS收集器提供了一个-XX:UseCMSCompactAtFullCollection开关参数，用于在Full  GC之后增加一个内存碎片的合并整理过程，但是内存整理过程是无法并发的，因此解决了空间碎片问题，却使停顿时间变长。还可通过-XX:CMSFullGCBeforeCompaction参数设置执行多少次不压缩的Full  GC之后，跟着来一次碎片整理过程（默认值是0，表示每次进入Full GC时都进行碎片整理）。

​			![](C:\Users\Administrator\Desktop\JVM\CMS收集器.jpg)

###### 		5）G1

​			　G1(Garbage First)收集器是JDK1.7提供的一个新的面向服务端应用的垃圾收集器，其目标就是替换掉JDK1.5发布的CMS收集器。其优点有：

　　		1.并发与并行：G1能充分利用多CPU、多核环境下的硬件优势，使用多个CPU（CPU或CPU核心）来缩短停顿（Stop The World）时间。

　			2.分代收集：G1不需要与其他收集器配合就能独立管理整个GC堆，但他能够采用不同方式去处理新建对象和已经存活了一段时间、熬过多次GC的老年代对象以获取更好收集效果。

　　		3.空间整合：从整体来看是基于“标记-整理”算法实现，从局部（两个Region之间）来看是基于“复制”算法实现的，但是都意味着G1运行期间不会产生内存碎片空间，更健康，遇到大对象时，不会因为没有连续空间而进行下一次GC，甚至一次Full GC。

　　		4.可预测的停顿：降低停顿是G1和CMS共同关注点，但G1除了追求低停顿，还能建立可预测的停顿模型，可以明确地指定在一个长度为M的时间片内，消耗在垃圾收集的时间不超过N毫秒

　　		5.跨代特性：之前的收集器进行收集的范围都是整个新生代或老年代，而G1扩展到整个Java堆(包括新生代，老年代)。

​	***那么是怎么实现的呢？***

　　		1.如何实现新生代和老年代全范围收集：其实它的Java堆布局就不同于其余收集器，它将整个Java堆划分为多个大小相等的独立区域（Region），仍然保留新生代和老年代的概念，可是不是物理隔离的，都是一部分Region（不需要连续）的集合。

　　		2.如何建立可预测的停顿时间模型：是因为有了独立区域Region的存在，就避免在Java堆中进行全区域的垃圾收集，G1跟踪各个Region里面的垃圾堆积的价值大小（回收可以获得的空间大小和回收所需要的时间的经验值），后台维护一个优先队列，根据每次允许的收集时间，优先回收价值最大的Region（Garbage-First理念）。因此使用Region划分内存空间以及有优先级的区域回收方式，保证了有限时间获得尽可能高的收集效率。

　　		3.如何保证垃圾回收真的在Region区域进行而不会扩散到全局：由于Region并不是孤立的，一个Region的对象可以被整个Java堆的任意其余Region的对象所引用，在做可达性判定确定对象是否存活时，仍然会关联到Java堆的任意对象，G1中这种情况特别明显。而以前在别的分代收集里面，新生代规模要比老年代小许多，新生代收集也频繁得多，也会涉及到扫描新生代时也会扫描老年代的情况，相反亦然。解决：G1收集器Region之间的对象引用以及新生代和老年代之间的对象引用，虚拟机都是使用Remembered Set来避免全堆扫描。G1中每个Region都有一个与之对应的Remembered Set,虚拟机发现程序在对Reference类型的数据进行写操作时，会产生一个Write Barrier暂时中断写操作，检查Reference引用的对象是否处于不同的Region之中(分代的例子中就检查是否老年代对象引用了新生代的对象)，如果是则通过CardTable把相关引用信息记录到被引用对象所属的Region的Remembered Set之中，当进行内存回收时，在GC根节点的枚举范围中加入Remembered Set即可避免全堆扫描。

忽略Remembered Set的维护，G1的运行步骤可简单描述为：

> ①.初始标记(Initial Marking)

> ②.并发标记(Concurrenr Marking)

> ③.最终标记(Final Marking)

> ④.筛选回收(Live Data Counting And Evacution)

​	1.初始标记：初始标记仅仅标记GC Roots能直接关联到的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的Region中创建新的对象。这阶段需要停顿线程，不可并行执行，但是时间很短。 

​	2.并发标记：此阶段是从GC Roots开始对堆中对象进行可达性分析，找出存活对象，此阶段时间较长可与用户程序并发执行。

​	3.最终标记：此阶段是为了修正在并发标记期间因为用户线程继续运行而导致标记产生变动的那一份标记记录，虚拟机将这段时间对象变化记录在线程Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这段时间需要停顿线程，但是可并行执行。

​	4.筛选回收：对各个Region的回收价值和成本进行排序，根据用户期望的GC停顿时间来制定回收计划。

　　如果现有的垃圾收集器没有出现任何问题，没有任何理由去选择G1，如果应用追求低停顿，G1可选择，如果追求吞吐量，和Parallel Scavenge/Parallel Old组合相比G1并没有特别的优势。

###### 	垃圾收集器参数总结

​		

> -XX:+<option> 启用选项
>
> -XX:-<option> 不启用选项
>
> -XX:<option>=<number> 
>
> -XX:<option>=<string>

| 参数                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| -XX:+UseSerialGC                   | Jvm运行在Client模式下的默认值，打开此开关后，使用Serial + Serial Old的收集器组合进行内存回收 |
| -XX:+UseParNewGC                   | 打开此开关后，使用ParNew + Serial Old的收集器进行垃圾回收    |
| -XX:+UseConcMarkSweepGC            | 使用ParNew + CMS +  Serial Old的收集器组合进行内存回收，Serial Old作为CMS出现“Concurrent Mode Failure”失败后的后备收集器使用。 |
| -XX:+UseParallelGC                 | Jvm运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge +  Serial Old的收集器组合进行回收 |
| -XX:+UseParallelOldGC              | 使用Parallel Scavenge +  Parallel Old的收集器组合进行回收    |
| -XX:SurvivorRatio                  | 新生代中Eden区域与Survivor区域的容量比值，默认为8，代表Eden:Subrvivor = 8:2,Subrvivor一直为2 |
| -XX:PretenureSizeThreshold         | 直接晋升到老年代对象的大小，设置这个参数后，大于这个参数的对象将直接在老年代分配 |
| -XX:MaxTenuringThreshold           | 晋升到老年代的对象年龄，每次Minor GC之后，年龄就加1，当超过这个参数的值时进入老年代 |
| -XX:UseAdaptiveSizePolicy          | 动态调整java堆中各个区域的大小以及进入老年代的年龄           |
| -XX:+HandlePromotionFailure        | 是否允许新生代收集担保，进行一次minor gc后, 另一块Survivor空间不足时，将直接会在老年代中保留 |
| -XX:ParallelGCThreads              | 设置并行GC进行内存回收的线程数                               |
| -XX:GCTimeRatio                    | GC时间占总时间的比列，默认值为99，即允许1%的GC时间，仅在使用Parallel Scavenge 收集器时有效 |
| -XX:MaxGCPauseMillis               | 设置GC的最大停顿时间，在Parallel Scavenge 收集器下有效       |
| -XX:CMSInitiatingOccupancyFraction | 设置CMS收集器在老年代空间被使用多少后出发垃圾收集，默认值为68%，仅在CMS收集器时有效，-XX:CMSInitiatingOccupancyFraction=70 |
| -XX:+UseCMSCompactAtFullCollection | 由于CMS收集器会产生碎片，此参数设置在垃圾收集器后是否需要一次内存碎片整理过程，仅在CMS收集器时有效 |
| -XX:+CMSFullGCBeforeCompaction     | 设置CMS收集器在进行若干次垃圾收集后再进行一次内存碎片整理过程，通常与UseCMSCompactAtFullCollection参数一起使用 |
| -XX:+UseFastAccessorMethods        | 原始类型优化                                                 |
| -XX:+DisableExplicitGC             | 是否关闭手动System.gc                                        |
| -XX:+CMSParallelRemarkEnabled      | 降低标记停顿                                                 |
| -XX:LargePageSizeInBytes           | 内存页的大小不可设置过大，会影响Perm的大小，-XX:LargePageSizeInBytes=128m |

> Client、Server模式默认GC

|        | 新生代GC方式                  | 老年代和持久**代**GC方式 |
| ------ | ----------------------------- | ------------------------ |
| Client | Serial 串行GC                 | Serial Old 串行GC        |
| Server | Parallel Scavenge  并行回收GC | Parallel Old 并行GC      |

> Sun/Oracle JDK GC组合方式

|                                          | 新生代GC方式                  | 老年代和持久**代**GC方式                                     |
| ---------------------------------------- | ----------------------------- | ------------------------------------------------------------ |
| -XX:+UseSerialGC                         | Serial 串行GC                 | Serial Old 串行GC                                            |
| -XX:+UseParallelGC                       | Parallel Scavenge  并行回收GC | Serial Old  并行GC                                           |
| -XX:+UseConcMarkSweepGC                  | ParNew 并行GC                 | CMS 并发GC  当出现“Concurrent Mode Failure”时 采用Serial Old 串行GC |
| -XX:+UseParNewGC                         | ParNew 并行GC                 | Serial Old 串行GC                                            |
| -XX:+UseParallelOldGC                    | Parallel Scavenge  并行回收GC | Parallel Old 并行GC                                          |
| -XX:+UseConcMarkSweepGC -XX:+UseParNewGC | Serial 串行GC                 | CMS 并发GC  当出现“Concurrent Mode Failure”时 采用Serial Old 串行GC |

#### 3.回收时机

#### 4.内存分配策略

##### 	1.优先分配到Eden

​		大多数情况下，对象在新生代Eden去中分配，但Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC。 

```java
/**
 * @author Administrator
 * Allocation：分配
 *内存分配策略
 *空间分配担保
 *VM参数：-XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
 */
public class AllocationTest {
	
	private static final int _1MB = 1024*1024;
	
	public static void main(String[] args) {
		
		byte[] allocation1,allocation2,allocation3,allocation4;

        allocation1 = new byte[2 * _1MB];
        allocation2 = new byte[2 * _1MB];
        allocation3 = new byte[2 * _1MB];
        allocation4 = new byte[4 * _1MB];
	}
	
}

```

![](C:\Users\Administrator\Desktop\JVM\优先分配Eden.png)

​		**[GC……]**就是GC日志记录，而Heap以下的信息是JVM关闭前的堆使用情况信息描述。

​		*GC代表MinorGC，如果是Full GC的话会直接显示Full GC；[DefNew: 7127K->534k(9216K), 0.0038532 secs]中DefNew代表使用的是代表Serial收集器，7127K->534k(9216K)中7127K代表新生代收集前使用内存，534K代表收集后的使用内存，而9216k代表新生代的总分配内存，0.0038532 secs为新生代收集时间；外层的7127K -> 534k(19456K)代表整个Java堆的收集前使用内存->收集后使用内存（总分配内存），0.0038917 secs为整个GC的收集时间，整串GC日志都是类Json格式。*

​		把JVM设置了不可扩展内存20MB，其中新生代10MB，老年代10MB，而新生代区域的分配比例是8:1:1，使用Serial/Serial Old组合收集器。从代码可以看出，allocation1、allocation2、allocation3一共需要6MB，而Eden一共有8MB，优先分配到Eden。但再分配allocation4的时候Eden空间不够，执行了一次Minor GC，在GC中，由于Survivor只有1MB，不够存放allocation1、allocation2、allocation3，所以直接迁移到老年代了，最后Eden空闲出来了就可以放allocation4了 

##### 	2.大对象直接分配到老年代

​		**大对象**是指需要**大量连续内存空间**的Java对象，最典型的大对象就是那种很长的字符串以及数组（例如上面例子的byte[]数组）。大对象对虚拟机内存分配来说是一个坏消息，经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够的连续空间来“安置”它们。虚拟机提供了一个-XX:PretenureSizeThreshold参数来设置大对象的界限，大于此值则直接分配在老年代 

```java
public class AllocationTest{
	
	 private static final int _1MB = 1024*1024;

	    /**
	     * VM参数：-XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
	     * -XX:PretenureSizeThreshold=4M 或者 -XX:PretenureSizeThreshold=3145728
	     */
	 public static void main(String[] args) {
	        byte[] allocation1;

	        allocation1 = new byte[4 * _1MB];
	    }
	
}
```

![](C:\Users\Administrator\Desktop\JVM\大对象分配.png)

​		allocation1分配的是4MB，达到PretenureSizeThreshold定义的4MB阀值，所以直接分配到老年代，没有GC

##### 	3.长期存活的对象分配到老年代

​		由于Minor GC跟Full GC是差别的，Minor的主要对象还是新生代，对象在Minor后并不都会直接进入老年代，除非Survivor空间不够，否则此存活对象会经过多次Minor GC后还生存的话才进入老年代，而虚拟机默认的Minor GC次数为15次，可通过-XX:MaxTenuringThreshold进行次数设置 

```java
public class AllocationTest{
	
	private static final int _1MB = 1024*1024;
    
    /**
     * VM参数：-XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
     * -XX:MaxTenuringThreshold=15 OR 1
     */
	public static void main(String[] args) {
        byte[] allocation1,allocation2,allocation3;

        allocation1 = new byte[1 * _1MB / 4];
        allocation2 = new byte[4 * _1MB];
        allocation3 = new byte[4 * _1MB];
        allocation3 = null;
        allocation3 = new byte[4 * _1MB];
        
//        System.gc();
    }
	
}

```

![](C:\Users\Administrator\Desktop\JVM\长期存活对象.png)

​		该环境为jdk8.0，无论**-XX:MaxTenuringThreshold=15 OR 1**，输出结果一样；或许内部有改动，但还是看出两次的GC，第二次直接将allocation1分配到老年代

##### 	4.**动态对象年龄判定**

​		为了能更好地适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringThreshold才能晋升老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或者等于该年龄的对象直接可以进入老年代，无须等到MaxTenuringThreshold中要求的年龄 

```java
public class AllocationTest{
	
	 private static final int _1MB = 1024*1024;

	    /**
	     * VM参数：-XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
	     */
	 public static void main(String[] args) {
	        byte[] allocation1,allocation2,allocation3,allocation4;

	        allocation1 = new byte[1 * _1MB / 4];
	        allocation2 = new byte[2 * _1MB / 4];//注释前后对比
	        allocation3 = new byte[4 * _1MB];
	        allocation4 = new byte[4 * _1MB];
	        allocation3 = null;
	        allocation4 = new byte[4 * _1MB];
	    }
}

```

![](C:\Users\Administrator\Desktop\JVM\年龄代分配.png)

​		由于jdk8.0的改进，两种情况输出一致

##### 	5.空间分配担保

		>​		在发生Minor GC前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象的空间，如果这个条件成立，那么Minor GC可以确保安全的。如果不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。如果允许，那么会检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次Minor GC，尽管这个Minor GC是有风险的；如果小于或者HandlePromotionFailure设置不允许冒险，那么这时也要改为进行一次Full GC了。说白了就是虚拟机避免Full GC执行的次数而去做的检查机制。
		>
		> 
		>
		>​		取平均值进行比较其实仍然是一种动态概率的手段，也就是说，如果某次Minor GC存活后的对象突增，远远高于平均值的话，依然会导致担保失败（Handle Promotion Failure）。如果出现了HandlePromotionFailure失败，那就只好在失败后重新发起一次Full GC。虽然担保失败时绕的圈子是最大的，但大部分情况下都还是会将HandlePromotionFailure开关打开，避免Full GC过于频繁。
		>
		> 
		>
		>​		另外需要提醒，在JDK 6 Update 24之后，虚拟机已经不再使用HandlePromotionFailure参数了，规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC，否则将进行Full GC

##### 6.逃逸分析与栈上分配

​	  栈上分配主要是指在Java程序的执行过程中，在方法体中声明的变量以及创建的对象，将直接从该线程所使用的栈中分配空间。 一般而言，创建对象都是从堆中来分配的，这里是指在栈上来分配空间给新创建的对象。 

​	 逃逸是指在某个方法之内创建的对象，除了在方法体之内被引用之外，还在方法体之外被其它变量引用到；这样带来的后果是在该方法执行完毕之后，该方法中创建的对象将无法被GC回收，由于其被其它变量引用。正常的方法调用中，方法体中创建的对象将在执行完毕之后，将回收其中创建的对象；故由于无法回收，即成为逃逸。

​	 在JDK 6之后支持对象的栈上分析和逃逸分析，在JDK 7中完全支持栈上分配对象。 其是否打开逃逸分析依赖于以下JVM的设置：

```java
-XX:+DoEscapeAnalysis
```

​	进行逃逸分析之后，产生的后果是所有的对象都将由栈上分配，而非从JVM内存模型中的堆来分配。 

​	优势表现在以下两个方面：

​		消除同步：线程同步的代价是相当高的，同步的后果是降低并发性和性能。逃逸分析可以判断出某个对象是否始终只被一个线程访问，如果只被一个线程访问，那么对该对象的同步操作就可以转化成没有同步保护的操作，这样就能大大提高并发程度和性能。
 		矢量替代：逃逸分析方法如果发现对象的内存存储结构不需要连续进行的话，就可以将对象的部分甚至全部都保存在CPU寄存器内，这样能大大提高访问速度。

      	劣势：  栈上分配受限于栈的空间大小，一般自我迭代类的需求以及大的对象空间需求操作，将导致栈的内存溢出；故只适用于一定范围之内的内存范围

### **四、性能监控工具**

#### 1.jps

​	java process status

​	java编译环境运行，产生本地虚拟机唯一id（local virtual machine id），没有具体应用名

![](C:\Users\Administrator\Desktop\JVM\jps.png)

#### 2.jstat

​	监控类加载，内存，垃圾收集i，jit编译的信息（与jps协作使用）

`https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html`

![](C:\Users\Administrator\Desktop\JVM\jstat.png)

![](C:\Users\Administrator\Desktop\JVM\jstat -gc.png)

​	元空间：本质与永久代类似，都是对jvm规范中方法区的实现	。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间大小仅受本地内存限制

#### 3.jinfo

​	实时查看和调整虚拟机的各项参数

​	![](C:\Users\Administrator\Desktop\JVM\jinfo.png)

#### 4.jmap

​	查看堆快照信息

​	`jmap -dump:format=b,file=e:\a.bin 10688`	将快照信息转储到指定文件中

![](C:\Users\Administrator\Desktop\JVM\jmap.png)

#### 5.jhat

​	JVM Heap Analysis Tool

​	堆jmap生成的快照文件进行分析，并会以http服务方式，提供端口查看具体分析信息，该工具在分析时会很占内存与CPU

![](C:\Users\Administrator\Desktop\JVM\jhat.png)

​	以localhost:7000访问

![](C:\Users\Administrator\Desktop\JVM\jhat-2.png)				

​	通过OQL对分析的堆内对象进行查询![](C:\Users\Administrator\Desktop\JVM\oql.png)

#### 6.jstack

​	生成当前虚拟机的线程快照

​	线程快照：当前虚拟机内，每一条线程执行的方法堆栈集合

![](C:\Users\Administrator\Desktop\JVM\jstack.png)

​	通过以下代码可实现类似功能

```java
public class StackTrace {

	public static void main(String[] args) {
		
		Map<Thread, StackTraceElement[]> map = Thread.getAllStackTraces();
		for(Map.Entry<Thread, StackTraceElement[]> entry : map.entrySet()) {
			
			Thread thread = entry.getKey();
			StackTraceElement[] vElements = entry.getValue();
			
			System.out.println("Thread Name Is:"+thread.getName());
			for(StackTraceElement sElement : vElements) {
				System.out.println("\t"+sElement.toString());
			}
		}
		
		
	}
	
}
```

![](C:\Users\Administrator\Desktop\JVM\stacktrace.png)

#### 7.jconsole

​	内存监控

```java
List<Object> list = new ArrayList<>();
		while (true) {
			Thread.sleep(500);
			list.add(new Object());
		}
```

![](C:\Users\Administrator\Desktop\JVM\jconsole_1.png)

​	线程监控

```java
		Scanner scanner = new Scanner(System.in);
		scanner.next();
		
		new Thread(()->{
			while (true) {
				
			}
		},"thread-1").start();
		
		Object object = new Object();
		
		Thread thread = new Thread(new Runnable() {
			
			@Override
			public void run() {
				
					
					
					try {
						wait();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				
				
			}
		},"thread-2");
		
		thread.start();
```

![](C:\Users\Administrator\Desktop\JVM\jconsol_2.png)

​	死锁

```java
public class Jconsole_3 {
	
	public static void main(String[] args) {
		
		Object obj1 = new Object();
		Object obj2 = new Object();
		
		new Thread(new DeadLock(obj1, obj2),"lock1").start();
		new Thread(new DeadLock(obj2,obj1),"lock2").start();
		
	}
	
	
	
}

class DeadLock implements Runnable{

	private Object obj1;
	private Object obj2;
	
	public DeadLock(Object obj1,Object obj2) {
		
		this.obj1 = obj1;
		this.obj2 = obj2;
		
	}

	
	@Override
	public void run() {
		synchronized (obj1) {
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			synchronized (obj2) {
				System.out.println("success");
			}
		}
		
	}
	
}
```

![](C:\Users\Administrator\Desktop\JVM\jconsole_2.png)

#### 8.VisualVM

![](C:\Users\Administrator\Desktop\JVM\jvisualVM.PNG)

### 五、**性能调优案例**

**1.大对象多导致full gc，gc时间长，应用出现暂时卡顿**

​	通过减小指定的老年代大小，同时使用服务器集群

**2.nio会对jvm堆外内存进行申请，当堆外内存不够时，导致内存溢出**

​	减小jvm内存或增大机器内存

**3.当有java应用在接受数据时，数据传输太快，导致大量数据堆积得不到处理，从而在连接时出现连接重置，导致jvm崩溃**

​	在两边数据不对等的情况（该情况），在传输的中间增加消息队列，在数据发送来时先进入消息队列，在数据处理时去消息队列获取即可

### 六、**认识类的文件结构**

​	Class文件时一组以8位字节位基础单位的二进制流，各个数据项目严格按照顺序紧凑的排列在class文件中，中间没有添加人格分隔符，整个class文件中存储的内容几乎全部是程序运行的必要数据，没有空隙纯在

​	当遇到8位字节以上的空间的数据项时，则会按照高位在前的方式分割成若干个8位字节进行存储

​	class文件中有两种数据结构，分别时无符号数和表

#### 1.Class文件结构

| 类型类型        | 名称                | 数量                  | 说明                                                         |
| --------------- | ------------------- | --------------------- | ------------------------------------------------------------ |
| u4              | magic               | 1                     | 魔数：确定一个文件是否是Class文件                            |
| u2              | minor_version       | 1                     | Class文件的次版本号                                          |
| u2              | major_version       | 1                     | Class文件的主版本号：一个JVM实例只能支持特定范围内版本号的Class文件（可以向下兼容）。 |
| u2              | constant_pool_count | 1                     | 常量表数量                                                   |
| cp_info         | constant_pool       | constant_pool_count-1 | 常量池:以理解为Class文件的资源仓库，后面的其他数据项可以引用常量池内容。 |
| u2              | access_flags        | 1                     | 类的访问标志信息：用于表示这个类或者接口的访问权限及基础属性。 |
| u2              | this_class          | 1                     | 指向当前类的常量索引：用来确定这个类的的全限定名。           |
| u2              | super_class         | 1                     | 指向父类的常量的索引：用来确定这个类的父类的全限定名。       |
| u2              | interfaces_count    | 1                     | 接口的数量                                                   |
| u2              | interfaces          | interfaces_count      | 指向接口的常量索引：用来描述这个类实现了哪些接口。           |
| u2              | fields_count        | 1                     | 字段表数量                                                   |
| field_info      | fields              | fields_count          | 字段表集合：描述当前类或接口声明的所有字段。                 |
| u2              | methods_count       | 1                     | 方法表数量                                                   |
| method_info     | methods             | methods_count         | 方法表集合：只描述当前类或接口中声明的方法，不包括从父类或父接口继承的方法。 |
| u2              | attributes_count    | 1                     | 属性表数量                                                   |
| attributes_info | attributes          | attributes_count      | 属性表集合：用于描述某些场景专有的信息，如字节码的指令信息等等。 |

> ​	以下分析的结构，参照该表中分配的字节数，如u2为两个字节

##### 魔数(magic)

​	class文件以魔数开头，通过魔数可判断当前文件是否为class，魔数后有可以判断jdk版本的major_version

![](C:\Users\Administrator\Desktop\JVM\魔数.png)

![](C:\Users\Administrator\Desktop\JVM\魔数-jdk.png)

##### 常量池(constant_pool)

![](C:\Users\Administrator\Desktop\JVM\常量池.png)

​	紧接着主次版本号就是常量池了，第一个是常量池数量(占两个字节)，接下来就是常量池表，通过cp_info，确定指向的常量，通过以下常量项目表来分析常量池位置大小

![](C:\Users\Administrator\Desktop\JVM\常量项目类型.png)

​	通过javap更直观的查看整个class文件中的常量池

​	![](C:\Users\Administrator\Desktop\JVM\javap.png)

##### 访问标志（access_flags）

​	用于表示这个类或者接口的访问权限及基础属性

​	**字节码文件：**

![](C:\Users\Administrator\Desktop\JVM\flags.png)

​	**javap -verbose :**

![](C:\Users\Administrator\Desktop\JVM\flags2.png)

​	**具体含义：**

| 标志名         | 标志值 | 标志含义                  | 针对的对像 |
| -------------- | ------ | ------------------------- | ---------- |
| ACC_PUBLIC     | 0x0001 | public类型                | 所有类型   |
| ACC_FINAL      | 0x0010 | final类型                 | 类         |
| ACC_SUPER      | 0x0020 | 使用新的invokespecial语义 | 类和接口   |
| ACC_INTERFACE  | 0x0200 | 接口类型                  | 接口       |
| ACC_ABSTRACT   | 0x0400 | 抽象类型                  | 类和接口   |
| ACC_SYNTHETIC  | 0x1000 | 该类不由用户代码生成      | 所有类型   |
| ACC_ANNOTATION | 0x2000 | 注解类型                  | 注解       |
| ACC_ENUM       | 0x4000 | 枚举类型                  | 枚举       |

![](C:\Users\Administrator\Desktop\JVM\access_flags.png)

##### 类索引

​	位于访问标志后，先是指向常量池中当前类索引，然后指向继承的父类索引，再是接口的数量，最后是每个接口对应的索引，该demo中没有接口

​	![](C:\Users\Administrator\Desktop\JVM\类索引.png)

##### 字段表集合

​	class文件：

​		![](C:\Users\Administrator\Desktop\JVM\字段表集合.png)

​	javap -verbose

![](C:\Users\Administrator\Desktop\JVM\字段表集合2.png)

​	字段表的结构： 字段修饰符放在access_flags中，它与类中的access_flags相似，都是一个u2的数据类型

| 类型           | 名称             | 数量             |
| -------------- | ---------------- | ---------------- |
| ｕ2            | access_flags     | 1                |
| ｕ2            | name_index       | 1                |
| ｕ2            | descriptor_index | 1                |
| ｕ2            | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

　字段访问标志： 

| 标志名称      | 标志值  | 含义                       |
| ------------- | ------- | -------------------------- |
| ACC_PUBLIC    | 0x00 01 | 字段是否为public           |
| ACC_PRIVATE   | 0x00 02 | 字段是否为private          |
| ACC_PROTECTED | 0x00 04 | 字段是否为protected        |
| ACC_ENUM      | 0x40 00 | 字段是否为enum             |
| ACC_STATIC    | 0x00 08 | 字段是否为static           |
| ACC_FINAL     | 0x00 10 | 字段是否为final            |
| ACC_VOLATILE  | 0x00 40 | 字段是否为volatile         |
| ACC_TRANSTENT | 0x00 80 | 字段是否为transient        |
| ACC_SYNCHETIC | 0x10 00 | 字段是否为由编译器自动产生 |

​	描述符标志含义： 

| 标志符 | 含义                |
| ------ | ------------------- |
| B      | 基本数据类型byte    |
| C      | 基本数据类型char    |
| D      | 基本数据类型double  |
| F      | 基本数据类型float   |
| I      | 基本数据类型int     |
| J      | 基本数据类型long    |
| S      | 基本数据类型short   |
| Z      | 基本数据类型boolean |
| V      | 基本数据类型void    |
| L      | 对象类型            |

##### 方法表集合

​	表结构与字段表一致

![](C:\Users\Administrator\Desktop\JVM\方法表集合.png)

##### 属性表集合

​	 在Class文件，字段表，方法表中都可以携带自己的属性表集合，以用于描述某些场景专有的信息。与Class文件中其它的数据项目要求的顺序、长度和内容不同，属性表集合的限制稍微宽松一些，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息，Java虚拟机运行时会忽略掉它不认识的属性 

​	对于每个属性，他的名称需要从常量池中引用一个CONSTANT_Utf8_info类型的常量来表示，而属性的结构则是完全自定义的，只需要通过一个u4的长度属性去说明属性值所占用的位数即可 

​	属性表结构：

| 类型 | 名称                 | 数量             |
| ---- | -------------------- | ---------------- |
| u2   | attribute_name_index | 1                |
| u4   | attribute_length     | 1                |
| u1   | info                 | attribute_length |

​	具体的info属性参照：[Java虚拟机规范](https://pan.baidu.com/s/1gG52p5qZw-F71vST-oyKcw)

#### 2.字节码指令

​	Java虚拟机的指令由一个字符长度的，代表着某种特定操作含义的数字，称之为操作码，以及跟随其后的0至多个代表此操作所需参数的操作数而构成。

​	操作吗的长度为1个字节，因此最大只有256条。

​	此外还有基于栈的指令集架构

##### 字节码与数据类型

​	在虚拟机的指令集中，大多数的指令都包含了其操作所对应数据类型信息；大多数指令是包含类型信息的；不包含类型信息的如：Goto与类型无关，ArrayLength操作数组类型；由于类型多，不会所以都有对应的指令，会通过类型转换指令来解决。

​	基本上指令的第一个字母表示数据类型

![](C:\Users\Administrator\Desktop\JVM\指令集数据类型.png)

##### 加载指令

​	加载和存储指令用于将数据子在栈帧中的局部变量表和操作数栈之间来回传输

​	将局部变量表加载到操作数栈：iload，lload，fload，dload，aload(引用）...

​	将一个数值从操作数栈存储到局部变量表：istore，	lfda...

​	将一个常量加载到操作数栈：bipush，sipush，ldc，ldc_w，ldc2_w，aconst_null，iconst_m1，iconst...

​	扩展局部变量表的访问索引的指令：wide

![](C:\Users\Administrator\Desktop\JVM\加载指令.png)

```java
public class demo{
	public int add(int a,int b){
	   int c = a+b;
	   return 1+1;	
	}
}
```

##### 运算指令

​	运算或算术指令用于对两个操作数栈上的值进行某种特定的运算，并把结果存储到操作数栈顶

​	加：add ，减：sub，乘：mul，除：div，取余：rem，取反：neg

​	操作数的栈的深度为2，当多个数同时运算时，会先将局部变量表中的前两个数据加载到操作数栈进行运算，结果返回到操作数栈，再去局部变量表取下一个数据进操作栈进行运算，以此类推。 

##### 类型转换指令

​	类型转换指令可以将两种不同的数值类型进行相互转换，这些转换操作一般用于实现用户代码中的显示类型转换操作，或用于出来字节码指令集中数据类型相关指令无法与数据类型一一对应的问题

```java
public int add(int a,int b){
	   int hour = 24;
	 	long mi = hour*60*60*1000;
		long mic = hour*60*60*1000*1000;
		System.out.println(mic/mi);
	   return 1+1;	
	}
```

i2l：int转换为long，以上代码问题为在int运算时越界，再将结果转换为long	

​	![ ](C:\Users\Administrator\Desktop\JVM\类型转换指令.png)

##### 对象创建与访问指令

​	创建类实例的指令：new

​	创建数组的指令：newarray ，anewarray，multianewarray

​	访问类字段：getfield，putfield，getstatic，putstatic

​	把数组元素加载到操作数栈的指令：baload（c/s/i/l/d/a）

​	将操作数栈的值存储到数组元素：astore

​	取数组长度的指令：arraylength

​	检查实例类型的指令： instanceof，checkcast

```java
public class demo{
	public static void main(String[] args){
		User user = new User();
		User users[] = new User[10];

		int[] arr = new int[10]; 

		user.name = "lov";

		String name = user.name;
	}
}
class User{
		String name ;
		static int age;
}
```

![](C:\Users\Administrator\Desktop\JVM\对象创建指令.png)

##### 操作数栈管理指令

​	操作数栈指令用于直接操作操作数栈

​	将操作数栈一个或两个元素出栈： pop，pop2

​	复制栈顶一个或两个数值并将复制或双份复制值重新压入栈顶：dup，dup2，dup_x1，dup_x2

​	将栈顶的两个数值替换：swap

##### 控制转移指令

​	控制转移指令可以让java虚拟机有条件或无条件的从指定的位置指令而不是控制转移指令的下一条指令继续执行程序，可以认为控制转移指令就是在修改pc寄存器的值

​	条件分支：if_eq，if_lt，if_le，if_ne，if_gt，if_null，if_cmple

​	复合条件分支：tableswitch，lookupswitch

​	无条件分支：goto，goto_w，jsr，jsr_w，ret

```java
int a = 1;
		if(a>1){
			System.out.println("true");
		}else{
			System.out.println("false");
		}
```

![](C:\Users\Administrator\Desktop\JVM\控制转移指令.png)

##### 方法调用与返回指令

​	**方法调用**

​	invokevitrual：调用对象的实例方法，根据对象的实际类型进行分配（虚方法分配），这也是java语言中最常见的方法分派方式

​	invokeinterface：用于调用接口方法，会在运行时搜索一个实现了这个接口方法的对象，找出适合的方法进行调用

​	invokespecial：用于调用一些需要特殊处理的实例方法，包括实例初始化方法、室友方法、父类方法

​	invokestatic：用于调用类方法（static）

​	**方法返回**

​	方法调用的指令与数据类型无关，而方法返回指令则时根据返回值的类型区分，包括ireturn（当返回值为boolean、byte、char、short、int），lreturn，freturn，dreturn，areturn，此外还有一条return指令供声明为void的方法，实例初始化方法，类和接口的类初始化方法使用

```java
public class demo{
	public static void main(String[] args){
		
		inf interf = new infImpl();

		int res = interf.add(1,2);

	}

	public int add2	 (int a,int b){

		return a+b;
	}
}

interface inf{
	int add(int a,int b);
}

class infImpl implements inf{
	
	public int add (int a,int b){

		return a+b;
	}
}
```

![](C:\Users\Administrator\Desktop\JVM\方法指令.png)	

##### 异常处理指令

​	在程序中显示抛出异常的操作会由athrow指令实现，除了这种情况，还有别的异常会在其他java虚拟机实例检测到异常状况时由虚拟机自动抛出

```java
try{
			int i = 1/0;
		}catch(Exception e){
			int b =0;
		}

		int c = 1/0;

		throw new RuntimeException("error");
```

![](C:\Users\Administrator\Desktop\JVM\异常指令.png)

##### 同步指令

​	 Java 虚拟机可以支持方法级的同步和方法内部一段指令序列的同步，这两种同步结构都是使用管程（Monitor）来支持的。

​        方法级的同步是隐式的，即无须通过字节码指令来控制，它实现在方法调用和返回操作之中。虚拟机可以从方法常量池的方法表结构中的 ACC_SYNCHRONIZED 方法标志得知一个方法是否声明为同步方法。当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程就要求先成功持有管程，然后才能执行方法，最后当方法完成（无论是正常完成还是非正常完成）时释放管程。在方法执行期间，执行线程持有了管程，其他任何线程都无法再获取到同一个管程。如果一个同步方法执行期间抛出了异常，并且在方法内部无法处理此异常，那么这个同步方法所持有的管程将在异常抛到同步方法之外时自动释放。

​        同步一段指令集序列通常是由 Java 语言中的 synchronized 语句块来表示的，Java 虚拟机的指令集中有 monitorenter 和 monitorexit 两条指令来支持 synchronized 关键字的语义，正确实现 synchronized 关键字需要 javac 编译器与 Java 虚拟机两者共同协作支持

```java
synchronized(demo.class){
			add(1,2);
		}
```

![](C:\Users\Administrator\Desktop\JVM\同步指令.png)

#### 3.

### 七、**类加载机制**

#### 1.

#### 2.

#### 3.

### **八、字节码执行引擎**

#### 1.

#### 2.

#### 3.

### **九、虚拟机编译及运行时优化**

#### 1.

#### 2.

#### 3.

### **十、java线程高级**

#### 1.

#### 2.

#### 3.