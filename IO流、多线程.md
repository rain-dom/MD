## I/O

![IO](E:\deng\note\IO.png)

### 1.1 字节流

---

#### 1.1 InputStream

##### 1.11 object-Stream

输入输出顺序须一致

```java
public class Person implements Serializable {
    //序列化操作
    long serialVersionUID = 1L;
    private String name;
    private int age;
    transient private int pwd;
    //加transient时不会被记录
```

```java
public class object输入输出 {

    public static void main(String[] args) {

        ObjectOutputStream objectOutputStream = null;
        FileOutputStream fileOutputStream = null;
        ObjectInputStream objectInputStream = null;
        FileInputStream fileInputStream = null;
        try {
            fileOutputStream = new FileOutputStream("abc.txt");
            objectOutputStream = new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeUTF("潘冬练习dzp");
            objectOutputStream.writeObject(new Person("潘冬",5,259));

            fileInputStream = new FileInputStream("abc.txt");
            objectInputStream = new ObjectInputStream(fileInputStream);

            //顺序必须于输入的一致
            System.out.println(objectInputStream.readUTF());
            Object object = objectInputStream.readObject();
            //判断输入流中的object是否是Person类的实例对象
            if(object instanceof Person){
                Person p = (Person) object;
                System.out.println(p);
            }
```



##### 1.12 Data-Stream

输入输出顺序须一致

#### 1.2 OutputStream

![java流分类](E:\deng\image\java流分类.png)

```java
package com.dzp.StreamDemo;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
、
public class StreamDemo {
    public static void main(String[] args) {
        //input流读取数据
        InputStream inputStream = null;
        try {
            inputStream = new FileInputStream("abc.txt");
            //
            byte[] buffer = new byte[1024];
            int length = 0;
            while((length = inputStream.read(buffer,2,5))!= -1){//从位置2开始放，放5个
                System.out.println(new String(buffer,0,length));//从0开始读去，读length个
            }

```



### 1.2 字符流

---

#### Reader

---

##### BufferedReader

```java
public class 控制台流输入输出 {
    public static void main(String[] args) {

        try(OutputStreamWriter outputStreamWriter = new OutputStreamWriter(System.out);
            InputStreamReader inputStreamReader  = new InputStreamReader(System.in);
            BufferedWriter bufferedWriter = new BufferedWriter(outputStreamWriter);
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
        ) {
            String str = " ";
            while (!str.equals("exit")) {
                str = bufferedReader.readLine();
                bufferedWriter.write(str);
                bufferedWriter.flush();
            }


        } catch (IOException e) {
                e.printStackTrace();
        }


    }
}
```



#### Write

---



### 1.3 打印流





~~~java
public class 打印流格式 {
    public static void main(String[] args) {
        PrintStream printStream =null;
        printStream = new PrintStream(System.out);
        try {
            printStream.write("hello".getBytes());
            printStream.printf("%s--%d---%.2f","abc",123,123.111);


        } catch (IOException e) {
            e.printStackTrace();
        }


    }
}
~~~

RandomAccessFile

* 可读可写

```java
//读取doc.txt后，以1024为块读取并打印
public class RandomAccessFileTest {
    public static void main(String[] args) {
        File file = new File("doc.txt");
        long length = file.length();
        int blockSize = 1024;
        int size =  (int)Math.ceil(length*1.0/blockSize);
        System.out.println();
        int actuallySize;
        int beginPoint;

        //这里是printf
        System.out.printf("共计切成<%d>块",size);
        for(int i=0;i<size;i++){
            beginPoint = i*1024;
            if(i != size-1){
                actuallySize = blockSize;
                length -= blockSize;
            }else{
                actuallySize = (int)length;
            }
            System.out.println("这是第"+i+"读取"+"---从"+beginPoint+"处读取，读取大小为"+actuallySize);
            readSpilt(i,beginPoint,actuallySize);
            System.out.println("-------------------------------------");

        }

    }
    public static void readSpilt(int i,int beginPoint,int actuallySize){
        RandomAccessFile randomAccessFile = null;
        try {
            //后面的 r 代表读取

            randomAccessFile = new RandomAccessFile(new File("doc.txt"),"r");
            randomAccessFile.seek(beginPoint);
            byte[] bytes = new byte[1024];

            randomAccessFile.read(bytes,0,actuallySize);
            System.out.println(new String(bytes));


        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                randomAccessFile.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }

}
```





## 多线程

### 前言

**进程**：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1--n个线程。（进程是资源分配的最小单位）

**线程**：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。（线程是cpu调度的最小单位）



main方法其实也是一个线程

在java中，每次程序运行至少启动2个线程。一个是main线程，一个是垃圾收集线程。因为每当使用java命令执行一个类的时候，实际上都会启动一个JVM，每一个jvm实际就是在操作系统中启动了一个进程。



interrupt结束线程

### 练习题目录

c_009

### **实现方式**

* 继承*Tread*类,重写run方法
* 实现*Runable*接口
  * new Thread(new Runable()).start
  * new Thread(()->{sout("hello，thread");}).start
* 实现callable接口（有返回值）

```java
public static void main(String[] args) {
    T01 t01 = new T01();
    //new Thread(()->t.m1(),"t1")
    //相当于内部new Runable()
    new Thread(t01::m1,"t1").start();
    new Thread(t01::m2,"t2").start();
}
```

#### 扩展java.lang.Thread类

：start()方法的调用后并不是立即执行多线程代码，而是使得该线程变为可运行态（Runnable），什么时候运行是由操作系统决定的。

#### 实现java.lang.Runnable接口

Thread类实际上也是实现了Runnable接口的类。

不管是扩展Thread类还是实现Runnable接口来实现多线程，最终还是通过Thread的对象的API来控制线程的，熟悉Thread类的API是进行多线程编程的基础。

#### Thread和Runnable的区别

runnable优势

1）：适合多个相同程序代码的线程去处理同一个资源

2）：可以避免java中的单继承的限制

3）：增加程序的健壮性，代码可以被多个线程共享，代码和数据独立

4）：线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类



### 线程的生命周期

![img](https://img-blog.csdn.net/20150309140927553)

#### 1、新生状态：

​          当创建好当前线程对象之后，没有启动之前（调用start方法之前）
​          ThreadDemo thread = new ThreadDemo()
​          RunnableDemo run = new RunnableDemo()

#### 2、就绪状态：

​		  线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。

#### 3、运行状态：

当当前进程获取到cpu资源之后，就绪队列中的所有线程会去抢占cpu的资源，谁先抢占到谁先执行，在执行的过程中就叫做运行状态
          就绪状态的线程获取了CPU，执行程序代码。

#### 4、死亡状态：

当运行中的线程正常执行完所有的代码逻辑或者因为异常情况导致程序结束叫做死亡状态
     进入的方式：
       1、正常运行完成且结束
        2、人为中断执行，比如使用stop方法
        3、程序抛出未捕获的异常

#### 5、阻塞状态：

在程序运行过程中，发生某些异常情况，导致当前线程无法再顺利执行下去，此时会进入阻塞状态，进入阻塞状态的原因消除之后，所有的阻塞队列会再次进入到就绪状态中，随机抢占cpu的资源，等待执行.
          进入的方式：

（一）、等待阻塞：运行的线程执行**wait()**方法，JVM会把该线程放入等待池中。(**wait会释放持有的锁**)

（二）、同步阻塞：运行的线程在获取对象的同步锁时，若该**同步锁被别的线程占用**，则JVM会把该线程放入锁池中。

（三）、其他阻塞：运行的线程执行**sleep()或join()方法**，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。（注意,sleep是不会释放持有的锁）

#### 注意：

* 在多线程的时候，可以实现唤醒和等待的过程，但是唤醒和等待操作的对应不是thread类，而是我们设置的共享对象或者共享变量，都是Object的子类
* 多线程并发访问的时候回出现数据安全问题：
  * 解决方式：
                  1、同步代码块
                      synchronized(共享资源、共享对象，需要是object的子类){具体执行的代码块}
                  2、同步方法
                      将核心的代码逻辑定义成一个方法，使用synchronized关键字进行修饰，此时不需要指定共享对象



### 线程调度

**1、调整线程优先级**

Java线程的优先级用整数表示，取值范围是1~10，Thread类有以下三个静态常量：

```cpp
static int MAX_PRIORITY
          线程可以具有的最高优先级，取值为10。
static int MIN_PRIORITY
          线程可以具有的最低优先级，取值为1。
static int NORM_PRIORITY
          分配给线程的默认优先级，取值为5。
```

setPriority()和getPriority()方法分别用来设置和获取线程的优先级。

**2、线程睡眠**

Thread.sleep(long millis)方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。

**3、线程等待：**

Object类中的wait()方法，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。

**4、线程让步**：Thread.yield() 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

**5、线程加入**：join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

 **6、线程唤醒**：



Thread implements Runnable

```java
//同步代码块
public class Runnerble卖票 implements Runnable {

    for(int i =0;i<10;i++){
            try {
                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (this){
                if(tickets>0) {
                    System.out.println(Thread.currentThread().getName() + "正在卖第" + (tickets--) + "张票");
                }
            }
        }
    public static void main(String[] args) {
        Runnerble卖票 runnerble卖票 = new Runnerble卖票();
        //所有的线程共用Runnerble一个接口，因此其中的变量会共享
        //相当于4个代理人干同一件事
        Thread thread1 = new Thread(runnerble卖票);
        Thread thread2 = new Thread(runnerble卖票);
        Thread thread3 = new Thread(runnerble卖票);
        Thread thread4 = new Thread(runnerble卖票);
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();

    }
}
```

```java
//同步方法
public class Runnerble卖票 implements Runnable {

    private int tickets =10;
    @Override
    public void run() {

        for(int i =0;i<10;i++){
            try {
                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.sale();
        }
    }
    public synchronized void  sale(){
        if(tickets>0) {
            System.out.println(Thread.currentThread().getName() + "正在卖第" + (tickets--) + "张票");
        }
    }
    public static void main(String[] args) {
        Runnerble卖票 runnerble卖票 = new Runnerble卖票();
        Thread thread1 = new Thread(runnerble卖票,"A");
        Thread thread2 = new Thread(runnerble卖票,"B");
        Thread thread3 = new Thread(runnerble卖票,"C");
        Thread thread4 = new Thread(runnerble卖票,"D");
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();

    }
}
```

### 线程api方法

![线程操作](E:\deng\image\线程操作.png)

**join()方法**

**等待该线程终止**

在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

**yield():**

**暂停当前正在执行的线程对象，并执行其他线程。**

**学术上存在，工程上很少用**

先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把 CPU 的占有权交给此线程，否则，继续运行原来的线程。

但有可能继续抢占到cpu

**wait()**

Obj.wait()，与Obj.notify()必须要与synchronized(Obj)一起使用,是针对已经获取了Obj锁进行操作

Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于Object.wait()在释放CPU同时，释放了对象锁的控制。

**sleep（）**

 sleep()是Thread类的Static(静态)的方法；因此他不能改变对象的机锁，所以当在一个Synchronized块中调用Sleep()方法是，线程虽然休眠了，但是对象的机锁并木有被释放，其他线程无法访问这个对象



## 面试题

高并发4 - 10:14

## 锁

锁的是对象不是代码，也不是你想同步的数字，这个锁定的对象甚至可以是你自定义的对象

synchronized(Object)

-但是不能用String Integer Long

**被锁的对象二进制的前两位状态不同**

### 锁分类

#### synchronize锁(悲观锁)

##### 特性

sleep不会释放锁，wait会

**可重入锁**
	如果加锁的方法a中调用了加锁方法b，当a发现b加锁的对象与自己相同时，会允许执行。
如果不可重入，会死锁。

##### 锁的升级(无锁、偏向、自旋、重量级)

synchronize的底层实现

jdk早期是 重量级锁-os ,都要找操作系统（内核）去申请锁
首先是**偏向锁**，markword 记录这个线程ID （二进制的头两位,00代表无锁，01代表偏向锁，10轻量级锁，11重量级锁）
如果线程争用，升级为**自旋锁**（循环判断是否能拿到锁，占用cpu，但是不访问操作系统，只在用户态解决）
自旋10次后升级为**重量级锁-OS**，进入等待状态，不占用cpu(状态不可退回，但虚拟机也可以实现)



另一个线程循环判断是否能拿到锁

偏向锁不是加锁，而是记住第一个拿到的线程，倾向于他先执行



##### 锁的不同实现方式

```java
public class T01 {

    private int i = 0;
    public Object o = new Object();
    public void m(){
        synchronized (o){
            i++;
            System.out.println(Thread.currentThread().getName());
        }
    }
    //锁住当前对象
    public void s(){
        synchronized (this){
            i++;
            System.out.println(Thread.currentThread().getName());
        }
    }
    //执行时与上一个等价
    public synchronized void n(){
        i++;
        System.out.println(Thread.currentThread().getName());
    }
    public synchronized static void o(){//此时等同于synchronized（T01.class）
    }
}
```



模拟银行账户

```java
/**
 * 面试题：模拟银行账户
 * 对业务写方法加锁
 * 对业务读方法不加锁
 * 这样行不行？
 *
 * 容易产生脏读问题（dirtyRead）
 */

package com.dzp;

import java.util.concurrent.TimeUnit;

public class Account {
    String name;
    double balance;
    public synchronized void set(String name,double balance){
        this.name = name;
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.balance =balance;
    }

    public double getBalance(String name){
        return this.balance;
    }

    public static void main(String[] args) {
        Account a = new Account();
        new Thread(()->a.set("lisi",100.0)).start();
        System.out.println(a.getBalance("lisi"));
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(a.getBalance("lisi"));
    }



}
```

##### 异常处理

程序在执行过程中，如果出现异常，默认情况锁会被释放。

```java
/**
 * 程序在执行过程中，如果出现异常，默认情况锁会被释放
 * 所以，在并发处理的过程中，有异常要多加小心，不然可能会发生不一致的情况。
 * 比如，在一个web app处理过程中，多个servlet线程共同访问同一个资源，这时如果异常处理不合适，
 * 在第一个线程中抛出异常，其他线程就会进入同步代码区，有可能会访问到异常产生时的数据。
 * 因此要非常小心的处理同步业务逻辑中的异常
 * @author mashibing
 */
package com.mashibing.juc.c_011;

import java.util.concurrent.TimeUnit;

public class T {
   int count = 0;
   synchronized void m() {
      System.out.println(Thread.currentThread().getName() + " start");
      while(true) {
         count ++;
         System.out.println(Thread.currentThread().getName() + " count = " + count);
         try {
            TimeUnit.SECONDS.sleep(1);
            
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
         
         if(count == 5) {
            int i = 1/0; //此处抛出异常，锁将被释放，要想不被释放，可以在这里进行catch，然后让循环继续
            System.out.println(i);
         }
      }
   }
   
   public static void main(String[] args) {
      T t = new T();
      Runnable r = new Runnable() {

         @Override
         public void run() {
            t.m();
         }
         
      };
      new Thread(r, "t1").start();
      
      try {
         TimeUnit.SECONDS.sleep(3);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
      
      new Thread(r, "t2").start();
   }
   
}
```







#### CAS(乐观锁、无锁优化 自旋)

* Compare And Set	(可以当作是一个方法)
* cas(V,Expected,NewValue)
* cas(要改的值，期望值，新值)
  * -if V == E
    V = New
    otherwise try again or fail
  * 因为cpu原语支持，所以这三部不会被打乱
  * 比如count++操作，这个线程首先拿到当前的值为1，后面要进行cas操作，那么期望值就是1，此时再看V是多少，看V==E否。
* ABA问题	加版本号解决
  * 基础类型没有影响
  * 如果是引用类型，比如引用没变，但是内容变了
  * 更改了对象引用的某个对象
* 内部利用了Unsafe类实现cas操作

LOCK类型？



##### 通过 Atomic 实现

```java
public class AtomicInteger_ {
    AtomicInteger count = new AtomicInteger(0);
    void m(){
        for (int i = 0; i <1000 ; i++) {
            count.incrementAndGet();
        }
    }

    public static void main(String[] args) {
        AtomicInteger_ t = new AtomicInteger_();
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i <10 ; i++) {
             threads.add(new Thread(t::m,"thread-"+i));
        }
        threads.forEach((o)->o.start());
        threads.forEach((o)->{
            try {
                o.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println(t.count);
    }


}
```

#### LongAdder

#### 不同方案并发时间比较

**LongAdder**是通过分块锁，将一定数量的线程合并加锁，最后再加起来得到总数。并发特别高的时候有优势

```java
package com.mashibing.juc.c_018_00_AtomicXXX;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.LongAdder;

public class T02_AtomicVsSyncVsLongAdder {
    static AtomicLong count1 = new AtomicLong(0L);
    static long count2 = 0L;
    static LongAdder count3 = new LongAdder();

    public static void main(String[] args) throws Exception {
        Thread[] threads = new Thread[1000];

        for(int i=0; i<threads.length; i++) {
            threads[i] =
                    new Thread(()-> {
                        for(int k=0; k<100000; k++) count1.incrementAndGet();
                    });
        }

        long start = System.currentTimeMillis();

        for(Thread t : threads ) t.start();

        for (Thread t : threads) t.join();

        long end = System.currentTimeMillis();

        //TimeUnit.SECONDS.sleep(10);

        System.out.println("Atomic: " + count1.get() + " time " + (end-start));
        //-----------------------------------------------------------
        Object lock = new Object();

        for(int i=0; i<threads.length; i++) {
            threads[i] =
                new Thread(new Runnable() {
                    @Override
                    public void run() {

                        for (int k = 0; k < 100000; k++)
                            synchronized (lock) {
                                count2++;
                            }
                    }
                });
        }

        start = System.currentTimeMillis();

        for(Thread t : threads ) t.start();

        for (Thread t : threads) t.join();

        end = System.currentTimeMillis();


        System.out.println("Sync: " + count2 + " time " + (end-start));


        //----------------------------------
        for(int i=0; i<threads.length; i++) {
            threads[i] =
                    new Thread(()-> {
                        for(int k=0; k<100000; k++) count3.increment();
                    });
        }

        start = System.currentTimeMillis();

        for(Thread t : threads ) t.start();

        for (Thread t : threads) t.join();

        end = System.currentTimeMillis();

        //TimeUnit.SECONDS.sleep(10);

        System.out.println("LongAdder: " + count1.longValue() + " time " + (end-start));

    }

    static void microSleep(int m) {
        try {
            TimeUnit.MICROSECONDS.sleep(m);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

#### ReentrantLock

* 特点
  
* 必须手动释放锁（放在finally中执行）
  
* 可重入锁
  * 在同一个线程中
  * 执行同步方法1中调用了同步方法2，两者锁的是同一个对象，2可以执行

* 比synchronize灵活

* 可设置为公平锁

  * ```java
    private static ReentrantLock lock=new ReentrantLock(true); //参数为true表示为公平锁，请对比输出结果
    ```

  * 新线程会先检查队列中有没有线程，如果有，就进队列等待其他线程先执行

  * 公平锁下不会出现大规模线程1在执行，大概是线程1,2交替执行，线程1也会连续执行2次，因为在1执行完时，其他线程还没有进队列，线程1又一次进入，所以会连续执行，但不会大规模连续执行，因为线程2不会拖延太长时间挤不进去。

* api

  * trylock（5，TimeUnit.SECONDS）
  * lockinterupptibly (用此方法加锁，其被它线程可用interrupt打断 )

  ```java
  /**
   * reentrantlock用于替代synchronized
   * 由于m1锁定this,只有m1执行完毕的时候,m2才能执行
   * 这里是复习synchronized最原始的语义
   * 
   * 使用reentrantlock可以完成同样的功能
   * 需要注意的是，必须要必须要必须要手动释放锁（重要的事情说三遍）
   * 使用syn锁定的话如果遇到异常，jvm会自动释放锁，但是lock必须手动释放锁，因此经常在finally中进行锁的释放
   * 
   * 使用reentrantlock可以进行“尝试锁定”tryLock，这样无法锁定，或者在指定时间内无法锁定，线程可以决定是否继续等待
   * @author mashibing
   */
  package com.mashibing.juc.c_020;
  
  import java.util.concurrent.TimeUnit;
  import java.util.concurrent.locks.Lock;
  import java.util.concurrent.locks.ReentrantLock;
  
  public class T03_ReentrantLock3 {
  	Lock lock = new ReentrantLock();
  	void m1() {
  		try {
  			lock.lock();
  			for (int i = 0; i < 3; i++) {
  				TimeUnit.SECONDS.sleep(1);
  
  				System.out.println(i);
  			}
  		} catch (InterruptedException e) {
  			e.printStackTrace();
  		} finally {
  			lock.unlock();
  		}
  	}
  
  	/**
  	 * 使用tryLock进行尝试锁定，不管锁定与否，方法都将继续执行
  	 * 可以根据tryLock的返回值来判定是否锁定
  	 * 也可以指定tryLock的时间，由于tryLock(time)抛出异常，所以要注意unclock的处理，必须放到finally中
  	 */
  	void m2() {
  		/*
  		boolean locked = lock.tryLock();
  		System.out.println("m2 ..." + locked);
  		if(locked) lock.unlock();
  		*/
  		
  		boolean locked = false;
  		
  		try {
  			locked = lock.tryLock(5, TimeUnit.SECONDS);
  			System.out.println("m2 ..." + locked);
  		} catch (InterruptedException e) {
  			e.printStackTrace();
  		} finally {
  			if(locked) lock.unlock();
  		}
  		
  	}
  
  	public static void main(String[] args) {
  		T03_ReentrantLock3 rl = new T03_ReentrantLock3();
  		new Thread(rl::m1).start();
  		try {
  			TimeUnit.SECONDS.sleep(1);
  		} catch (InterruptedException e) {
  			e.printStackTrace();
  		}
  		new Thread(rl::m2).start();
  	}
  }
  
  ```





#### CountDownLatch

设置门栓数量，主线程调用await阻塞。
线程调用countDown来使latch减一（随便你怎么减），直到latch减为0，解除await。

也可以让其它线程调用join达到同样目的

```java
package com.mashibing.juc.c_020;

import java.util.concurrent.CountDownLatch;

public class T06_TestCountDownLatch {
    public static void main(String[] args) {
        usingJoin();
        usingCountDownLatch();
    }

    private static void usingCountDownLatch() {
        Thread[] threads = new Thread[100];
        CountDownLatch latch = new CountDownLatch(threads.length);

        for(int i=0; i<threads.length; i++) {
            threads[i] = new Thread(()->{
                int result = 0;
                for(int j=0; j<10000; j++) result += j;
                latch.countDown();
            });
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }

        //拴住，latch不到0不让走。
       	//和调用join方法相似（join是把主线程放在所有线程之后执行）
        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("end latch");
    }

    private static void usingJoin() {
        Thread[] threads = new Thread[100];

        for(int i=0; i<threads.length; i++) {
            threads[i] = new Thread(()->{
                int result = 0;
                for(int j=0; j<10000; j++) result += j;
            });
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }

        for (int i = 0; i < threads.length; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("end join");
    }
}
```

#### CyclicBarrier

CyclicBarrier barrier = new CyclicBarrier(20，new Runnable(){})
调用 barrier.await()，直到await达到20次，执行一次runnable中的代码
每满20发一车

**场景：**一个复杂操作需要其它各个操作完成再执行。那么在并发场景下，其它操作调用await

```java
public class T07_TestCyclicBarrier {
    public static void main(String[] args) {
        //CyclicBarrier barrier = new CyclicBarrier(20);

        CyclicBarrier barrier = new CyclicBarrier(20, () -> System.out.println("满人"));

        /*CyclicBarrier barrier = new CyclicBarrier(20, new Runnable() {
            @Override
            public void run() {
                System.out.println("满人，发车");
            }
        });*/

        for(int i=0; i<100; i++) {

                new Thread(()->{
                    try {
                        barrier.await();

                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } catch (BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                }).start();
        }
    }
}
```

#### Phaser

分阶段的栅栏

架构03 	1：50



#### 共享锁和排它锁

##### 读锁

允许其它线程读，不许写

所有readLock.lock均不会互相阻碍


##### 写锁   

​	读锁共享，写锁排他

```java
public class T10_TestReadWriteLock {
    static Lock lock = new ReentrantLock();
    private static int value;

    static ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    static Lock readLock = readWriteLock.readLock();
    static Lock writeLock = readWriteLock.writeLock();

    public static void read(Lock lock) {
        try {
            lock.lock();
            Thread.sleep(1000);
            System.out.println("read over!");
            //模拟读取操作
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void write(Lock lock, int v) {
        try {
            lock.lock();
            Thread.sleep(1000);
            value = v;
            System.out.println("write over!");
            //模拟写操作
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }



    public static void main(String[] args) {
        //Runnable readR = ()-> read(lock); 不用读写锁的情况
        Runnable readR = ()-> read(readLock);

        //Runnable writeR = ()->write(lock, new Random().nextInt());
        Runnable writeR = ()->write(writeLock, new Random().nextInt());

        for(int i=0; i<18; i++) new Thread(readR).start();
        for(int i=0; i<2; i++) new Thread(writeR).start();


    }
}
```

#### Semaphore

 `acquire()`阻塞获取锁，成功就-1，最后再`release()`,再+1.

限制同时运行的线程数量，默认是非公平锁

```java
public class T11_TestSemaphore {
    public static void main(String[] args) {
        //Semaphore s = new Semaphore(2);
        Semaphore s = new Semaphore(2, true);
        //允许一个线程同时执行
        //Semaphore s = new Semaphore(1);

        new Thread(()->{
            try {
                //将使得设定的数字-1，当减为0时，其它线程无法拿到锁
                s.acquire();

                System.out.println("T1 running...");
                Thread.sleep(200);
                System.out.println("T1 running...");

            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                s.release();
            }
        }).start();

        new Thread(()->{
            try {
                s.acquire();

                System.out.println("T2 running...");
                Thread.sleep(200);
                System.out.println("T2 running...");

                s.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

#### Exchamge

交换两个线程中的值

#### LockSupport

无需synchronize锁住某个目标（或调用wait），就可以实现对线程的阻塞等控制

LockSupport.park

LockSupport.unpark (t)		

unpark可以先于park调用,不像notify，必须wait以后调用



```java
import java.util.concurrent.locks.LockSupport;

public class TestLockSupport {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(i);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if (i == 3){
                    LockSupport.park();
                }
            }
        });

        t.start();
//        LockSupport.unpark(t);

    }
}
```

 

#### 阿里题目

观察者线程1与加内容线程2 synchronized同一个对象。观察者线程先启动，判断size不是5，调用wait，另一个线程到达5时唤醒线程1，同时自己调用wait，释放锁让1拿到锁继续走流程，流程最后需要再调用notify唤醒线程2



直接用LinkedList这种非线程安全的容器会出问题，因为size方法更新不是即使的，另一个线程无法即使感知到当前线程添加元素导致的size值的变化。给list加上volatile也无法解决

##### *synchronize 写法

wait notify写法

```java
/**
 * 曾经的面试题：（淘宝？）
 * 实现一个容器，提供两个方法，add，size
 * 写两个线程，线程1添加10个元素到容器中，线程2实现监控元素的个数，当个数到5个时，线程2给出提示并结束
 * 
 * 给lists添加volatile之后，t2能够接到通知，但是，t2线程的死循环很浪费cpu，如果不用死循环，该怎么做呢？
 * 
 * 这里使用wait和notify做到，wait会释放锁，而notify不会释放锁
 * 需要注意的是，运用这种方法，必须要保证t2先执行，也就是首先让t2监听才可以
 * 
 * 阅读下面的程序，并分析输出结果
 * 可以读到输出结果并不是size=5时t2退出，而是t1结束时t2才接收到通知而退出
 * 想想这是为什么？
 * 
 * notify之后，t1必须释放锁，t2退出后，也必须notify，通知t1继续执行
 * 整个通信过程比较繁琐
 * @author mashibing
 */
package com.mashibing.juc.c_020_01_Interview;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;


public class T04_NotifyFreeLock {

   //添加volatile，使t2能够得到通知
   volatile List lists = new ArrayList();

   public void add(Object o) {
      lists.add(o);
   }

   public int size() {
      return lists.size();
   }
   
   public static void main(String[] args) {
      T04_NotifyFreeLock c = new T04_NotifyFreeLock();
      
      final Object lock = new Object();
      
      new Thread(() -> {
         synchronized(lock) {
            System.out.println("t2启动");
            if(c.size() != 5) {
               try {
                  lock.wait();
               } catch (InterruptedException e) {
                  e.printStackTrace();
               }
            }
            System.out.println("t2 结束");
            //通知t1继续执行
            lock.notify();
         }
         
      }, "t2").start();
      
      try {
         TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e1) {
         e1.printStackTrace();
      }

      new Thread(() -> {
         System.out.println("t1启动");
         synchronized(lock) {
            for(int i=0; i<10; i++) {
               c.add(new Object());
               System.out.println("add " + i);
               
               if(c.size() == 5) {
                  // 唤醒t1
                  lock.notify();
                  try {
                      //释放锁，让t2得以执行
                     lock.wait();
                  } catch (InterruptedException e) {
                     e.printStackTrace();
                  }
               }
               
               try {
                  TimeUnit.SECONDS.sleep(1);
               } catch (InterruptedException e) {
                  e.printStackTrace();
               }
            }
         }
      }, "t1").start();
   }
}
```

##### CountDownLatch写法

两道门栓才行，1线程打开2的门栓后，先于2执行下面的逻辑

给1也设门栓，2执行后再打开1的门栓

```java
public class T05_CountDownLatch {

   // 添加volatile，使t2能够得到通知
   volatile List lists = new ArrayList();

   public void add(Object o) {
      lists.add(o);
   }

   public int size() {
      return lists.size();
   }

   public static void main(String[] args) {
      T05_CountDownLatch c = new T05_CountDownLatch();

      CountDownLatch latch = new CountDownLatch(1);

      new Thread(() -> {
         System.out.println("t2启动");
         if (c.size() != 5) {
            try {
               latch.await();
               
               //也可以指定等待时间
               //latch.await(5000, TimeUnit.MILLISECONDS);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }
         }
         System.out.println("t2 结束");

      }, "t2").start();

      try {
         TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e1) {
         e1.printStackTrace();
      }

      new Thread(() -> {
         System.out.println("t1启动");
         for (int i = 0; i < 10; i++) {
            c.add(new Object());
            System.out.println("add " + i);

            if (c.size() == 5) {
               // 打开门闩，让t2得以执行
               latch.countDown();
            }

            /*try {
               TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }*/
         }

      }, "t1").start();

   }
}
```

##### LockSupport_WithoutSleep

t2直接park，t1判断自己size到5的时候，unpark打开t2，自己park住。t2执行完再unpark打开t1

​	睡眠会给线程2缓冲的时间

```java
public class T07_LockSupport_WithoutSleep {

   // 添加volatile，使t2能够得到通知
   volatile List lists = new ArrayList();

   public void add(Object o) {
      lists.add(o);
   }

   public int size() {
      return lists.size();
   }

   static Thread t1 = null, t2 = null;

   public static void main(String[] args) {
      T07_LockSupport_WithoutSleep c = new T07_LockSupport_WithoutSleep();

      t1 = new Thread(() -> {
         System.out.println("t1启动");
         for (int i = 0; i < 10; i++) {
            c.add(new Object());
            System.out.println("add " + i);

            if (c.size() == 5) {
               LockSupport.unpark(t2);
               LockSupport.park();
            }
         }
      }, "t1");

      t2 = new Thread(() -> {
         //System.out.println("t2启动");
         //if (c.size() != 5) {

            LockSupport.park();

         //}
         System.out.println("t2 结束");
         LockSupport.unpark(t1);


      }, "t2");

      t2.start();
      t1.start();





   }
}
```

##### 消费者问题

```java
/**
 * 面试题：写一个固定容量同步容器，拥有put和get方法，以及getCount方法，
 * 能够支持2个生产者线程以及10个消费者线程的阻塞调用
 * 
 * 使用wait和notify/notifyAll来实现
 * 
 * @author mashibing
 */
package com.mashibing.juc.c_021_01_interview;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;

public class MyContainer1<T> {
   final private LinkedList<T> lists = new LinkedList<>();
   final private int MAX = 10; //最多10个元素
   private int count = 0;
   
   
   public synchronized void put(T t) {
      while(lists.size() == MAX) { //想想为什么用while而不是用if？
         try {
            this.wait(); //effective java
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }
      
      lists.add(t);
      ++count;
      this.notifyAll(); //通知消费者线程进行消费
   }
   
   public synchronized T get() {
      T t = null;
      while(lists.size() == 0) {
         try {
            this.wait();
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }
      t = lists.removeFirst();
      count --;
      this.notifyAll(); //通知生产者进行生产
      return t;
   }
   
   public static void main(String[] args) {
      MyContainer1<String> c = new MyContainer1<>();
      //启动消费者线程
      for(int i=0; i<10; i++) {
         new Thread(()->{
            for(int j=0; j<5; j++) System.out.println(c.get());
         }, "c" + i).start();
      }
      
      try {
         TimeUnit.SECONDS.sleep(2);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
      
      //启动生产者线程
      for(int i=0; i<2; i++) {
         new Thread(()->{
            for(int j=0; j<25; j++) c.put(Thread.currentThread().getName() + " " + j);
         }, "p" + i).start();
      }
   }
}
```

##### 消费者2

```java
/**
 * 面试题：写一个固定容量同步容器，拥有put和get方法，以及getCount方法，
 * 能够支持2个生产者线程以及10个消费者线程的阻塞调用
 * 
 * 使用wait和notify/notifyAll来实现
 * 
 * 使用Lock和Condition来实现
 * 对比两种方式，Condition的方式可以更加精确的指定哪些线程被唤醒
 * 
 * @author mashibing
 */
package com.mashibing.juc.c_021_01_interview;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class MyContainer2<T> {
   final private LinkedList<T> lists = new LinkedList<>();
   final private int MAX = 10; //最多10个元素
   private int count = 0;
   
   private Lock lock = new ReentrantLock();
   //Condition 本质就是等待队列
   private Condition producer = lock.newCondition();
   private Condition consumer = lock.newCondition();
   
   public void put(T t) {
      try {
         lock.lock();
         while(lists.size() == MAX) { //想想为什么用while而不是用if？
            producer.await();
         }
         
         lists.add(t);
         ++count;
         consumer.signalAll(); //通知消费者线程进行消费
      } catch (InterruptedException e) {
         e.printStackTrace();
      } finally {
         lock.unlock();
      }
   }
   
   public T get() {
      T t = null;
      try {
         lock.lock();
         while(lists.size() == 0) {
            consumer.await();
         }
         t = lists.removeFirst();
         count --;
         producer.signalAll(); //通知生产者进行生产
      } catch (InterruptedException e) {
         e.printStackTrace();
      } finally {
         lock.unlock();
      }
      return t;
   }
   
   public static void main(String[] args) {
      MyContainer2<String> c = new MyContainer2<>();
      //启动消费者线程
      for(int i=0; i<10; i++) {
         new Thread(()->{
            for(int j=0; j<5; j++) System.out.println(c.get());
         }, "c" + i).start();
      }
      
      try {
         TimeUnit.SECONDS.sleep(2);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
      
      //启动生产者线程
      for(int i=0; i<2; i++) {
         new Thread(()->{
            for(int j=0; j<25; j++) c.put(Thread.currentThread().getName() + " " + j);
         }, "p" + i).start();
      }
   }
}
```

### volatile

堆内存共享，各线程有自己的工作内存，使用堆内存变量时先取回自己的工作内存，更改后写回堆内存的时间不好控制。导致线程之间不可见

* 保证线程可见性

  * 不同线程取到公共堆中的值，先在copy到自己的空间改变，再写回公共空间，而另一个线程不知道何时发生改变，双方数据不能及时共享，及不可见。

    加了volatile后，一个线程改变变量后，另一个线程会马上发现

  * 实际底层根据了cpu的 MESI 缓存一致性协议
    毕竟如果读取过程中又被改了，还是需要硬件层面来保证

* 禁止指令重排序

  * 创建对象流程：
    * 1.申请内存，如给a赋值为0，此时不为null
    * 2.给a赋值，如a=8
    * 3.将a赋值给INSTANCE
  * 如果先指向了a=0，此时不是空值了，另一个线程判断INSTANCE不为空，直接用了a=0，大问题。但一般也不会出现

没有把握尽量不要用volatile，用也是修饰小一点的东西，别修饰引用对象

```java
public class T1_H {
    /*volatile*/ static boolean time = true;
    public void m(){
        System.out.println("start");
        while (time){


        }
        System.out.println("stop");
    }

    public static void main(String[] args) {
        T1_H t1_h = new T1_H();
        new Thread(t1_h::m).start();
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //更改了主线程的time,但另一个线程不知道
        time = false;
    }

}
```

```java
public class Mgr06 {
    private static volatile Mgr06 INSTANCE; //JIT

    private Mgr06() {
    }

    public static Mgr06 getInstance() {
        if (INSTANCE == null) {
            //双重检查
            synchronized (Mgr06.class) {
                if(INSTANCE == null) {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    INSTANCE = new Mgr06();
                }
            }
        }
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for(int i=0; i<100; i++) {
            new Thread(()->{
                System.out.println(Mgr06.getInstance().hashCode());
            }).start();
        }
    }
}
```

### 锁优化

* 采用细粒度的锁，尽量不要将无需锁的代码放入其中
* 当锁在一片区域比较密集的时候，可以粗化锁



* 锁定某对象o，如果o的属性发生改变，不影响锁的使用。

但是如果o变为另一个对象，锁将失效（另一个对象的前两位不相同）

因此应该	final Object o = new Object();



### **锁的选择**

自旋时是占用cpu而不占用系统内核的。

执行时间长，线程数量多用重量级（OS）锁

时间短，线程数量少用自旋锁（比如1个在执行，999个在自旋等待）

* * 

实际用起来synchronize也不慢

### 2.3 JUC-java.util .concurrent

---

#### Runnable和Callable

#### Execute和Submit

submit添加新任务的方法会有返回值，用于逻辑判断

```java
//Callable接口的Task
public class testc {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for(int i =0;i<10;i++){
            Future<String> submit = executorService.submit(new Taskc());
            String str = null;
            try {
                str = submit.get();
                System.out.println(str);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
//Runnable接口的Task
public class testR {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for(int i =0;i<100;i++){
            executorService.execute(new Taskr());
        }
        executorService.shutdown();

    }
```

## 锁原码

### 要点

·跑不起来不读
·解决问题就好-目的性
·一条线索到底
·无关细节略过
·一般不读静态
·一般动态读法



模板方法：父类只提供方法，子类去实现细节



### AQS

上面所有锁的底层实现

使用了模板方法、回调函数

默认非公平锁的实现

![AQS](E:\deng\deng\MD\jvm\img\AQS.png)

![AQS](E:\deng\deng\MD\image\AQS.png)

上来调用tryquire，拿不到锁就进队列

state是一个volatle的int类型，用于判断是否加锁，并且设置state的方法除了有setState,还有compareAndSetState

所以有人称AQS底层就是volatile+CAS

state会根据子类不同实现，有着不同的意义

state内部维护着一个队列，是AQS的内部类
node中装着thread,并且node是个双向链表，没抢到锁的通过addwaiter方法进入队列
(addWaiter方法是一个死循环，cas试图加在链表后面，因为不是锁整个链表，所以效率还行)

拿到后判断，如果c == 0 ,通过cas将state其变为1,并将当前线程设置为独占线程



==高并发5 20:00

#### VarHandle

1.普通属性进行原子操作

2.比反射快，直接操纵二进制码



### ThreadLocal

线程独有，线程B向threadLocal设值，线程A无法从threadLocal中读到。

* `threadLocal.set()`
  * Thread.currentThread.map.set(ThreadLocal,value)
  * 值被设置到了一个map中，key是调用该方法的当前线程，是设置到了当前线程的map，所以另一个线程get不到
  * 这个map就是个weakHahsMap
* 在声明式事务中
  * 第一个拿到connection后，放入ThreadLocal中，后续的从local拿，这样不会使得拿到不同的connection。
* map中的key是弱引用，value是强引用
  * tl强引用指向ThreadLocal，并且是作为map中的key，并且key是弱引用指向ThreadLocal
  * 当tl对ThreadLocal的引用消失时，key的引用会自动消失，否则会造成内存泄露
  * 即便如此还还会有内存泄露（如下图中小字），因此tl不用的时候要`tl.remove()`掉
* 

![弱引用](E:\deng\deng\MD\image\弱引用.png)

```java
/**
 * ThreadLocal线程局部变量
 *
 * ThreadLocal是使用空间换时间，synchronized是使用时间换空间
 * 比如在hibernate中session就存在与ThreadLocal中，避免synchronized的使用
 *
 * 运行下面的程序，理解ThreadLocal
 * 
 * @author 马士兵
 */
package com.mashibing.juc.c_022_RefTypeAndThreadLocal;

import java.util.concurrent.TimeUnit;

public class ThreadLocal2 {
   //volatile static Person p = new Person();
   static ThreadLocal<Person> tl = new ThreadLocal<>();
   
   public static void main(String[] args) {
            
      new Thread(()->{
         try {
            TimeUnit.SECONDS.sleep(2);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
         
         System.out.println(tl.get());
      }).start();
      
      new Thread(()->{
         try {
            TimeUnit.SECONDS.sleep(1);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
         tl.set(new Person());
      }).start();
   }
   
   static class Person {
      String name = "zhangsan";
   }
}
```





## 引用

### 强引用

普通引用就是强引用

垃圾回收会调用`finalize()`，重写它来观察什么时候对对象进行了回收。

```java
public class T01_NormalReference {
    public static void main(String[] args) throws IOException {
        M m = new M();
        m = null;
        //因为改变了m指向，new M()变成空了
        System.gc(); //DisableExplicitGC
        
        //这个方法可以阻塞住线程
        System.in.read();
    }
}
```

### 软引用

做缓存用，内存够你就在放着，用到时直接拿。内存不够时再把你GC掉(面试用）

memachace、tomcat中用到了

```java
/**
 * 软引用
 * 软引用是用来描述一些还有用但并非必须的对象。
 * 对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围进行第二次回收。
 * 如果这次回收还没有足够的内存，才会抛出内存溢出异常。
 * -Xmx20M
 */
package com.mashibing.juc.c_022_RefTypeAndThreadLocal;

import java.lang.ref.SoftReference;

public class T02_SoftReference {
    public static void main(String[] args) {
        SoftReference<byte[]> m = new SoftReference<>(new byte[1024*1024*10]);
        //m = null;
        System.out.println(m.get());
        System.gc();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(m.get());

        //再分配一个数组，heap将装不下，这时候系统会垃圾回收，先回收一次，如果不够，会把软引用干掉
        byte[] b = new byte[1024*1024*15];
        System.out.println(m.get());
    }
}

//软引用非常适合缓存使用
```

### 弱引用

GC发现后会直接干掉

一般用在容器中，和强引用指向同一对象，强引用消失时，弱引用一并回收



### 虚引用

高并发5 2h

管理对外内存，是给写JVM的人用的，对调用api的人无意义，因此get为null;

## 线程池



### 基础概念

#### Executor

将线程定义和线程执行分开

#### Callable

与Runnable类似，但callable有返回值

只能执行任务，返回值需要存储在Future中

```java
public class T03_Callable {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Callable<String> c = new Callable() {
            @Override
            public String call() throws Exception {
                return "Hello Callable";
            }
        };
        //将callable扔进下面这个线程池
        ExecutorService service = Executors.newCachedThreadPool();
        Future<String> future = service.submit(c); //异步

        //lamda写法
        Future<Integer> future2 = service.submit(()->{
            TimeUnit.MILLISECONDS.sleep(500);
            return 1;
        }); //异步

        System.out.println(future.get());//阻塞

        service.shutdown();
    }

}
```

#### Future

存储执行产生的将来的结果

#### FutureTask

既是Runnable又是Future

既能执行，又能存储结果

是个异步操作

```java
public class T06_00_Future {
   public static void main(String[] args) throws InterruptedException, ExecutionException {
      FutureTask<Integer> task = new FutureTask<>(()->{
         TimeUnit.MILLISECONDS.sleep(500);
         return 500;
      });
      new Thread(task).start();
      System.out.println(task.get());
   }
}
```

#### ==CompletableFuture

对于多个Future的管理，需要查三家网站的同一产品价格，最后汇总展示

allOf(A,B,C)	

```java
/**
 * 假设你能够提供一个服务
 * 这个服务查询各大电商网站同一类产品的价格并汇总展示
 * @author 马士兵 http://mashibing.com
 */

package com.mashibing.juc.c_026_01_ThreadPool;

import java.io.IOException;
import java.util.Random;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class T06_01_CompletableFuture {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long start, end;

        /*start = System.currentTimeMillis();

        priceOfTM();
        priceOfTB();
        priceOfJD();

        end = System.currentTimeMillis();
        System.out.println("use serial method call! " + (end - start));*/

        start = System.currentTimeMillis();

        //执行异步任务。 
        CompletableFuture<Double> futureTM = CompletableFuture.supplyAsync(()->priceOfTM());
        CompletableFuture<Double> futureTB = CompletableFuture.supplyAsync(()->priceOfTB());
        CompletableFuture<Double> futureJD = CompletableFuture.supplyAsync(()->priceOfJD());                     

        //必须下面三个任务都完成，才输出结果
        CompletableFuture.allOf(futureTM, futureTB, futureJD).join();

 /*       CompletableFuture.supplyAsync(()->priceOfTM())
                .thenApply(String::valueOf)
                .thenApply(str-> "price " + str)
                .thenAccept(System.out::println);

*/
        end = System.currentTimeMillis();
        System.out.println("use completable future! " + (end - start));

        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //模拟去其他网站爬虫的时间，返回价格
    private static double priceOfTM() {
        delay();
        return 1.00;
    }

    private static double priceOfTB() {
        delay();
        return 2.00;
    }

    private static double priceOfJD() {
        delay();
        return 3.00;
    }

    /*private static double priceOfAmazon() {
        delay();
        throw new RuntimeException("product not exist!");
    }*/
    

    private static void delay() {
        int time = new Random().nextInt(500);
        try {
            TimeUnit.MILLISECONDS.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.printf("After %s sleep!\n", time);
    }
}
```

### Executors-线程池的工厂

### ThreadPoolExecutor

里面放置着各个线程和各个任务，各个线程去同一个队列拿任务

![ThreadPool原理](E:\deng\deng\MD\image\ThreadPool原理.png)

线程池的执行器

~~~java
public ThreadPoolExecutor(int corePoolSize,  
                          int maximumPoolSize,  
                          long keepAliveTime,  
                          TimeUnit unit,  
                          BlockingQueue<Runnable> blockingQueue,  
                          ThreadFactory threadFactory,  
                          RejectedExecutionHandler handler)
~~~

**corePoolSize**：就是线程池中的核心线程数量 

**maximumPoolSize**：就是线程池中可以容纳的最大线程的数量

**keepAliveTime**：线程池中除了核心线程之外的其他的最长可以保留的时间

**TimeUnit.SECONDS**：时间单位

**workQueue（BlockingQueue）**：任务队列，任务可以储存在任务队列中等待被执行

**threadFactory**：就是创建线程的线程工厂（可自定义创建线程方式，默认的创建线程方式）

**handler**(RejectStrategy)：拒绝策略，jdk默认提供了4种，可自定义

> 阿里规范线程必须通过线程池提供，不允许自行显式创建

![为什么要线程池](E:\deng\image\为什么要线程池.png)

![线程池工作原理](E:\deng\image\线程池工作原理.png)









### 自义线程池

估算后再压测，确定具体设定

![调整线程池大小](E:\deng\deng\MD\image\调整线程池大小.png)



```java
public class 自定义线程池 {
    public static void main(String[] args) {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2,5,60, TimeUnit.SECONDS,new ArrayBlockingQueue(5));
                                    //只用一次的内部类
        threadPoolExecutor.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                System.out.println(Math.random());
                return null;
            }
        });
    }
}
```

### 功能线程池

不建议使用，因为LinkedBlockingQueue有上界，导致OOM

还有要自定义线程的名称，方便排查问题



**FixedThreadPool**

- **特点**：只有核心线程，线程数量固定，执行完立即回收，任务队列为链表结构的有界队列。
- **应用场景**：控制线程最大并发数。

**CachedThreadPool:**	可缓存的线程池

- **特点**：无核心线程，非核心线程数量无限，执行完闲置60s后回收，任务队列为不存储元素的阻塞队列。SynchronousQueue
- **应用场景**：执行大量、耗时少的任务。4444

**SecudleThreadPool:**

- **特点**：核心线程数量固定，非核心线程数量无限，执行完闲置10ms后回收，任务队列为延时阻塞队列。
- **应用场景**：执行定时或周期性的任务。

**SingleThreadPool:**	

- **特点**：只有1个核心线程，无非核心线程，执行完立即回收，任务队列为链表结构的有界队列。
- **应用场景**：不适合并发但可能引起IO阻塞性及影响UI线程响应的操作，如数据库操作、文件操作等

![img](https://img-blog.csdnimg.cn/20190721095954211.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM1NDExNDA=,size_16,color_FFFFFF,t_70)



后三种方法实际上只是ExecutorService的不同构造方法

ExecutorService继承自Executor

ThreadPoolExecutor实现了ExecutorService接口



Executors.newCachedThreadPool

```java
//newScheduledThreadPool线程池
public class ScheduledDemo {
    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(3);
        System.out.println(System.currentTimeMillis());
        //创建内部类
        scheduledExecutorService.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println("延迟3秒");
                System.out.println(System.currentTimeMillis());
            }
            //延时1秒，3秒执行一次
        },1,3, TimeUnit.SECONDS);
//        scheduledExecutorService.shutdown();
        System.out.println(System.currentTimeMillis());
    }
}
```

线程池的生命周期

![线程池的生命周期](E:\deng\image\线程池的生命周期.png)

![线程池的生命周期2](E:\deng\image\线程池的生命周期2.png)

### 拒绝策略

以下是jdk默认策略，实际一般不用，用自定义的，保存到mq等等。

**AbortPolicy:**不执行新任务，直接抛出异常，提示线程池已满

**DisCardPolicy:**不执行新任务，也不抛出异常

**DisCardOldSetPolicy:**将排队时间最久的（第一个）替换为当前新进来的任务执行

**CallerRunsPolicy:**多出的任务直接调用execute来执行。

##### Execute方法执行逻辑

![Execute方法执行逻辑](E:\deng\image\Execute方法执行逻辑.png)





## ForkJoinPool

这是一个线程池，任务需要扔进去，fjp.execute(task)

大任务分散后汇总任务

RecursiveAction

#### 原理

每个线程维护自己的队列，执行完就去其它队列尾部拿。而ThreadPoolExecutor是多个线程用同一个队列

线程执行自己的任务时不需要加锁，而执行其它线程任务时需要加锁

![forkjoin原理](E:\deng\deng\MD\image\forkjoin原理.png)

#### ForkJoinPool

加总一个数组的总和，用forkjoin

创建一个加总任务继承RecursiveTask<Long> （带返回值）

```java

package com.mashibing.juc.c_026_01_ThreadPool;

import java.io.IOException;
import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;
import java.util.concurrent.RecursiveTask;

public class T12_ForkJoinPool {
   static int[] nums = new int[1000000];
   //每个线程计算的最大数量
   static final int MAX_NUM = 50000;
   static Random r = new Random();
   
   static {
      for(int i=0; i<nums.length; i++) {
         nums[i] = r.nextInt(100);
      }
      
      System.out.println("---" + Arrays.stream(nums).sum()); //stream api
   }
   

   static class AddTask extends RecursiveAction {

      int start, end;

      AddTask(int s, int e) {
         start = s;
         end = e;
      }

      @Override
      protected void compute() {

         if(end-start <= MAX_NUM) {
            long sum = 0L;
            for(int i=start; i<end; i++) sum += nums[i];
            System.out.println("from:" + start + " to:" + end + " = " + sum);
         } else {

            int middle = start + (end-start)/2;

            AddTask subTask1 = new AddTask(start, middle);
            AddTask subTask2 = new AddTask(middle, end);
            subTask1.fork();
            subTask2.fork();
         }


      }

   }

   
   static class AddTaskRet extends RecursiveTask<Long> {
      
      private static final long serialVersionUID = 1L;
      int start, end;
      
      AddTaskRet(int s, int e) {
         start = s;
         end = e;
      }

      @Override
      protected Long compute() {
         
         if(end-start <= MAX_NUM) {
            long sum = 0L;
            for(int i=start; i<end; i++) sum += nums[i];
            return sum;
         } 
         
         int middle = start + (end-start)/2;
         
         AddTaskRet subTask1 = new AddTaskRet(start, middle);
         AddTaskRet subTask2 = new AddTaskRet(middle, end);
         subTask1.fork();
         subTask2.fork();
         
         return subTask1.join() + subTask2.join();
      }
      
   }
   
   public static void main(String[] args) throws IOException {
      /*ForkJoinPool fjp = new ForkJoinPool();
      AddTask task = new AddTask(0, nums.length);
      fjp.execute(task);*/

      T12_ForkJoinPool temp = new T12_ForkJoinPool();

      ForkJoinPool fjp = new ForkJoinPool();
      AddTaskRet task = new AddTaskRet(0, nums.length);
      fjp.execute(task);
      long result = task.join();
      System.out.println(result);
      
      //System.in.read();
      
   }
}
```

#### ParallelStramAPI

```JAVA
package com.mashibing.juc.c_026_01_ThreadPool;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class T13_ParallelStreamAPI {
   public static void main(String[] args) {
      List<Integer> nums = new ArrayList<>();
      Random r = new Random();
      for(int i=0; i<10000; i++) nums.add(1000000 + r.nextInt(1000000));
      
      //System.out.println(nums);
      
      long start = System.currentTimeMillis();
      nums.forEach(v->isPrime(v));
      long end = System.currentTimeMillis();
      System.out.println(end - start);
      
      //使用parallel stream api
      
      start = System.currentTimeMillis();
      nums.parallelStream().forEach(T13_ParallelStreamAPI::isPrime);
      end = System.currentTimeMillis();
      
      System.out.println(end - start);
   }
   
   static boolean isPrime(int num) {
      for(int i=2; i<=num/2; i++) {
         if(num % i == 0) return false;
      }
      return true;
   }
}
```





## 容器

![容器](E:\deng\deng\MD\image\容器.png)

Queue是为了高并发准备的

Vector Hashtable 自带锁，但现在不用

T023 FromHashtableToCHM

### 容器历史

* vector、hashtable 自带锁 （有缺陷，现在不用）
* hashmap 去除锁，增加效率
* synchronizeHashMap	增加加锁版本（性能也不比hashtable高多少）
* concurrentHashMap	多线程专用
  * 读效率很高，但写效率跟上面几个差不多
  * 但是没有concurrentTreeMap,因为树实现cas太复杂了
  * 但又需要一个排好序的容器，因此产生了 如下
* concurrentSkipListMap 跳表
  因为concurrentTreeMap不好实现，又需要一个排好序的便于查找
  先从顶层查找，依次向下
  * ![跳表](E:\deng\deng\MD\image\跳表.png)



queue和list区别

* queue和list的区别就体现在这里
  queue提供了这些对多线程友好的操作



#### 性能测试demo

```java
package com.mashibing.juc.c_023_02_FromHashtableToCHM;

import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

public class T04_TestConcurrentHashMap {

    static Map<UUID, UUID> m = new ConcurrentHashMap<>();

    static int count = Constants.COUNT;
    static UUID[] keys = new UUID[count];
    static UUID[] values = new UUID[count];
    static final int THREAD_COUNT = Constants.THREAD_COUNT;

    
    //先整好kv键值对
    static {
        for (int i = 0; i < count; i++) {
            keys[i] = UUID.randomUUID();
            values[i] = UUID.randomUUID();
        }
    }

    static class MyThread extends Thread {
        int start;
        int gap = count/THREAD_COUNT;

        public MyThread(int start) {
            this.start = start;
        }

        @Override
        public void run() {
            for(int i=start; i<start+gap; i++) {
                m.put(keys[i], values[i]);
            }
        }
    }

    public static void main(String[] args) {

        long start = System.currentTimeMillis();

        Thread[] threads = new Thread[THREAD_COUNT];

        for(int i=0; i<threads.length; i++) {
            threads[i] =
            new MyThread(i * (count/THREAD_COUNT));
        }

        for(Thread t : threads) {
            t.start();
        }

        for(Thread t : threads) {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        long end = System.currentTimeMillis();
        System.out.println(end - start);

        System.out.println(m.size());

        //-----------------------------------

        start = System.currentTimeMillis();
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(()->{
                for (int j = 0; j < 10000000; j++) {
                    m.get(keys[10]);
                }
            });
        }

        for(Thread t : threads) {
            t.start();
        }

        for(Thread t : threads) {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        end = System.currentTimeMillis();
        System.out.println(end - start);
    }
}
```

#### Vector测试

vector所谓的线程安全指的是调用容器方法加锁，如`size()`时，访问时安全的。但是当卖到最后一张票时，各个线程判断size是否为空时没有争用，分别进入睡眠，再去取票，产生冲突。

```java
/**
 * 有N张火车票，每张票都有一个编号
 * 同时有10个窗口对外售票
 * 请写一个模拟程序
 * 
 * 分析下面的程序可能会产生哪些问题？
 *  
 * 使用Vector或者Collections.synchronizedXXX
 * 分析一下，这样能解决问题吗？
 * 
 * @author 马士兵
 */
package com.mashibing.juc.c_024_FromVectorToQueue;

import java.util.Vector;
import java.util.concurrent.TimeUnit;

public class TicketSeller2 {
   static Vector<String> tickets = new Vector<>();
   
   
   static {
      for(int i=0; i<1000; i++) tickets.add("票 编号：" + i);
   }
   
   public static void main(String[] args) {
      
      for(int i=0; i<10; i++) {
         new Thread(()->{
            while(tickets.size() > 0) {
               
               try {
                  TimeUnit.MILLISECONDS.sleep(10);
               } catch (InterruptedException e) {
                  e.printStackTrace();
               }
               
               
               System.out.println("销售了--" + tickets.remove(0));
            }
         }).start();
      }
   }
}
```

#### Queue卖票测试

poll是一个原子操作，内部是cas实现的

```java
package com.mashibing.juc.c_0test;

import java.util.Queue;
import java.util.concurrent.ConcurrentLinkedQueue;

public class t2 {
    static Queue<String> tickets = new ConcurrentLinkedQueue<>();
    static{
        for (int i = 0; i < 100; i++) {
            tickets.add("编号" + i);
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                while (true){
//poll是一个原子操作，判断tickets是否为空，内部是cas实现的
                    String poll = tickets.poll();
                    if (poll == null) break;
                    System.out.println("销售了" + poll);
                }
            }).start();
        }
    }


}
```





### 阻塞队列-BlockingQueue

由Vector进一步推出queue

天然的支持多线程存取，因为自带阻塞，不需要自己写程序的时候用synchronize之流加锁

#### 方法

##### queue的方法

offer()		//add，返回Boolean

peek()	//取，不remove

poll()		//取，并remove

##### blockQueue在此基础上加的方法

put		//装不进去就先阻塞

take		//取阻塞



* queue和list的区别就体现在这里
  queue提供了这些对多线程友好的操作



#### 种类

##### LinkedBlockingQueue

LinkedBlockingQuee实现取票

实际就是`take()`自带阻塞，就可以实现卖票。线程数最大值是INTEGER最大值

```java
public class T08_LinkedBlockingQueue {
    static BlockingQueue<String> tickets = new LinkedBlockingQueue<>();
    static Random r = new Random();

    public static void main(String[] args) {
        new Thread(()->{
            for (int i = 0; i < 1000; i++) {
                try {
                    tickets.put("code"+ i);
                    TimeUnit.MILLISECONDS.sleep(r.nextInt(1000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    while (true){
                        System.out.println(Thread.currentThread().getName()+ "take off the ===="+tickets.take());
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }

    }



}
```

#### ==DelayQueue

用于时间调度任务

```java
package com.mashibing.juc.c_025;

import java.util.Calendar;
import java.util.Random;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.DelayQueue;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

public class T07_DelayQueue {

   static BlockingQueue<MyTask> tasks = new DelayQueue<>();

   static Random r = new Random();
   
   static class MyTask implements Delayed {
      String name;
      long runningTime;
      
      MyTask(String name, long rt) {
         this.name = name;
         this.runningTime = rt;
      }

      @Override
      public int compareTo(Delayed o) {
         if(this.getDelay(TimeUnit.MILLISECONDS) < o.getDelay(TimeUnit.MILLISECONDS))
            return -1;
         else if(this.getDelay(TimeUnit.MILLISECONDS) > o.getDelay(TimeUnit.MILLISECONDS)) 
            return 1;
         else 
            return 0;
      }

      @Override
      public long getDelay(TimeUnit unit) {
         
         return unit.convert(runningTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
      }
      
      
      @Override
      public String toString() {
         return name + " " + runningTime;
      }
   }

   public static void main(String[] args) throws InterruptedException {
      long now = System.currentTimeMillis();
      MyTask t1 = new MyTask("t1", now + 1000);
      MyTask t2 = new MyTask("t2", now + 2000);
      MyTask t3 = new MyTask("t3", now + 1500);
      MyTask t4 = new MyTask("t4", now + 2500);
      MyTask t5 = new MyTask("t5", now + 500);
      
      tasks.put(t1);
      tasks.put(t2);
      tasks.put(t3);
      tasks.put(t4);
      tasks.put(t5);
      
      System.out.println(tasks);
      
      for(int i=0; i<5; i++) {
         System.out.println(tasks.take());
      }
   }
}
```

#### SynchronizeQueue

这个队列是空的，不能在里面放东西，只能1先take()在那阻塞，等着2去take()。相当于两个线程手把手交换。

```java
package com.mashibing.juc.c_025;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.SynchronousQueue;

public class T08_SynchronizeQueue { //容量为0
   public static void main(String[] args) throws InterruptedException {
      BlockingQueue<String> strs = new SynchronousQueue<>();
      
      new Thread(()->{
         try {
            System.out.println(strs.take());
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }).start();

      strs.put("aaa"); //阻塞等待消费者消费
      //strs.put("bbb");
      //strs.add("aaa");
      System.out.println(strs.size());
   }
}
```

#### TransferQueue

strs.transfer("aaa"),必须有结果才会离开

MQ底层就是用这种逻辑，比如，确认用户的订单得到了处理
当你自己去实现mq的效果时就需要用这个了

```java
package com.mashibing.juc.c_025;

import java.util.concurrent.LinkedTransferQueue;

public class T09_TransferQueue {
   public static void main(String[] args) throws InterruptedException {
      LinkedTransferQueue<String> strs = new LinkedTransferQueue<>();
      
      new Thread(() -> {
         try {
            System.out.println(strs.take());
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }).start();
      
      strs.transfer("aaa");
      
      //strs.put("aaa");


      /*new Thread(() -> {
         try {
            System.out.println(strs.take());
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }).start();*/


   }
}
```

#### 应用案例（交替打印）

##### synchronized方法

```
package com.mashibing.juc.c_026_00_interview.A1B2C3;


public class T06_00_sync_wait_notify {
    public static void main(String[] args) {
        final Object o = new Object();

        char[] aI = "1234567".toCharArray();
        char[] aC = "ABCDEFG".toCharArray();

        new Thread(()->{
            synchronized (o) {
                for(char c : aI) {
                    System.out.print(c);
                    try {
                        o.notify();
                        o.wait(); //让出锁
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                o.notify(); //必须，否则无法停止程序
            }

        }, "t1").start();

        new Thread(()->{
            synchronized (o) {
                for(char c : aC) {
                    System.out.print(c);
                    try {
                        o.notify();
                        o.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                o.notify();
            }
        }, "t2").start();
    }
}

//如果我想保证t2在t1之前打印，也就是说保证首先输出的是A而不是1，这个时候该如何做？
```

##### Locksupport

```java
package com.mashibing.juc.c_026_00_interview.A1B2C3;


import java.util.concurrent.locks.LockSupport;

//Locksupport park 当前线程阻塞（停止）
//unpark(Thread t)

public class T02_00_LockSupport {


    static Thread t1 = null, t2 = null;

    public static void main(String[] args) throws Exception {
        char[] aI = "0123456789".toCharArray();
        char[] aC = "ABCDEFGHIJ".toCharArray();

        t1 = new Thread(() -> {

                for(char c : aI) {
                    System.out.print(c);
                    LockSupport.unpark(t2); //叫醒T2
                    LockSupport.park(); //T1阻塞
                }

        }, "t1");

        t2 = new Thread(() -> {

            for(char c : aC) {
                LockSupport.park(); //t2阻塞
                System.out.print(c);
                LockSupport.unpark(t1); //叫醒t1
            }

        }, "t2");

        t1.start();
        t2.start();
    }
}
```

cas写法

```java
package com.mashibing.juc.c_026_00_interview.A1B2C3;


public class T03_00_cas {

    enum ReadyToRun {T1, T2}

    static volatile ReadyToRun r = ReadyToRun.T1; //思考为什么必须volatile

    public static void main(String[] args) {

        char[] aI = "1234567".toCharArray();
        char[] aC = "ABCDEFG".toCharArray();

        new Thread(() -> {

            for (char c : aI) {
                while (r != ReadyToRun.T1) {}
                System.out.print(c);
                r = ReadyToRun.T2;
            }

        }, "t1").start();

        new Thread(() -> {

            for (char c : aC) {
                while (r != ReadyToRun.T2) {}
                System.out.print(c);
                r = ReadyToRun.T1;
            }
        }, "t2").start();
    }
}
```

##### transferQueue

```java
package com.mashibing.juc.c_026_00_interview.A1B2C3;

import java.util.concurrent.LinkedTransferQueue;
import java.util.concurrent.TransferQueue;

public class T13_TransferQueue {
    public static void main(String[] args) {
        char[] aI = "1234567".toCharArray();
        char[] aC = "ABCDEFG".toCharArray();

        TransferQueue<Character> queue = new LinkedTransferQueue<Character>();
        new Thread(()->{
            try {
                for (char c : aI) {
                    System.out.print(queue.take());
                    queue.transfer(c);
                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "t1").start();

        new Thread(()->{
            try {
                for (char c : aC) {
                    queue.transfer(c);
                    System.out.print(queue.take());
                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "t2").start();
    }
}
```







乐观锁

