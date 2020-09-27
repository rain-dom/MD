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

### **实现方式**

* 继承*Tread*类
* 实现*Runable*接口
* 实现callable接口（有返回值）

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

（一）、等待阻塞：运行的线程执行**wait()**方法，JVM会把该线程放入等待池中。(wait会释放持有的锁)

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

先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把 CPU 的占有权交给此线程，否则，继续运行原来的线程。

但有可能继续抢占到cpu

**wait()**

Obj.wait()，与Obj.notify()必须要与synchronized(Obj)一起使用,是针对已经获取了Obj锁进行操作

Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于Object.wait()在释放CPU同时，释放了对象锁的控制。

**sleep（）**

 sleep()是Thread类的Static(静态)的方法；因此他不能改变对象的机锁，所以当在一个Synchronized块中调用Sleep()方法是，线程虽然休眠了，但是对象的机锁并木有被释放，其他线程无法访问这个对象



### 锁机制

#### synchronize锁



锁的是对象不是代码

this XX.class

其他线程看被锁对象的前两位（64位）

```java
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

##### 优化

采用细粒度的锁，尽量不要将无需锁的代码放入其中

当锁比较密集的时候，可以粗化锁



锁定某对象o，如果o的属性发生改变，不影响锁的使用。

但是如果o变为另一个对象，锁将失效（另一个对象的前两位不相同）

因此应该	final Object o = new Object();

##### 锁的升级

​	首先是偏向锁，如果线程争用，升级为自旋锁（占用cpu）

​	另一个线程循环判断是否能拿到锁

​	偏向锁不是加锁，而是记住第一个拿到的线程，倾向于他先执行

​	10次后升级为重量级锁

执行时间长，线程数量多用重量级（OS）锁

时间短，线程数量少用自旋锁



#### volatile

* 保证线程可见性
  * 不同线程取到公共堆中的值，先在自己的空间改变，双方数据不能及时共享，及不可见
  * MESI 缓存一致性协议
* 禁止指令重排序
  * �үO�˒�O�H�&u�Œ�����\�FM��|��£���J���K�K�3�Z�� XZ�����G�.�y��]=nL��ɜߥ�>5<�O�ҔJ*�(��,Y%�dH (    eZ?Y����9�VQ{55�?���/�g�8<�S}��sJ��e��/��D$���7�QI	�F�������Q��l�e�O�u�b+�����X�/բ�'ˤx @            
ʤc$�P��c�7�:��)R<Qۚ-gR��ß��	qA3�ڃ٣v��I��ڊ�x2�f���"6�3���i��4$�X�� �    0�,�BVK*�(e�ɔdÓ%�-��-7�Kɞ>����oЗ�<}/գ���x>��22W(d�Z|D�}^ο�?���!��vq� Zv� �#����~�^��╵V�.q�������UJ��e7�8i���庻��Aσ�c���,�=J���]Ԕb�q��+�6���d�h�Ec���R������n��[O��m�(�����DzmV�����).�i�q�g���ΡS�'�G��\6���J_�J��GNޏTi҂�I�����n�O5�,�)��"�:���<Ŕ�}n�ث���I�˿#�8rWGg���}m/Ew�Y���a��m���(���]6z���TuZ�\�	%�$oKF;GÏ/m������B���J�$��塪�\��h�)TY�g�č^\:]|uI?{H�6������X>�&�.��T��\u��Q�Z���QqN�Viˮ|��?�*^� �=&p�<݂P��5�3�_o�i�w��9���f�x��߮� ���/�|�"�<7u���W���Y��bK1{5��I$�IrH�S,֓X�{r`�L�{{9�o#iE՞e'�c�L�ӫ:�eV��9=����/�կ;��p��C�W���ɟO��)^�y|>o&rߦ<CL�2S$�4z�=l�Kg�F����3K_��x�g<��v���z��)'��)�~�'�dX�,��:��"�I�*@P      ���r��ZQ�b6��M�xM��E5�?*�Sd�u�m���a''�A�aq>���:���U���T�w���` P            ƿ�����Gn,�Y��)�qx|����9M)O��j���g�5i}h�
җ{��$�S�I�R�^��3���	�M���6T�o���ȒK e�    �Y�%�hU�e�0ʬ�rdK��ؗ���ԙ���<}7���oЗ�<t�z�>e������#%r}M>���{4󪿻4||��ٗ�l������~��\Oׯ��6�t���F�x���c>��Zv��t�U�t�I(�*�Xoe��Z�;�R�Z<P��Y�NJ:>�mUU�l��ɹ7��>E'Vz���Yk�oI�{����yo*c�%���7٧�J�Zߺ4}�F������k8�"���#�i������ˊx�]b��~��ǥ������Jg���z�n.zE�I?ri���fZ��KwV?3�S�J抜jҨ�ԑ�GM����P�Q��Mɼyd�,�I�NGse�J�gKg��j)�զ�Jk⏧��0���T�Qb�z��#�X�;+�u��b�$������kc�����V����:�M2�R����4� �YzW��]j��z�UiW���jS�۪hʖ�amUշ�P�����<�sŞ1�k0����l��Y���:|���%��m|�5MB6��]��)Ǯ||��tf� J�� �� ��8���{K�3$Nlt�iz��0���#��DrX���GM8�K}WFoq��_�#�Xݻ;�VY��f�b� �NXp��Z}���rgJ����֭;�
�Ӥ�<u����=t%%-�	.\ԓ<֥h�%�N^�7�ˑ����tK��G���İ�ϼ�d��:|���[O���|�|ﹽ���� x�g,��K��j��%�"�).gç������dQGYy�D�\��J,U$�P@     ��s��:I��Vb�/R\p���8E�E;��{x�1�0Μ8�Z�$�J��X\ٙ��Q0�J��cȥ4��<H4��,��~e� ��            (�,2��x��.<'�4��d'��z��Y\���:���cN��(�I��*TǢ����M�֕HǮ�S��^60[���1	3+ �      �,�TC*�eEYVY�|��HG�ez��������%�x�=|����!-�_̼?U��d6FHl��|�^���Ϋ�mS���g�=��M�ə�LjZ����}��X� �_�����a� �R��?��� ��1� ,=���ik^��N�����O���P�:�b#P�kM�s.�kۻ6ݭĩ'�<���c�z��8����4�O�NLNL�aҼ���E�iNu'*�')�\�'����A�����33;���w6rnڼ�g�q���鞹��<?H�}����F|LNZw0�^FZƢҼ�:�u*NS���O,�5��ՍZ3��y<gy��]1�9��������JI�W)��]�w9s�BrJ�}0�����ۮ��}F��J��)�<!Z�����UQA�>�X�����#"w�m�$צm٢d�'']8-�����x�g.N��� ���~���V�W����i�~�/�d]���/4-Բ*��J$�I%RI�  (    Vrq�V�YMx�g��ٚҏ�/N�M&�75��b�-�iˊM��x�g8�%�:)Â>Ҕ���3RZ}������ϑe��[ @         �X�ω0����V8|^%�a�w�M5�R�>%�̥9��L�ǉt�����<�B���>&�)�D��:U�5i`� �     !�K ���QVU�eYa|ʷ�ϙI���D��#��~�=l�W�y
�Th��?�/ս5F}�d���3�����r�QYod����;9IV�)�$�4����s�n�M�p��^)��V=���R\O~�O7���3�T� ��~1�~gN�y;6��4�8����-��wi�h��j��d�ƚN/����FX��_~��`��h���4��]e�:s�*%���:^�[S��Z�J�F���x����������ʔ8���� Y>��EQ�-�K3�~-� ���d��5g�NF�;��� ���N<3�Z��Jx�W���v�XT��ߺ��ɘkz��JV��cJ����I�>֛w+�
72J2��.YO�6ώ��g��qN.k[k�+O�I���=o�?�}���>�IQ�f�U1</�������Y��KVu���ǧVJ�7����N��N�\�r�O|��gl*Q�m��Uk�r�V}����u�_�� ײr�ޗ���k��Lv�w��eNr�ӌ��i�g���%�+g���XqEj��F��K��jOrg�M�[Vt�^%k{��xx��n�V�b���.�|#���v֔b�*έN���W�=N-'���y>q�q��>M�^��nΣ��d����V�Vk���QJΘ�q��m��{)SE�j,F�{c,�ϋ�i����N�xZ���Q�,�q�����S���*��\U�Z�✹�b����E�+n~^^=��WS�#�)�Jd��~�)c�O����œ�L���#�ӗ����z���J�G���̾���Y\�#��к,T����!IT�HD� @     (�-ь�8�Q�Vq��bf��`��}��K���Ӆx"F�+�T��/�MI�d�2-c�-odF����@ e�         �%�ed!(�G;Xx:J�
K=MD�F�ӗ�Xxe�<K%�xf�����a�$Ԗ36�է���<��xQbL�X�@ #@     d�eDe�����,�|��>fs{�Hʧ2_�X�J��/��W?�~��I�/��w�\I/z��ꗇ�ފ���+���gO�'9>�e_����|��2w�w��ըԩ%rno��EfqZ!���+��?/Q�Oܕ��� r<FO��m!gRڤ�cQc�t|���+O��*��X],b0p�͟3�ɦ,s���&L��k��r{�/�U��G�|��GO�Ӹ)�Ѕj�q��In���7���n4�S�\t=	/���2�rNl1x�wN(��'��O?��kWq� ��@Դ��̵>��Q��L.UI%5(�8�+�}kjvvз�ۅ5��_�9��K�#˯��&�-�^k����w�=K�x�Z�7��J��)(�����{b�k5ǎ$�Z-�,����O��� �?�=mz���J�6��ɥ��d�S������=V�� �������#y+����c�>e�*R����ѫ�%K<�g5}���]'���~'�g�.)�~�{�f�l}p񳹝��R�<9B��U���>���c[��*u��{��M�����N$�T|���>�~�ZV|V׎���� 3����^u:|�,�k�lq�����{�R�
���ԑ�AeB�ayoMR�HG�q���3��E���u>��Nk�p��NN�ׂ��o�NS�ǂI���q� ��p�r�>,�,j_$�c;{O�.wi�������>���r��Q��9� J^�'�U�^��^�'�~v�_���i�.v��E���i(�$�J$�I       �e�EaU����JK�Q�xf��g;n'n��YD7��2�,<>�����Mwk���=�5�n���t���ǅ{Y`K���˯�         1��c:�d�_,�±��=Ѳi����&��jj�[嬠��g�	���_�&w��"X�S�pip�i`�K,odF�Dd�Ȓ( 
      �A,��|�d�
���VVY�Ư4n�z����~�&yK������-�ymO*�^H��7�/�� N�#%r2}�?9��U���FC&�o������8��v��Ģ�3��u;g^Pj��!.�ux��$y̌�[q0�w0�W����6���y]׸��R]z%��DM��ŕu^ڣ��Ϫ��k��H���ק�]��n���z(v²�U,#)uq����8���_�t�oF^�`�)/�����x�k;�zm��z�͖R�^��� �?���o�<�FM��L��Xy90�|�����n�>79p���J����Eبw�q�{�ec��y%���16����f��Ҵg(�3��Zi�i�L���]N�b���Up����FK�<��'.�O�Lo�(���GJs�msO=���ښ�+�����
�+��>FI~6;��q��c�����5��6��/�)�,#�N�J�Z�%9��əd��5�<~�g7+.o\�O�d�I��O6��>������}M=�����~��\?׫���_��M������<�[���EQdv��E�TX�BQ$"ĕ!  (      D����
Ȍ���'�2p�^Py�-�C`k���!O�w�/�r1���\��u��t���yKŵ��e�z�1��l 9��       �� �(��2�EIa��./sq;s��	8�a��mNY�<	hZϲ[�Ɍ����TyxOdV1�x,F��;�Ғ�|ˀa� @     �%���	deYvU�Tg=u�~K3�SM>L����ی��)],���4}
��,���':^���wɎ���wyk�:��mǎj+��rd�r��3�^h�k�t�u?g�����ϭ�]�y?K�?,}���#&�V��'�ZxK�0���bѸ|{D�u)`���6 Ѵ�F�����4m �4m �4mbS*24ml���zu��N�ۗ/�>힕B�)c��ۗ��y�r���=��?.y��*�I�]�O4��y�q�l��C��͒K�9>&~]���~���ǃ��['M�۹�:M��e��t�)]wz2�'�,����:9,�!E���"H�(       $��qyꎃ*���CU�-��JK�X��F�x�d��;�ue�
3%��m���s;�&��c°D`��X��۬h �       I�d�R-�k+��6��-3�Y�`����5���Ɩ-�S��\,�4�46�?T\��b���"RQBgk�����y.IX�� �     ��bEYV\�*(ʴ]����3*�T�[3v��Y��$L�ÊQp{��;ZMa����?���xw�h���q�H8N*Q|�["��gg%� O�Ϯ�Oa��NLS�\��1g���Է��փ��4ўOg^��}��58��^L�����M:�r������3���c��ݥ��OӲ��^���!�8�4�4��A�|$�iddh�A6��2���J�j��(Js|�VY�,�9%R�~T��o�8eϏn��a���:�>E��{ʜ)�>���gݳ��[�+��T����>�#
T�:P�8.J+�?�/~���q��J~Y;�VI,c��"1rxH��[���Z�ԛV��gJon^'E:J;�~,�E��)��$�(�*�t�3	E�E��J$�Pi$�I��        +Q��%Ľ�I��Rpi�r!�1H؎��g��i��4�TV%e51����X��Fr��2���6�mq7�"�`��e� @     :�����$����T���:m�bahM��j`iN]���>˘�\R�M�>fX���.�ЈG�8$��j#P �     @@�,AEHe�**ʲ�h"�h�4i�U���Y�)AIa��N�[�tu4CD�Ś��_����lΩҌ��?��)C~k��jM^�䭻K�+=Eb�N�a����{P�n�S���� ��k���^�i�y��]���ɋ��x�?Nś�v����=}捧�Y�1ip���I�Q��*�L�.)5�Ԏ���9x�x�K�\�7.@W#'�O�r}=7A�Ԓ�����V����b�+�7iӦ<w�n�F��[�Yl�v��Z*�侍I�~��t>ť�������V�Z����J����g��}Jg�7���'� v_�(�YS��)(.�{�^l��\�ӡ)o/E&m|����u�1WQ�D��Y�(u����"��t�8�.V�3��K	`�D�N�"E�	HʉH`�#Ab$�D"H� �  (       ���f�M�����C��ݲyY$Γ|�N\>d�v�{"s�˙�#�4���ӎ^_!7ϑ�XXFfV� ]         �(��"6�:jVPR�*�>��RL��MĳpkȎO(؇���'K9K�����Keb>@Vr��QM�ɥ�u�@        �T�U����D`�EZ�EZ4h��mѣEpTҍh�`���P��^�9�NT���w���5�bԉu�Y���ɵ;�F�Hƭ),:sYM�j����sJ2�ēL�1j����NK����f��S���Ш�~O�{��G���k���
�$������ǤM����ewq8pJ��|�v/��z|� ����V���[=N�ҝ�W�+�ԋ��#���J��{.Q\��hS�G���L�re��;{�`ł��i4�FU:ax�zvю����7��?��o�eN�a�e���N	H��癙�H��٣�JE�#ZBE�	"�"�E	�$ @        ~�����ۙXO3B�O,���H��y5qLpEt.�R�E�H�i��r�9(�&�WQ	OUɾlhꆠ.H@ P      ��Ը�bJy6��hͧ�#D�[�8�L,Y��d��T}�s=�o/,Қ�L��%��R�|�.�       � �@EpA`�EHh�EpU��#��f�4�i�x#�H�SJ`���kYF�<8�Y��>�S�s�� 3�']c-��N�-�<�Т��I"�'�"<1kM���pN	�vΑ�pY"R"�	�8%"mL�8'T`���
0N��@ �  (        D�W�H��1)%�4L��S����9K/g����3��~9�oNs;B9y|��\��I-�2�� e�          +('�ضR�S�N�8��lU�?a��5V2�~�S��M�bT���SS��z����49I�cA�'` (        A �#�(�-�0QV��a��S`��E0F��Sp�`�i�08K�`lҘ/���J���8&�pN	�8"���$���8$�	 �H@            g�ԟ��1��ڳYE�F�%�%&=�ˢ(YA�e�4]�3��i7ȼa���K��͓k� G�D��          I��ML��D�h��YMu(�r�4�$��h��&��H��$���|(kU��K,� �            �@A "��(��8(��`J���`
�`�+�pN�`���A8	�8S �   �             �6^l�Ę�đ�g?� �K,�aW4�nUɲ�f�rl�`%�����/�hBXXD��t�� 
          �X`��\�D��v�՘,���ᮅ�jW���V^�!<=���{LR�, fZ��
�X��VA���    㼾�kUB��{��x���{7g?�{��{��?����6�9k�gwڍ>����uUN�I��Y����~�m��u-����W��g���YW֮jN���Mf�V������k��i�M����Ç�~,��u�
޾gO���^��Kx��>���;-2���
�
�c�?y�����J�U%·n�I{�'���ջ��Ԡ���*c=w��}���k�V�+
���<a<-�b�J׏#�c�^ܙ�>��{��++�<2��X��#mUT��;p��q�G���V��:��J�I/J��xg�6�;�{�r>��Mf4�࡭i��-!qZ��xe��x�v�VZ^��%�=_F���~����eީq�o���>Q�g~?2c���C����Xǎ7>��kףmIկR4��NRxK;��R
p��_&�S>�g;�W�R��N�N����<�g����jwU�F��,r�~]>,d�l�o��8�W���3���emyJ��
�YI�^}D��ּ���<&����Τ�J5e9<�����\֘�Ng2�+���i>ogiԆ�BU+ίx��^�g���i��ߎ�t�k�0	���\��Ie���aCU���IS���Em��/nz)\�E���U����gЧ���:|̟R�m��ߢ���=��F�i^�դ�8߭��5$�yO�G�6b�Y��Ȧz���$�^�` ���ݞ�qjR�j9X��g�X_UT��9M�^	/����O�W͛�3��_��=�Ƨ�� w�񣝖y�c���ui�IԜ`��x&)�Y��/'��kk�gJ�G�O	��>���R��4{�9$��R;e~g+��V��������w�A�;5�W����d�%(M�ۣ=%�qˊح�g������������缻��襗�Gθ�e�=�¥gѥ¿��,��NN~�C��Yo���e���FS�ߤ� ׸�ζ��Mǎ��[��K���W�}n������zq��^¾����T�s&��;�%(�E��&�O