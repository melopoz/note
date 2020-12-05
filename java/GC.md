方法区的两种实现：(方法去只是一个逻辑概念，类似接口，1.7的实现是老年代；1.8的实现是元数据区)

​	老年代 tenured：由JVM管理，必须设置大小。是常量的存储位置

​	元数据区 matespace：直接归操作系统管理。只受限于物理内存。是常量的存储位置

​		ps:元数据 描述事物具有什么信息的数据。也就是类。Class对象。

老年代在堆空间中，必须指定他的大小。

元数据区不在堆空间中，不用指定大小。它不受JVM管理。直接由操作系统管理。只受限于物理内存。

堆空间逻辑分区

![GC-堆内存逻辑分区](images\GC-堆内存逻辑分区.png)



![image-20200319200908704](images\image-20200319200908704.png)



GCroots 根可达算法：从main种的根对象找他引用的所有对象 能找到就不是垃圾



GC的三种算法

Mark-Sweep 标记清除。会产生内存碎片

Coping 复制：只用一半，将A部分钟有用的复制到B，把A清空。

Mark-Compact：标记整理，效率偏低，但是对创建新对象很友好。优化就是优化这个整理的算法

从jdk 1.0到14 所有垃圾回收器

![](C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200319211208873.png)

serial+serialOld已经没了。

jdk1.8默认  parallelGC   

调优时可能使用：ParNew + CMS

1.8可使用G1

1.9版本把CMS移除了

![image-20200319211349862](C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200319211349862.png)

前六种分代 

G1 概念分代 最下边笔记

ZGC Shenandoah 不分代  采用颜色标记    能管理4T的原因： 2^42=4T 

​	用42位代表一个对象。



配合使用。比如： Young使用Serial，Old使用CMS。看连线

serial 连续的 单线程的   Serial 能处理几十兆内存 太小了 所以现在不用了。

parallel 平行的 多线程的  几个G

CMS 几十个G  但是没有jdk默认使用CMS     关键词 三色标记(会产生漏标)  所以最后必须有个ReMark！

​									黄箭头就是SWT

![image-20200319204630528](C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200319204630528.png)

STW 停止其他工作，stop-the-world。这个使用在老年代

parallel scavenge:复制使用多个GC线程的收集器

CMS 可以不停止工作线程进行垃圾清除 （工作在老年代）

![image-20200319205206684](C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200319205206684.png)

场景1-浮动垃圾：gc线程标记上一个对象说他不是垃圾。同时工作线程给那个变量赋值null了，那个对象就变成垃圾了。      情况不严重。下次再清。

场景2-标记失误：gc线程把一个线程当成垃圾没有给标记。准备给清掉了。这就灾难性了。所以会有重新标记 reMark。

重新标记(reMark)的时候使用STW。不能出错。

标记之后并发清理。因为标记了所以不能有变量指向那些垃圾对象。因为是垃圾。



调优是调 gc时间

心得：

如果真的要优化，可以在gc过程中对实际需求做手脚。比如在并发标记和重新标记时。





# G1     Garbage First  优先回收垃圾最多的区

号称支持上百G内存的gc

ZGC  4T   不分代     和C4学的 

Shenandoah     千G   Zing 就一个参数

#### 分区回收   以前使用分代          Region:地区

G1有FullGC 有STW

![image-20200319212153011](C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200319212153011.png)

G1方案：SATB 其实的时候做一个快照，

### 三色标记算法     			黑色 灰色 白色

<img src="C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200324204256335.png" alt="image-20200324204256335" style="zoom:25%;" />

1. 场景1

   ​	如果gc线程将B处理完(B有个属性使用着D引用)，然后工作线程把这个引用取消了(B.setD(null);)

   ​	这种情况不严重。下次gc还会扫描b(因为B还是灰色)。

2. 场景2

   如果在场景1之后，工作线程给A添加了引用指向D。（A.setD(D);）

   这就危险了。可能还有用的D对象会被回收。

   ### 解决方案：

   ##### CMS：Incremental Update

   ​	把A标成灰色。(还有问题。还是并发问题...  多个GC线程操作了同一个对象... 如下图,A变成灰色之后还是被错误的标记成黑色)			所以!!! CMS最后有个STW的remark

   <img src="C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200324205046890.png" alt="image-20200324205046890" style="zoom:25%;" />

   ##### G1:	SATB Snapshot At the Begining

   ​			把B中指向D的引用推到GC的堆栈。下次扫描的时候会将堆栈扫描一遍。

   ​							<img src="C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200324205539388.png" alt="image-20200324205539388" style="zoom:25%;" />

   ​		RSet：rememberSet   

   <img src="C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200325101908712.png" alt="image-20200325101908712" style="zoom:25%;" />

   ​	   CSet：Collection Set

   ​			CSet记录的是GC要收集的Region的集合，CSet里的Region可以是任意代的。在GC的时候，对于old->young和old->old的跨代对象引用，只要扫描对应的CSet中的RSet即可。







# 传统分代GC

![image-20200319202756695](C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200319202756695.png)

新生代  采用复制 效率比较高 因为eden好多对象都是朝生夕死的，创建不久就成了垃圾。一个一个清除效率太低了，所以直接复制有用的到survivor，把eden全部内存擦除



老年代是满了才进行Old GC(FullGC)，

存活多久才进入tenured

PSPO 15

CMS 6

G1 15

EGC 没有，因为不分代

各种名称。。。。

YoungGC	YGC	MinorGC

FullGC		 FGC	MajorGC



如果能在栈空间分配就很理想。  对象就不会有gc之类的巴拉巴拉

![](C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200319204050605.png)

TLAB 属于Eden，如果Y 就会有属于当前线程的一块内存。如果N，就会去抢占内存。





![image-20200319215822908](C:\Users\melopoz\AppData\Roaming\Typora\typora-user-images\image-20200319215822908.png)

确定最大最小内存  让jvm内存不在扩大缩小。也会提升性能。(如果能够确定多大内存够用)









safepoint 安全点   在STW要求停止其他工作线程时 会找一个合适的点不能是比如一个线程在拿锁拿了一半的时候。



post

屏障

任何一次i回收再挪动对象()之后会更新





压缩指针 32G 会失效



java对象8字节对齐 能被8整除！  0-7中的数肯定不是java对象的开头。内存中  只有8的倍数-8的倍数 才会是一个java对象





查看GC日志

找到出问题的占内存的部分

查看那些对象