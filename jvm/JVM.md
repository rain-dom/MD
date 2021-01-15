## 基础知识

jvm是一种规范，它定义了java虚拟机能够执行什么，应该具备哪些模块，遇到不同指令应该做什么

任何语言只要能编译成class，符合class规范，就可以在java虚拟机上运行



### 编译到执行

通过javac编译成class文件，jvm才可读
调用java命令的时候被装载到内存classloader

![编译到执行](E:\deng\deng\MD\jvm\img\编译到执行.png)



任何语言只要能编译成class文件，就可以在jvm运行

jvm只与class文件有关，而与java是无关的
jvm是一种规范
虚构出来的计算机

---

解释和编译是混合的，执行次数多的部分会进行编译

### JDK JRE JVM 

jvm是最后来执行的

![JDKJREJVM](E:\deng\deng\MD\jvm\img\JDKJREJVM.png)

### class文件内容

整个class文件格式就是一个二进制字节流

## class执行过程

一个硬盘里的class文件是如何到内存中准备好执行的呢

![加载过程](E:\deng\deng\MD\jvm\img\加载过程.png)

### 类加载-初始化

1. 加载过程

   1. Loading

      1. 双亲委派，主要出于安全来考虑
         jvm所有的class都是类加载器加载到内存的，也就是classloader。

         class被load到内存后创建了两块内容，第一块是把String.class二进制扔到内存中，第二块是生成一个class类的对象，这个对象指向第一块内容

      2. LazyLoading 五种情况

         > jvm规范并没规定合适加载，但严格规定了什么时候必须初始化

         1. –new getstatic putstatic invokestatic指令，访问final变量除外

            –java.lang.reflect对类进行反射调用时

            –初始化子类的时候，父类首先初始化

            –虚拟机启动时，被执行的主类必须初始化

            –动态语言支持java.lang.invoke.MethodHandle解析的结果为REF_getstatic REF_putstatic REF_invokestatic的方法句柄时，该类必须初始化

      3. ClassLoader的源码

         1. findInCache -> parent.loadClass -> findClass()

      4. 自定义类加载器

         1. extends ClassLoader
         2. overwrite findClass() -> defineClass(byte[] -> Class clazz)
         3. 加密
         4. <font color=red>第一节课遗留问题：parent是如何指定的，打破双亲委派，学生问题桌面图片</font>
            1. 用super(parent)指定
            2. 双亲委派的打破
               1. 如何打破：重写loadClass（）
               2. 何时打破过？
                  1. JDK1.2之前，自定义ClassLoader都必须重写loadClass()
                  2. ThreadContextClassLoader可以实现基础类调用实现类代码，通过thread.setContextClassLoader指定
                  3. 热启动，热部署
                     1. osgi tomcat 都有自己的模块指定classloader（可以加载同一类库的不同版本）

      5. ==混合==执行 编译执行 解释执行

         1. 检测热点代码：-XX:CompileThreshold = 10000、

         > -Xmixed 默认混合模式
         > -Xint 使用解释模式，启动快，执行稍慢
         > -Xcomp 使用纯编译模式，启动慢，执行很快

   2. Linking 

      1. Verification
         1. 验证文件是否符合JVM规定、不符合的话就不进行下一步
      2. Preparation
         1. 给静态成员变量赋默认值
      3. Resolution
         1. 将类、方法、属性等符号引用解析为直接引用
            常量池中的各种符号引用解析为指针、偏移量等内存地址的直接引用

   3. Initializing

      1. 调用类初始化代码 <clinit>，给静态成员变量赋初始值

2. 小总结：

   1. load - 默认值 - 初始值

      > 申请内存后先是会先赋予默认值,再通过类构造方法赋予初始值

   2. new - 申请内存 - 默认值 - 初始值

      > new 不是原子操作，申请内存后先是会先赋予默认值，然后INSTANCE可能已经指向内存了，此时它已经不等于空了。这时另外一个线程过来判断它不为空，可能就直接用了默认值

## 双亲委派机制

1.class loader收到类加载的请求，向上委托给父类加载器完成，直到boot

application --> extension --> bootStrap

如果上面两个都没有此类才会执行application的

### 什么是双亲委派机制

当某个类加载器需要加载某个`.class`文件时，它首先把这个任务委托给他的上级类加载器，递归这个操作，如果上级的类加载器没有加载，自己才会去加载这个类。

### 类加载器的类别

#### BootstrapClassLoader（启动类加载器）

`c++`编写，加载`java`核心库 `java.*`,构造`ExtClassLoader`和`AppClassLoader`。由于引导类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到启动类加载器的引用，所以不允许直接通过引用进行操作

#### ExtClassLoader （标准扩展类加载器）

`java`编写，加载扩展库，如`classpath`中的`jre` ，`javax.*`或者
 `java.ext.dir` 指定位置中的类，开发者可以直接使用标准扩展类加载器。

#### AppClassLoader（系统类加载器）

```
java`编写，加载程序所在的目录，如`user.dir`所在的位置的`class
```

#### CustomClassLoader（用户自定义类加载器）

`java`编写,用户自定义的类加载器,可加载指定路径的`class`文件

![img](https://upload-images.jianshu.io/upload_images/7634245-7b7882e1f4ea5d7d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 双亲委派机制的作用

1、防止重复加载同一个`.class`。通过委托去向上面问一问，加载过了，就不用再 加载一遍。保证数据安全。
 2、保证核心`.class`不能被篡改。通过委托方式，不会去篡改核心`.clas`，即使篡改也不会去加载，即使加载也不会是同一个`.class`对象了。不同的加载器加载同一个`.class`也不是同一个`Class`对象。这样保证了`Class`执行安全。

### 自定义classloader

1.继承classloader
2.重写findClass方法
3.将自己的内容load进内存，用defindclass方法将二进制流转换为class类对象（这一步可以加密）

## JMM内存模型

### 硬件屏障

cpu读取的时候先到高速缓存中找，找不到的话逐级向下去找，然后load到最近的一级

L3或者主存中的可能会被load到不用的cpu内部，可能会产生数据不一致的问题。

> 读取缓存是以cache line为基本单位，面前是64bytes

**总线锁**：在L3 L2中间加个锁，老cpu干的事情，影响效率

**一致性协议（MESI）**: 给每个缓存内容做了一个标记，



X86

#### 合并写

### 内存屏障



### JVM规范

LoadLoad、StoreStrore



### 硬件层数据一致性

协议很多

intel 用MESI

https://www.cnblogs.com/z00377750/p/9180644.html

现代CPU的数据一致性实现 = 缓存锁(MESI ...) + 总线锁

读取缓存以cache line为基本单位，目前64bytes

位于同一缓存行的两个不同数据，被两个不同CPU锁定，产生互相影响的伪共享问题

伪共享问题：JUC/c_028_FalseSharing

使用缓存行的对齐能够提高效率

### 乱序问题

CPU为了提高指令执行效率，会在一条指令执行过程中（比如去内存读数据（慢100倍）），去同时执行另一条指令，前提是，两条指令没有依赖关系

https://www.cnblogs.com/liushaodong/p/4777308.html

写操作也可以进行合并

https://www.cnblogs.com/liushaodong/p/4777308.html

JUC/029_WriteCombining

乱序执行的证明：JVM/jmm/Disorder.java

原始参考：https://preshing.com/20120515/memory-reordering-caught-in-the-act/

### 如何保证特定情况下不乱序

硬件内存屏障 X86

>  sfence:  store| 在sfence指令前的写操作当必须在sfence指令后的写操作前完成。
>  lfence：load | 在lfence指令前的读操作当必须在lfence指令后的读操作前完成。
>  mfence：modify/mix | 在mfence指令前的读写操作当必须在mfence指令后的读写操作前完成。

> 原子指令，如x86上的”lock …” 指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持程序顺序

JVM级别如何规范（JSR133）

> LoadLoad屏障：
> 	对于这样的语句Load1; LoadLoad; Load2， 
>
> 	在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
>
> StoreStore屏障：
>
> 	对于这样的语句Store1; StoreStore; Store2，
> 	
> 	在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
>
> LoadStore屏障：
>
> 	对于这样的语句Load1; LoadStore; Store2，
> 	
> 	在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
>
> StoreLoad屏障：
> 	对于这样的语句Store1; StoreLoad; Load2，
>
> ​	 在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。

volatile的实现细节

1. 字节码层面
   ACC_VOLATILE

2. JVM层面
   volatile内存区的读写 都加屏障

   > StoreStoreBarrier
   >
   > volatile 写操作
   >
   > StoreLoadBarrier

   > LoadLoadBarrier
   >
   > volatile 读操作
   >
   > LoadStoreBarrier

3. OS和硬件层面
   https://blog.csdn.net/qq_26222859/article/details/52235930
   hsdis - HotSpot Dis Assembler
   windows lock 指令实现 | MESI实现

synchronized实现细节

1. 字节码层面
   ACC_SYNCHRONIZED
   monitorenter monitorexit

2. JVM层面

   发现monitorenter monitorexit后
   C C++ 调用了操作系统提供的同步机制

3. OS和硬件层面
   X86 : lock cmpxchg / xxx
   [https](https://blog.csdn.net/21aspnet/article/details/88571740)[://blog.csdn.net/21aspnet/article/details/](https://blog.csdn.net/21aspnet/article/details/88571740)[88571740](https://blog.csdn.net/21aspnet/article/details/88571740)

## JavaAgent_AboutObject

使用JavaAgent测试Object的大小

### 对象大小（64位机）

### 观察虚拟机配置

java -XX:+PrintCommandLineFlags -version

### 普通对象

1. 对象头：markword  8
2. ClassPointer指针：-XX:+UseCompressedClassPointers 为4字节 不开启为8字节
3. 实例数据
   1. 引用类型：-XX:+UseCompressedOops 为4字节 不开启为8字节 
      Oops Ordinary Object Pointers
4. Padding对齐，8的倍数

### 数组对象

1. 对象头：markword 8
2. ClassPointer指针同上
3. 数组长度：4字节
4. 数组数据
5. 对齐 8的倍数

### 实验

1. 新建项目ObjectSize （1.8）

2. 创建文件ObjectSizeAgent

   ```java
   package com.mashibing.jvm.agent;
   
   import java.lang.instrument.Instrumentation;
   
   public class ObjectSizeAgent {
       private static Instrumentation inst;
   
       public static void premain(String agentArgs, Instrumentation _inst) {
           inst = _inst;
       }
   
       public static long sizeOf(Object o) {
           return inst.getObjectSize(o);
       }
   }
   ```

3. src目录下创建META-INF/MANIFEST.MF

   ```java
   Manifest-Version: 1.0
   Created-By: mashibing.com
   Premain-Class: com.mashibing.jvm.agent.ObjectSizeAgent
   ```

   注意Premain-Class这行必须是新的一行（回车 + 换行），确认idea不能有任何错误提示

4. 打包jar文件

5. 在需要使用该Agent Jar的项目中引入该Jar包
   project structure - project settings - library 添加该jar包

6. 运行时需要该Agent Jar的类，加入参数：

   ```java
   -javaagent:C:\work\ijprojects\ObjectSize\out\artifacts\ObjectSize_jar\ObjectSize.jar
   ```

7. 如何使用该类：

   ```java
   ​```java
      package com.mashibing.jvm.c3_jmm;
      
      import com.mashibing.jvm.agent.ObjectSizeAgent;
      
      public class T03_SizeOfAnObject {
          public static void main(String[] args) {
              System.out.println(ObjectSizeAgent.sizeOf(new Object()));
              System.out.println(ObjectSizeAgent.sizeOf(new int[] {}));
              System.out.println(ObjectSizeAgent.sizeOf(new P()));
          }
      
          private static class P {
                              //8 _markword
                              //4 _oop指针
              int id;         //4
              String name;    //4
              int age;        //4
      
              byte b1;        //1
              byte b2;        //1
      
              Object o;       //4
              byte b3;        //1
      
          }
      }
   ```

### Hotspot开启内存压缩的规则（64位机）

1. 4G以下，直接砍掉高32位
2. 4G - 32G，默认开启内存压缩 ClassPointers Oops
3. 32G，压缩无效，使用64位
   内存并不是越大越好（^-^）

### IdentityHashCode的问题

回答白马非马的问题：

当一个对象计算过identityHashCode之后，不能进入偏向锁状态

https://cloud.tencent.com/developer/article/1480590
 https://cloud.tencent.com/developer/article/1484167

https://cloud.tencent.com/developer/article/1485795

https://cloud.tencent.com/developer/article/1482500

### 对象定位

•https://blog.csdn.net/clover_lily/article/details/80095580

1. 句柄池
2. 直接指针



### 对象头里有什么

根据对象的锁状态来分配

![头两位](E:\deng\deng\MD\jvm\img\头两位.png)

==锁标志位==和==分代年龄==重点



## 运行时数据区

jvms 2.4 2.5运行时数据区和常用指令

### 指令集分类

1. 基于寄存器的指令集
2. 基于栈的指令集
   Hotspot中的Local Variable Table = JVM中的寄存器

### Runtime Data Area

**PC 程序计数器**

> 存放指令位置
>
> 虚拟机的运行，类似于这样的循环：
>
> while( not end ) {
>
> ​	取PC中的位置，找到对应位置的指令；
>
> ​	执行该指令；
>
> ​	PC ++;
>
> }

**JVM Stack**

> 每个线程有自己的jvm stack,想象画面

1. Frame - 每个方法对应一个栈帧
   1. Local Variable Table 局部变量
   2. Operand Stack
      对于long的处理（store and load），多数虚拟机的实现都是原子的
      jls 17.7，没必要加volatile
   3. ==Dynamic Linking==
      https://blog.csdn.net/qq_41813060/article/details/88379473 
      jvms 2.6.3
   4. return address
      a() -> b()，方法a调用了方法b, b方法的返回值放在什么地方

![栈帧面试题](E:\deng\deng\MD\jvm\img\栈帧面试题.png)

1：把8放到操作栈
2：将栈中的8赋值给局部变量表变为8
3:   把局部变量表中i拿出来压栈
4：将局部变量表中位置为1的数+1变为9
5：将栈中的8赋值给局部变量表覆盖了9

---

![一个实验](E:\deng\deng\MD\jvm\img\一个实验.png)

dup复制一次用于初始化对象，剩下一个完成赋值

---

**Direct Memory**

> java1.4后出现，用户空间直接去访问内核空间的一些内存（NIO相关）



**Heap**

**Method Area 方法区**

> 1.8前指的就是永久区，1.8后meta space

1. Perm Space (<1.8)
   字符串常量位于PermSpace
   FGC不会清理
   大小启动的时候指定，不能变
2. Meta Space (>=1.8)
   字符串常量位于堆
   会触发FGC清理
   不设定的话，最大就是物理内存

Runtime Constant Pool

> 常量池的内容扔在里面

Native Method Stack

Direct Memory

> JVM可以直接访问的内核空间的内存 (OS 管理的内存)
>
> NIO ， 提高效率，实现zero copy

思考：

> 如何证明1.7字符串常量位于Perm，而1.8位于Heap？
>
> 提示：结合GC， 一直创建字符串常量，观察堆，和Metaspace

![线程共享区域](E:\deng\deng\MD\jvm\img\线程共享区域.png)

### 常见指令

store

load

pop

mul

sub

invoke

1. InvokeStatic
2. InvokeVirtual
3. InvokeInterface
4. InovkeSpecial
   可以直接定位，不需要多态的方法
   private 方法 ， 构造方法
5. InvokeDynamic
   JVM最难的指令
   lambda表达式或者反射或者其他动态语言scala kotlin，或者CGLib ASM，动态产生的class，会用到的指令

##  GC的基础

> C++
>
> 	- 手工处理垃圾
>
>  - 忘记回收
>    - 内存泄露
>  - 回收多次
>    - 非法访问
>    - 开发效率低，执行效率高
>
> Java
>
> 	* GC处理垃圾
> 	* 开发效率高，执行效率低

==没有任何引用==指向的一个对象或者多个对象（==循环引用==）

#### 2.如何定位垃圾

* 1.引用计数（ReferenceCount）
  * 被引用次数为0 时清除
  * 无法解决循环引用，引起内存泄露

* 2.根可达算法(RootSearching)
  * 通过正在使用的一些root程序向下找,找不到的都是垃圾

#### 3.常见的垃圾回收算法

1. 标记清除(mark sweep) - 找到垃圾，标记。
   * 位置不连续 产生碎片 效率偏低（两遍扫描）
   * 存活对象多的情况下效率比较高
2. 拷贝算法 (copying) - 没有碎片，浪费空间
   * 只用一半内存，回收时把在用的内存拷贝到备用空间。
3. 标记压缩(mark compact) - 没有碎片，效率偏低（两遍扫描，指针需要调整）
   * 效率比copy低，

#### 4.JVM内存分代模型（用于分代垃圾回收算法）

![堆内存逻辑分区](E:\deng\deng\MD\jvm\img\堆内存逻辑分区.png)

8:1:1是因为根据经验，9成对象会被回收

1. 部分垃圾回收器使用的模型

   > 除Epsilon ZGC Shenandoah之外的GC都是使用逻辑分代模型
   >
   > G1是逻辑分代，物理不分代
   >
   > 除此之外不仅逻辑分代，而且物理分代

2. 新生代 + 老年代 + 永久代（1.7）Perm Generation/ 
   元数据区(1.8) Metaspace

   1. 永久代 元数据 - Class
   2. 永久代必须指定大小限制 ，元数据可以设置，也可以不设置，无上限（受限于物理内存）
   3. 字符串常量 1.7 - 永久代，1.8 - 堆
   4. MethodArea逻辑概念 - 永久代、元数据

3. 新生代 = Eden + 2个suvivor区 

   1. YGC回收之后，大多数的对象会被回收，活着的拷贝进s0

      > 比如一个for循环会new很多对象，然后被GC掉

   2. 再次YGC，活着的对象eden + s0 -> s1

   3. 再次YGC，eden + s1 -> s0

   4. 年龄足够 -> 老年代 （大部分15岁， CMS 6岁）

   5. s区==装不下== -> 老年代

4. 老年代

   1. 顽固分子
   2. 老年代满了FGC（Full GC）

5. GC Tuning (Generation) ==调优目标==

   1. 尽量减少FGC

      > 非常耗时，会出现STW（stop world），系统停顿现象

   2. MinorGC = YGC

   3. MajorGC = FGC

6. 对象分配过程图

> 

   ![](E:/deng/deng/MD/jvm/对象分配过程详解.png)

7. 动态年龄：（不重要）
   https://www.jianshu.com/p/989d3b06a49d

8. 分配担保：（不重要）
   YGC期间 survivor区空间不够了 空间担保直接进入老年代
   参考：https://cloud.tencent.com/developer/article/1082730

#### 5.常见的垃圾回收器

![常用垃圾回收器](E:\deng\deng\MD\jvm\img\常用垃圾回收器.png)

> 左边物理和概念上都分代，右边只是逻辑上分代
>
> 一般线上默认PS+PO
>
> 还有				PN+CMS

1. JDK诞生后就伴随Serial，提高效率，诞生了PS，为了配合CMS，诞生了PN，CMS是1.4版本后期引入，CMS是里程碑式的GC，它开启了并发回收的过程，但是CMS毛病较多，因此目前没有  任何一个JDK版本默认是CMS
   并发垃圾回收是因为无法忍受STW

2. Serial 年轻代 串行回收

   > 单线程，STW，回收时其他线程停止，现在用的极少

3. PS 年轻代 并行回收

   > 多线程，回收时其他线程停止

4. ParNew 年轻代 配合CMS的并行回收

   > PS基础上为了配合CMS

5. SerialOld 

6. ParallelOld

7. ConcurrentMarkSweep 老年代 并发的， 垃圾回收和应用程序同时运行，降低STW的时间(200ms)
   CMS问题比较多，所以现在没有一个版本默认是CMS，只能手工指定
   CMS既然是MarkSweep，就一定会有碎片化的问题，碎片到达一定程度，CMS的老年代分配对象分配不下的时候，使用SerialOld 进行老年代回收
   想象一下：
   PS + PO -> 加内存 换垃圾回收器 -> PN + CMS + SerialOld（几个小时 - 几天的STW）
   几十个G的内存，单线程回收 -> G1 + FGC 几十个G -> 上T内存的服务器 ZGC
   算法：三色标记 + Incremental Update

8. G1(10ms)
   算法：三色标记 + SATB

9. ZGC (1ms) PK C++
   算法：ColoredPointers + LoadBarrier

10. Shenandoah
    算法：ColoredPointers + WriteBarrier

11. Eplison

12. PS 和 PN区别的延伸阅读：
    ▪[https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html#GUID-3D0BB91E-9BFF-4EBB-B523-14493A860E73](https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html)

13. 垃圾收集器跟内存大小的关系

    1. Serial 几十兆
    2. PS 上百兆 - 几个G
    3. CMS - 20G
    4. G1 - 上百G
    5. ZGC - 4T - 16T（JDK13）

1.8默认的垃圾回收：==PS== + ==ParallelOld==



![CMS](E:\deng\deng\MD\jvm\img\CMS.png)







# 其它

## 沙箱安全机制

凡带了native关键字的，说明超过了java的作用范围，会去调用底层C语言的库。

进入本地方法栈，调用JNI

JNI作用：扩展java使用，融合不同编程语言为java所用

诞生之初是为了在C、C++的环境下立足。

## 方法区

Method Area方法区
方法区是被所有线程共享，所有字段和方法字节码，以及一些特殊方法，如构造函数，接口代码也在此定义，简单说，所有定义的方法的信息都保存在该区域，此区域属于共享区间；

**静态变量、常量、类信息（构造方法、接口定义）、运行时的常量池存在方法区中，但是实例变量存在堆内存中，和方法区无关**
static final，Class，常量池

## 栈

数据结构

栈内存，主管程序的运行，生命周期和线程同步

线程结束，也就是栈内存释放，对于栈来说，不存在垃圾回收问题

栈内：八大基本类型+对象引用+实例方法

![栈](E:\deng\image\jvm\栈.png)

## 堆

堆里的是方法引用，常量池分为静态常量池和运行时常量池，方法区里的是静态常量池，堆里的是运行时常量池（查的资料）

堆内存细分为三个区域

* 新生区
  * 
* 老年区
* 永久区
  * 1.6前：永久代，常量池在方法区
  * 1.7    ：永久区，但慢慢退化，`去永久代`，常量池在堆中
  * 1.8后：无永久代，常量池在元空间
  * 这个区域常驻内存的。用来存放DK自身携带的Class对象。Interface元数据，存储的是Java运行时的一些环境或类信息~，这个区域不存在垃圾回收！关闭VM虚拟就会释放这个区域的内存~



jdk8以后，永久存储区改为了元空间



OOM 堆内存满

-Xms：设置初始化内存分配大小 1/64

-Xms：设置最大分配内存，默认1/4

-XX:+PrintGCDetails 打印GC垃圾回收信息

-Xms8m -Xms8m -XX:+PrintGCDetails

​		1.尝试扩大内存看结果

-Xms1m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError





## GC

分代收集算法

轻GC、重GC

### 引用计数法

占内存

### 复制算法

![复制算法](E:\deng\image\jvm\复制算法.png)



年轻代主要是用此算法

1.每次GC都会将Eden活的对象移到幸存区中的to区，同时将原来的from区复制到to区，然后剩下空的变成to区。

2.当一个对象经历了15次GC，都还没有死，进入养老区
-XX:-XX:MaxTenuring Threshold=9999通过这个参数可以设定进入老年代的次数

**好处**：没有内存碎片

**坏处**：浪费了一半的内存空间。

**应用场景**：对象存活度较低时，否则每次要从from复制大量的数据到to



### 标记清除

老年代用此

优点：无需额外空间

缺点：两次扫描，浪费时间，产生碎片

