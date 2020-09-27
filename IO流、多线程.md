## I/O

![IO](E:\deng\note\IO.png)

### 1.1 å­—èŠ‚æµ

---

#### 1.1 InputStream

##### 1.11 object-Stream

è¾“å…¥è¾“å‡ºé¡ºåºé¡»ä¸€è‡´

```java
public class Person implements Serializable {
    //åºåˆ—åŒ–æ“ä½œ
    long serialVersionUID = 1L;
    private String name;
    private int age;
    transient private int pwd;
    //åŠ transientæ—¶ä¸ä¼šè¢«è®°å½•
```

```java
public class objectè¾“å…¥è¾“å‡º {

    public static void main(String[] args) {

        ObjectOutputStream objectOutputStream = null;
        FileOutputStream fileOutputStream = null;
        ObjectInputStream objectInputStream = null;
        FileInputStream fileInputStream = null;
        try {
            fileOutputStream = new FileOutputStream("abc.txt");
            objectOutputStream = new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeUTF("æ½˜å†¬ç»ƒä¹ dzp");
            objectOutputStream.writeObject(new Person("æ½˜å†¬",5,259));

            fileInputStream = new FileInputStream("abc.txt");
            objectInputStream = new ObjectInputStream(fileInputStream);

            //é¡ºåºå¿…é¡»äºè¾“å…¥çš„ä¸€è‡´
            System.out.println(objectInputStream.readUTF());
            Object object = objectInputStream.readObject();
            //åˆ¤æ–­è¾“å…¥æµä¸­çš„objectæ˜¯å¦æ˜¯Personç±»çš„å®ä¾‹å¯¹è±¡
            if(object instanceof Person){
                Person p = (Person) object;
                System.out.println(p);
            }
```



##### 1.12 Data-Stream

è¾“å…¥è¾“å‡ºé¡ºåºé¡»ä¸€è‡´

#### 1.2 OutputStream

![javaæµåˆ†ç±»](E:\deng\image\javaæµåˆ†ç±».png)

```java
package com.dzp.StreamDemo;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
ã€
public class StreamDemo {
    public static void main(String[] args) {
        //inputæµè¯»å–æ•°æ®
        InputStream inputStream = null;
        try {
            inputStream = new FileInputStream("abc.txt");
            //
            byte[] buffer = new byte[1024];
            int length = 0;
            while((length = inputStream.read(buffer,2,5))!= -1){//ä»ä½ç½®2å¼€å§‹æ”¾ï¼Œæ”¾5ä¸ª
                System.out.println(new String(buffer,0,length));//ä»0å¼€å§‹è¯»å»ï¼Œè¯»lengthä¸ª
            }

```



### 1.2 å­—ç¬¦æµ

---

#### Reader

---

##### BufferedReader

```java
public class æ§åˆ¶å°æµè¾“å…¥è¾“å‡º {
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



### 1.3 æ‰“å°æµ





~~~java
public class æ‰“å°æµæ ¼å¼ {
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

* å¯è¯»å¯å†™

```java
//è¯»å–doc.txtåï¼Œä»¥1024ä¸ºå—è¯»å–å¹¶æ‰“å°
public class RandomAccessFileTest {
    public static void main(String[] args) {
        File file = new File("doc.txt");
        long length = file.length();
        int blockSize = 1024;
        int size =  (int)Math.ceil(length*1.0/blockSize);
        System.out.println();
        int actuallySize;
        int beginPoint;

        //è¿™é‡Œæ˜¯printf
        System.out.printf("å…±è®¡åˆ‡æˆ<%d>å—",size);
        for(int i=0;i<size;i++){
            beginPoint = i*1024;
            if(i != size-1){
                actuallySize = blockSize;
                length -= blockSize;
            }else{
                actuallySize = (int)length;
            }
            System.out.println("è¿™æ˜¯ç¬¬"+i+"è¯»å–"+"---ä»"+beginPoint+"å¤„è¯»å–ï¼Œè¯»å–å¤§å°ä¸º"+actuallySize);
            readSpilt(i,beginPoint,actuallySize);
            System.out.println("-------------------------------------");

        }

    }
    public static void readSpilt(int i,int beginPoint,int actuallySize){
        RandomAccessFile randomAccessFile = null;
        try {
            //åé¢çš„ r ä»£è¡¨è¯»å–

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





## å¤šçº¿ç¨‹

### å‰è¨€

**è¿›ç¨‹**ï¼šæ¯ä¸ªè¿›ç¨‹éƒ½æœ‰ç‹¬ç«‹çš„ä»£ç å’Œæ•°æ®ç©ºé—´ï¼ˆè¿›ç¨‹ä¸Šä¸‹æ–‡ï¼‰ï¼Œè¿›ç¨‹é—´çš„åˆ‡æ¢ä¼šæœ‰è¾ƒå¤§çš„å¼€é”€ï¼Œä¸€ä¸ªè¿›ç¨‹åŒ…å«1--nä¸ªçº¿ç¨‹ã€‚ï¼ˆè¿›ç¨‹æ˜¯èµ„æºåˆ†é…çš„æœ€å°å•ä½ï¼‰

**çº¿ç¨‹**ï¼šåŒä¸€ç±»çº¿ç¨‹å…±äº«ä»£ç å’Œæ•°æ®ç©ºé—´ï¼Œæ¯ä¸ªçº¿ç¨‹æœ‰ç‹¬ç«‹çš„è¿è¡Œæ ˆå’Œç¨‹åºè®¡æ•°å™¨(PC)ï¼Œçº¿ç¨‹åˆ‡æ¢å¼€é”€å°ã€‚ï¼ˆçº¿ç¨‹æ˜¯cpuè°ƒåº¦çš„æœ€å°å•ä½ï¼‰



mainæ–¹æ³•å…¶å®ä¹Ÿæ˜¯ä¸€ä¸ªçº¿ç¨‹

åœ¨javaä¸­ï¼Œæ¯æ¬¡ç¨‹åºè¿è¡Œè‡³å°‘å¯åŠ¨2ä¸ªçº¿ç¨‹ã€‚ä¸€ä¸ªæ˜¯mainçº¿ç¨‹ï¼Œä¸€ä¸ªæ˜¯åƒåœ¾æ”¶é›†çº¿ç¨‹ã€‚å› ä¸ºæ¯å½“ä½¿ç”¨javaå‘½ä»¤æ‰§è¡Œä¸€ä¸ªç±»çš„æ—¶å€™ï¼Œå®é™…ä¸Šéƒ½ä¼šå¯åŠ¨ä¸€ä¸ªJVMï¼Œæ¯ä¸€ä¸ªjvmå®é™…å°±æ˜¯åœ¨æ“ä½œç³»ç»Ÿä¸­å¯åŠ¨äº†ä¸€ä¸ªè¿›ç¨‹ã€‚



interruptç»“æŸçº¿ç¨‹

### **å®ç°æ–¹å¼**

* ç»§æ‰¿*Tread*ç±»
* å®ç°*Runable*æ¥å£
* å®ç°callableæ¥å£ï¼ˆæœ‰è¿”å›å€¼ï¼‰

#### æ‰©å±•java.lang.Threadç±»

ï¼šstart()æ–¹æ³•çš„è°ƒç”¨åå¹¶ä¸æ˜¯ç«‹å³æ‰§è¡Œå¤šçº¿ç¨‹ä»£ç ï¼Œè€Œæ˜¯ä½¿å¾—è¯¥çº¿ç¨‹å˜ä¸ºå¯è¿è¡Œæ€ï¼ˆRunnableï¼‰ï¼Œä»€ä¹ˆæ—¶å€™è¿è¡Œæ˜¯ç”±æ“ä½œç³»ç»Ÿå†³å®šçš„ã€‚

#### å®ç°java.lang.Runnableæ¥å£

Threadç±»å®é™…ä¸Šä¹Ÿæ˜¯å®ç°äº†Runnableæ¥å£çš„ç±»ã€‚

ä¸ç®¡æ˜¯æ‰©å±•Threadç±»è¿˜æ˜¯å®ç°Runnableæ¥å£æ¥å®ç°å¤šçº¿ç¨‹ï¼Œæœ€ç»ˆè¿˜æ˜¯é€šè¿‡Threadçš„å¯¹è±¡çš„APIæ¥æ§åˆ¶çº¿ç¨‹çš„ï¼Œç†Ÿæ‚‰Threadç±»çš„APIæ˜¯è¿›è¡Œå¤šçº¿ç¨‹ç¼–ç¨‹çš„åŸºç¡€ã€‚

#### Threadå’ŒRunnableçš„åŒºåˆ«

runnableä¼˜åŠ¿

1ï¼‰ï¼šé€‚åˆå¤šä¸ªç›¸åŒç¨‹åºä»£ç çš„çº¿ç¨‹å»å¤„ç†åŒä¸€ä¸ªèµ„æº

2ï¼‰ï¼šå¯ä»¥é¿å…javaä¸­çš„å•ç»§æ‰¿çš„é™åˆ¶

3ï¼‰ï¼šå¢åŠ ç¨‹åºçš„å¥å£®æ€§ï¼Œä»£ç å¯ä»¥è¢«å¤šä¸ªçº¿ç¨‹å…±äº«ï¼Œä»£ç å’Œæ•°æ®ç‹¬ç«‹

4ï¼‰ï¼šçº¿ç¨‹æ± åªèƒ½æ”¾å…¥å®ç°Runableæˆ–callableç±»çº¿ç¨‹ï¼Œä¸èƒ½ç›´æ¥æ”¾å…¥ç»§æ‰¿Threadçš„ç±»



### çº¿ç¨‹çš„ç”Ÿå‘½å‘¨æœŸ

![img](https://img-blog.csdn.net/20150309140927553)

#### 1ã€æ–°ç”ŸçŠ¶æ€ï¼š

â€‹          å½“åˆ›å»ºå¥½å½“å‰çº¿ç¨‹å¯¹è±¡ä¹‹åï¼Œæ²¡æœ‰å¯åŠ¨ä¹‹å‰ï¼ˆè°ƒç”¨startæ–¹æ³•ä¹‹å‰ï¼‰
â€‹          ThreadDemo thread = new ThreadDemo()
â€‹          RunnableDemo run = new RunnableDemo()

#### 2ã€å°±ç»ªçŠ¶æ€ï¼š

â€‹		  çº¿ç¨‹å¯¹è±¡åˆ›å»ºåï¼Œå…¶ä»–çº¿ç¨‹è°ƒç”¨äº†è¯¥å¯¹è±¡çš„start()æ–¹æ³•ã€‚è¯¥çŠ¶æ€çš„çº¿ç¨‹ä½äºå¯è¿è¡Œçº¿ç¨‹æ± ä¸­ï¼Œå˜å¾—å¯è¿è¡Œï¼Œç­‰å¾…è·å–CPUçš„ä½¿ç”¨æƒã€‚

#### 3ã€è¿è¡ŒçŠ¶æ€ï¼š

å½“å½“å‰è¿›ç¨‹è·å–åˆ°cpuèµ„æºä¹‹åï¼Œå°±ç»ªé˜Ÿåˆ—ä¸­çš„æ‰€æœ‰çº¿ç¨‹ä¼šå»æŠ¢å cpuçš„èµ„æºï¼Œè°å…ˆæŠ¢å åˆ°è°å…ˆæ‰§è¡Œï¼Œåœ¨æ‰§è¡Œçš„è¿‡ç¨‹ä¸­å°±å«åšè¿è¡ŒçŠ¶æ€
          å°±ç»ªçŠ¶æ€çš„çº¿ç¨‹è·å–äº†CPUï¼Œæ‰§è¡Œç¨‹åºä»£ç ã€‚

#### 4ã€æ­»äº¡çŠ¶æ€ï¼š

å½“è¿è¡Œä¸­çš„çº¿ç¨‹æ­£å¸¸æ‰§è¡Œå®Œæ‰€æœ‰çš„ä»£ç é€»è¾‘æˆ–è€…å› ä¸ºå¼‚å¸¸æƒ…å†µå¯¼è‡´ç¨‹åºç»“æŸå«åšæ­»äº¡çŠ¶æ€
     è¿›å…¥çš„æ–¹å¼ï¼š
       1ã€æ­£å¸¸è¿è¡Œå®Œæˆä¸”ç»“æŸ
        2ã€äººä¸ºä¸­æ–­æ‰§è¡Œï¼Œæ¯”å¦‚ä½¿ç”¨stopæ–¹æ³•
        3ã€ç¨‹åºæŠ›å‡ºæœªæ•è·çš„å¼‚å¸¸

#### 5ã€é˜»å¡çŠ¶æ€ï¼š

åœ¨ç¨‹åºè¿è¡Œè¿‡ç¨‹ä¸­ï¼Œå‘ç”ŸæŸäº›å¼‚å¸¸æƒ…å†µï¼Œå¯¼è‡´å½“å‰çº¿ç¨‹æ— æ³•å†é¡ºåˆ©æ‰§è¡Œä¸‹å»ï¼Œæ­¤æ—¶ä¼šè¿›å…¥é˜»å¡çŠ¶æ€ï¼Œè¿›å…¥é˜»å¡çŠ¶æ€çš„åŸå› æ¶ˆé™¤ä¹‹åï¼Œæ‰€æœ‰çš„é˜»å¡é˜Ÿåˆ—ä¼šå†æ¬¡è¿›å…¥åˆ°å°±ç»ªçŠ¶æ€ä¸­ï¼ŒéšæœºæŠ¢å cpuçš„èµ„æºï¼Œç­‰å¾…æ‰§è¡Œ.
          è¿›å…¥çš„æ–¹å¼ï¼š

ï¼ˆä¸€ï¼‰ã€ç­‰å¾…é˜»å¡ï¼šè¿è¡Œçš„çº¿ç¨‹æ‰§è¡Œ**wait()**æ–¹æ³•ï¼ŒJVMä¼šæŠŠè¯¥çº¿ç¨‹æ”¾å…¥ç­‰å¾…æ± ä¸­ã€‚(waitä¼šé‡Šæ”¾æŒæœ‰çš„é”)

ï¼ˆäºŒï¼‰ã€åŒæ­¥é˜»å¡ï¼šè¿è¡Œçš„çº¿ç¨‹åœ¨è·å–å¯¹è±¡çš„åŒæ­¥é”æ—¶ï¼Œè‹¥è¯¥**åŒæ­¥é”è¢«åˆ«çš„çº¿ç¨‹å ç”¨**ï¼Œåˆ™JVMä¼šæŠŠè¯¥çº¿ç¨‹æ”¾å…¥é”æ± ä¸­ã€‚

ï¼ˆä¸‰ï¼‰ã€å…¶ä»–é˜»å¡ï¼šè¿è¡Œçš„çº¿ç¨‹æ‰§è¡Œ**sleep()æˆ–join()æ–¹æ³•**ï¼Œæˆ–è€…å‘å‡ºäº†I/Oè¯·æ±‚æ—¶ï¼ŒJVMä¼šæŠŠè¯¥çº¿ç¨‹ç½®ä¸ºé˜»å¡çŠ¶æ€ã€‚å½“sleep()çŠ¶æ€è¶…æ—¶ã€join()ç­‰å¾…çº¿ç¨‹ç»ˆæ­¢æˆ–è€…è¶…æ—¶ã€æˆ–è€…I/Oå¤„ç†å®Œæ¯•æ—¶ï¼Œçº¿ç¨‹é‡æ–°è½¬å…¥å°±ç»ªçŠ¶æ€ã€‚ï¼ˆæ³¨æ„,sleepæ˜¯ä¸ä¼šé‡Šæ”¾æŒæœ‰çš„é”ï¼‰

#### æ³¨æ„ï¼š

* åœ¨å¤šçº¿ç¨‹çš„æ—¶å€™ï¼Œå¯ä»¥å®ç°å”¤é†’å’Œç­‰å¾…çš„è¿‡ç¨‹ï¼Œä½†æ˜¯å”¤é†’å’Œç­‰å¾…æ“ä½œçš„å¯¹åº”ä¸æ˜¯threadç±»ï¼Œè€Œæ˜¯æˆ‘ä»¬è®¾ç½®çš„å…±äº«å¯¹è±¡æˆ–è€…å…±äº«å˜é‡ï¼Œéƒ½æ˜¯Objectçš„å­ç±»
* å¤šçº¿ç¨‹å¹¶å‘è®¿é—®çš„æ—¶å€™å›å‡ºç°æ•°æ®å®‰å…¨é—®é¢˜ï¼š
  * è§£å†³æ–¹å¼ï¼š
                  1ã€åŒæ­¥ä»£ç å—
                      synchronized(å…±äº«èµ„æºã€å…±äº«å¯¹è±¡ï¼Œéœ€è¦æ˜¯objectçš„å­ç±»){å…·ä½“æ‰§è¡Œçš„ä»£ç å—}
                  2ã€åŒæ­¥æ–¹æ³•
                      å°†æ ¸å¿ƒçš„ä»£ç é€»è¾‘å®šä¹‰æˆä¸€ä¸ªæ–¹æ³•ï¼Œä½¿ç”¨synchronizedå…³é”®å­—è¿›è¡Œä¿®é¥°ï¼Œæ­¤æ—¶ä¸éœ€è¦æŒ‡å®šå…±äº«å¯¹è±¡



### çº¿ç¨‹è°ƒåº¦

**1ã€è°ƒæ•´çº¿ç¨‹ä¼˜å…ˆçº§**

Javaçº¿ç¨‹çš„ä¼˜å…ˆçº§ç”¨æ•´æ•°è¡¨ç¤ºï¼Œå–å€¼èŒƒå›´æ˜¯1~10ï¼ŒThreadç±»æœ‰ä»¥ä¸‹ä¸‰ä¸ªé™æ€å¸¸é‡ï¼š

```cpp
static int MAX_PRIORITY
          çº¿ç¨‹å¯ä»¥å…·æœ‰çš„æœ€é«˜ä¼˜å…ˆçº§ï¼Œå–å€¼ä¸º10ã€‚
static int MIN_PRIORITY
          çº¿ç¨‹å¯ä»¥å…·æœ‰çš„æœ€ä½ä¼˜å…ˆçº§ï¼Œå–å€¼ä¸º1ã€‚
static int NORM_PRIORITY
          åˆ†é…ç»™çº¿ç¨‹çš„é»˜è®¤ä¼˜å…ˆçº§ï¼Œå–å€¼ä¸º5ã€‚
```

setPriority()å’ŒgetPriority()æ–¹æ³•åˆ†åˆ«ç”¨æ¥è®¾ç½®å’Œè·å–çº¿ç¨‹çš„ä¼˜å…ˆçº§ã€‚

**2ã€çº¿ç¨‹ç¡çœ **

Thread.sleep(long millis)æ–¹æ³•ï¼Œä½¿çº¿ç¨‹è½¬åˆ°é˜»å¡çŠ¶æ€ã€‚milliså‚æ•°è®¾å®šç¡çœ çš„æ—¶é—´ï¼Œä»¥æ¯«ç§’ä¸ºå•ä½ã€‚å½“ç¡çœ ç»“æŸåï¼Œå°±è½¬ä¸ºå°±ç»ªï¼ˆRunnableï¼‰çŠ¶æ€ã€‚

**3ã€çº¿ç¨‹ç­‰å¾…ï¼š**

Objectç±»ä¸­çš„wait()æ–¹æ³•ï¼Œå¯¼è‡´å½“å‰çš„çº¿ç¨‹ç­‰å¾…ï¼Œç›´åˆ°å…¶ä»–çº¿ç¨‹è°ƒç”¨æ­¤å¯¹è±¡çš„ notify() æ–¹æ³•æˆ– notifyAll() å”¤é†’æ–¹æ³•ã€‚

**4ã€çº¿ç¨‹è®©æ­¥**ï¼šThread.yield() æ–¹æ³•ï¼Œæš‚åœå½“å‰æ­£åœ¨æ‰§è¡Œçš„çº¿ç¨‹å¯¹è±¡ï¼ŒæŠŠæ‰§è¡Œæœºä¼šè®©ç»™ç›¸åŒæˆ–è€…æ›´é«˜ä¼˜å…ˆçº§çš„çº¿ç¨‹ã€‚

**5ã€çº¿ç¨‹åŠ å…¥**ï¼šjoin()æ–¹æ³•ï¼Œç­‰å¾…å…¶ä»–çº¿ç¨‹ç»ˆæ­¢ã€‚åœ¨å½“å‰çº¿ç¨‹ä¸­è°ƒç”¨å¦ä¸€ä¸ªçº¿ç¨‹çš„join()æ–¹æ³•ï¼Œåˆ™å½“å‰çº¿ç¨‹è½¬å…¥é˜»å¡çŠ¶æ€ï¼Œç›´åˆ°å¦ä¸€ä¸ªè¿›ç¨‹è¿è¡Œç»“æŸï¼Œå½“å‰çº¿ç¨‹å†ç”±é˜»å¡è½¬ä¸ºå°±ç»ªçŠ¶æ€ã€‚

 **6ã€çº¿ç¨‹å”¤é†’**ï¼š















Thread implements Runnable

```java
//åŒæ­¥ä»£ç å—
public class Runnerbleå–ç¥¨ implements Runnable {

    for(int i =0;i<10;i++){
            try {
                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (this){
                if(tickets>0) {
                    System.out.println(Thread.currentThread().getName() + "æ­£åœ¨å–ç¬¬" + (tickets--) + "å¼ ç¥¨");
                }
            }
        }
    public static void main(String[] args) {
        Runnerbleå–ç¥¨ runnerbleå–ç¥¨ = new Runnerbleå–ç¥¨();
        //æ‰€æœ‰çš„çº¿ç¨‹å…±ç”¨Runnerbleä¸€ä¸ªæ¥å£ï¼Œå› æ­¤å…¶ä¸­çš„å˜é‡ä¼šå…±äº«
        //ç›¸å½“äº4ä¸ªä»£ç†äººå¹²åŒä¸€ä»¶äº‹
        Thread thread1 = new Thread(runnerbleå–ç¥¨);
        Thread thread2 = new Thread(runnerbleå–ç¥¨);
        Thread thread3 = new Thread(runnerbleå–ç¥¨);
        Thread thread4 = new Thread(runnerbleå–ç¥¨);
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();

    }
}
```

```java
//åŒæ­¥æ–¹æ³•
public class Runnerbleå–ç¥¨ implements Runnable {

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
            System.out.println(Thread.currentThread().getName() + "æ­£åœ¨å–ç¬¬" + (tickets--) + "å¼ ç¥¨");
        }
    }
    public static void main(String[] args) {
        Runnerbleå–ç¥¨ runnerbleå–ç¥¨ = new Runnerbleå–ç¥¨();
        Thread thread1 = new Thread(runnerbleå–ç¥¨,"A");
        Thread thread2 = new Thread(runnerbleå–ç¥¨,"B");
        Thread thread3 = new Thread(runnerbleå–ç¥¨,"C");
        Thread thread4 = new Thread(runnerbleå–ç¥¨,"D");
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();

    }
}
```

### çº¿ç¨‹apiæ–¹æ³•

![çº¿ç¨‹æ“ä½œ](E:\deng\image\çº¿ç¨‹æ“ä½œ.png)

**join()æ–¹æ³•**

**ç­‰å¾…è¯¥çº¿ç¨‹ç»ˆæ­¢**

åœ¨å¾ˆå¤šæƒ…å†µä¸‹ï¼Œä¸»çº¿ç¨‹ç”Ÿæˆå¹¶èµ·åŠ¨äº†å­çº¿ç¨‹ï¼Œå¦‚æœå­çº¿ç¨‹é‡Œè¦è¿›è¡Œå¤§é‡çš„è€—æ—¶çš„è¿ç®—ï¼Œä¸»çº¿ç¨‹å¾€å¾€å°†äºå­çº¿ç¨‹ä¹‹å‰ç»“æŸï¼Œä½†æ˜¯å¦‚æœä¸»çº¿ç¨‹å¤„ç†å®Œå…¶ä»–çš„äº‹åŠ¡åï¼Œéœ€è¦ç”¨åˆ°å­çº¿ç¨‹çš„å¤„ç†ç»“æœï¼Œä¹Ÿå°±æ˜¯ä¸»çº¿ç¨‹éœ€è¦ç­‰å¾…å­çº¿ç¨‹æ‰§è¡Œå®Œæˆä¹‹åå†ç»“æŸï¼Œè¿™ä¸ªæ—¶å€™å°±è¦ç”¨åˆ°join()æ–¹æ³•äº†ã€‚

**yield():**

**æš‚åœå½“å‰æ­£åœ¨æ‰§è¡Œçš„çº¿ç¨‹å¯¹è±¡ï¼Œå¹¶æ‰§è¡Œå…¶ä»–çº¿ç¨‹ã€‚**

å…ˆæ£€æµ‹å½“å‰æ˜¯å¦æœ‰ç›¸åŒä¼˜å…ˆçº§çš„çº¿ç¨‹å¤„äºåŒå¯è¿è¡ŒçŠ¶æ€ï¼Œå¦‚æœ‰ï¼Œåˆ™æŠŠ CPU çš„å æœ‰æƒäº¤ç»™æ­¤çº¿ç¨‹ï¼Œå¦åˆ™ï¼Œç»§ç»­è¿è¡ŒåŸæ¥çš„çº¿ç¨‹ã€‚

ä½†æœ‰å¯èƒ½ç»§ç»­æŠ¢å åˆ°cpu

**wait()**

Obj.wait()ï¼Œä¸Obj.notify()å¿…é¡»è¦ä¸synchronized(Obj)ä¸€èµ·ä½¿ç”¨,æ˜¯é’ˆå¯¹å·²ç»è·å–äº†Objé”è¿›è¡Œæ“ä½œ

Thread.sleep()ä¸Object.wait()äºŒè€…éƒ½å¯ä»¥æš‚åœå½“å‰çº¿ç¨‹ï¼Œé‡Šæ”¾CPUæ§åˆ¶æƒï¼Œä¸»è¦çš„åŒºåˆ«åœ¨äºObject.wait()åœ¨é‡Šæ”¾CPUåŒæ—¶ï¼Œé‡Šæ”¾äº†å¯¹è±¡é”çš„æ§åˆ¶ã€‚

**sleepï¼ˆï¼‰**

 sleep()æ˜¯Threadç±»çš„Static(é™æ€)çš„æ–¹æ³•ï¼›å› æ­¤ä»–ä¸èƒ½æ”¹å˜å¯¹è±¡çš„æœºé”ï¼Œæ‰€ä»¥å½“åœ¨ä¸€ä¸ªSynchronizedå—ä¸­è°ƒç”¨Sleep()æ–¹æ³•æ˜¯ï¼Œçº¿ç¨‹è™½ç„¶ä¼‘çœ äº†ï¼Œä½†æ˜¯å¯¹è±¡çš„æœºé”å¹¶æœ¨æœ‰è¢«é‡Šæ”¾ï¼Œå…¶ä»–çº¿ç¨‹æ— æ³•è®¿é—®è¿™ä¸ªå¯¹è±¡



### é”æœºåˆ¶

#### synchronizeé”



é”çš„æ˜¯å¯¹è±¡ä¸æ˜¯ä»£ç 

this XX.class

å…¶ä»–çº¿ç¨‹çœ‹è¢«é”å¯¹è±¡çš„å‰ä¸¤ä½ï¼ˆ64ä½ï¼‰

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

##### ä¼˜åŒ–

é‡‡ç”¨ç»†ç²’åº¦çš„é”ï¼Œå°½é‡ä¸è¦å°†æ— éœ€é”çš„ä»£ç æ”¾å…¥å…¶ä¸­

å½“é”æ¯”è¾ƒå¯†é›†çš„æ—¶å€™ï¼Œå¯ä»¥ç²—åŒ–é”



é”å®šæŸå¯¹è±¡oï¼Œå¦‚æœoçš„å±æ€§å‘ç”Ÿæ”¹å˜ï¼Œä¸å½±å“é”çš„ä½¿ç”¨ã€‚

ä½†æ˜¯å¦‚æœoå˜ä¸ºå¦ä¸€ä¸ªå¯¹è±¡ï¼Œé”å°†å¤±æ•ˆï¼ˆå¦ä¸€ä¸ªå¯¹è±¡çš„å‰ä¸¤ä½ä¸ç›¸åŒï¼‰

å› æ­¤åº”è¯¥	final Object o = new Object();

##### é”çš„å‡çº§

â€‹	é¦–å…ˆæ˜¯åå‘é”ï¼Œå¦‚æœçº¿ç¨‹äº‰ç”¨ï¼Œå‡çº§ä¸ºè‡ªæ—‹é”ï¼ˆå ç”¨cpuï¼‰

â€‹	å¦ä¸€ä¸ªçº¿ç¨‹å¾ªç¯åˆ¤æ–­æ˜¯å¦èƒ½æ‹¿åˆ°é”

â€‹	åå‘é”ä¸æ˜¯åŠ é”ï¼Œè€Œæ˜¯è®°ä½ç¬¬ä¸€ä¸ªæ‹¿åˆ°çš„çº¿ç¨‹ï¼Œå€¾å‘äºä»–å…ˆæ‰§è¡Œ

â€‹	10æ¬¡åå‡çº§ä¸ºé‡é‡çº§é”

æ‰§è¡Œæ—¶é—´é•¿ï¼Œçº¿ç¨‹æ•°é‡å¤šç”¨é‡é‡çº§ï¼ˆOSï¼‰é”

æ—¶é—´çŸ­ï¼Œçº¿ç¨‹æ•°é‡å°‘ç”¨è‡ªæ—‹é”



#### volatile

* ä¿è¯çº¿ç¨‹å¯è§æ€§
  * ä¸åŒçº¿ç¨‹å–åˆ°å…¬å…±å †ä¸­çš„å€¼ï¼Œå…ˆåœ¨è‡ªå·±çš„ç©ºé—´æ”¹å˜ï¼ŒåŒæ–¹æ•°æ®ä¸èƒ½åŠæ—¶å…±äº«ï¼ŒåŠä¸å¯è§
  * MESI ç¼“å­˜ä¸€è‡´æ€§åè®®
* ç¦æ­¢æŒ‡ä»¤é‡æ’åº
  * ç”Ò¯OË’âOäH½&u¶Å’±¹¬éŸÉ\îFMéÍ|äÒÂ£·ÔèJŸ£ŞK‚KÅ3ìZİÿ XZıêùœòGá.üy˜Ë]=nLçëÉœß¥î>5<¿O“Ò”J*™(êó,Y%–dH (    eZ?Y¤˜Ü9ãVQ{î¡55·?’£Ö/ÜgÃ8<ã—S}¥ÏsJ¿¬e¨Ã/‰òD$êÍô7ŠQI	FˆÎÒöÜç©QÉál¾eëO…uæb+å­ìé¥ú´X­/Õ¢Æ'Ë¤x @            
Ê¤c$ŸP‹õcÃ7í:ÊÊ)R<QÛš-gR–ÃŸ‘Ó	qA3˜ÚƒÙ£vğçIîÔÚŠËx2föÆ"6é3ØÑâi¶Û4$öX€      0©,ƒBVK*Ã(e¡É”dÃ“%ı-ãõ-7èKÉ>‹ş¾oĞ—“<}/Õ£ÓÂ÷x>©â­22W(dúZ|D¶}^Î¿ë?îßÍ!³êvqÿ Zvÿ #ô¥êâ~½^¢ââ•µV­.qæñœ¶ú½…İUJ…Ìe7Ê8i¿‰½åº»´©AÏƒc‹Áó,û=JÒæå]Ô”b¸q¹ò+æ³6ïÑd¶h¼Ec·»·R´å¤àâœÒnÁ[Oµúmä(¼¨½ä×DzmVş…¤§).òiªqñgÃìÛÎ¡Sü'óG«­\6—ƒ—J_‘JşïGNŞTiÒ‚òIÖúµÕnæ…Â•O5Ÿ,ó)­·"¾:¤¿š<Å”±}nÖØ«™ÏÉI¼Ë¿#•8rWGg©Ôì£}m/EwĞY„º¿aæìmİİä(å¤÷“]6zìàóšTuZÑ\£	%ş$oKF;GÃ/mš“óå÷â¡BŠ„§J—$—´å¡ªÙ\ÖîhÜ)TYÛgËÄ^\:]|uI?{Hó6ıòƒğ©™ŒX>å&Ó.¼TàÉ\uÏQ¨ZÆúŞQqN¬ViË®|…¥?ë*^ÿ “=&pö<İ‚PÖÕ5Ê3š_oiœw¬ü9òñÄfÇx÷—ß®ÿ  ©ü/ä|½"Ã<7u—¢¿WÕøùY¨ÉbK1{5ìôI$¶IrHóS,Ö“X÷{r`®L‘{{9ïo#iEÕe'´cÖLóÓ«:ÕeV¬³9=ıÅì/¨Õ¯;êŠápÊ…C¤WùøœÉŸO†)^¯y|>o&rß¦<CLì2S$å4zŸ=l³KgFÛïÌÉ3K_Şßx¾g<¾‰vÁúµzÂ“õ‰)'¹ñ)ê~§'¥dX¢,Ï:è‰"¬I’*@P      ¨ŸÆr¼£ZQç¹b6ÌÛMÃxMø…E5·?*¼Sd×ußm°“â“a''„Aµaq>¼³:‡ËUÈ®¤T¸wÉÉÙ` P            Æ¿¬¼ŒëGn,¾Y·…)Ôqx|ÓÊÊ9M)O‹êjÑîÍgÙ5i}hü
Ò—{”•$ŞSÆIÛR³^û†3››ü	§MÉåò6Tâºo„ŠûÈ’K e°    ‚YÈ%hU•eŠ0Ê¬˜rdK‹Ø—ô·Ô™ú’ò<}7ˆ¾oĞ—“<t¢z¸>eàú¯Š´É#%r}M>ÖÉõ{4óª¿»4||û­Ù—ılşéüÑçä~”½\O×¯ú½6¡tììªÜFŸxé¬ğñc>óãZv§étèU²tÕI(©*œXoe¶»Z;ŠR£Z<PšÃYÆNJ:>mUU¥l”ãÉ¹7‹>E'Vz£»ôYkoIÔ{¶»¶¥yo*c•%³ëâ7Ù§J¤Zßº4}ıFú…¤ëÔk8Ä"ş´º#Çi÷ÎÂúÃËŠxš]bùş~ãÑÇ¥­ŠĞñòïJg¤ÏõzÍn.zEÂŠËI?ri³ÊØfZ…ºKwV?3ÚS©JæŠœjÒ¨¼Ô‘ÏGM°µ«ßP·Q©ºMÉ¼ydÆ,ñ“I‡NGse®JÏgKg›Ğj)êÕ¦¹Jkâ§«ê0±µ”T—Qbœzùù#ÌXŞ;+ÊuğÜbñ$º§ÌéÇÅkc´ü¹òóV¹©ğõ:´M2¼RËáâø4ÿ ÌYzWÖñ]jÇæzúUiW¥”ÜjSšÛªhÊ–ŸamUÕ·¶P©†²äŞ<²sÅ1Òk0éÈâÎl•ÉYìØó:|ÔõÕ%ÉÎm|ö5MB6ÏÒ]ìÖ)Ç®||‘ğtfÿ JĞÿ öÿ µ8ôŸ·{K—3$Nltiz™Î0ƒ”#–ßDrXê¯ÕGM8¸K}WFoq½½_à#ÊXİ»;¸VYáõf¼bÿ ÖNXpıÊZ}áèärgJÄø—İÖ­;ú
êÓ¤±<u¸ø=t%%-§	.\Ô“<Ö¥hì¯%—N^•7â¿Ë‘êáåÜtKçıG§î×Ä°ÈÏ¼¢däú:|¯“[OÛí¾ñ|Î|ï¹½Ÿíöÿ x¾g,±øK·õjõ…%ë"Ù).gÃ§©ú¬¾•‘dQGYy—D•\Ë¨J,U$ªP@     ”s°:I¹Vbü/R\p‹ø›8EóE;•{xÜ1Ó0Îœ8ZÙ$–J³áX\Ù™ËQ0¥J™ôcÈ¥4ÜÖ<H4¢³,øñ~e° æì            (©,2½ìx°ö.<'—4—šd'‡“zâY\ÌÃÃ:ÄíÎcN˜¾(¦Iè´*TÇ¢¹œõßMï¶Ö•HÇ®äS›^60[³¢á1	3+ ¶      ‚,†TC*ËeEYVY•|ŠŠHG¨ez™¿†±ú–›ô%äxå²=|Ÿ¢ü!-_Ì¼?UôÔd6FHlúº|¶^ŞîâÎ««mS»©Œg…=½æM•É™¬LjZ­¦³¸}öƒXÿ _ô£ùõıaÿ çRşê?‘óÛ åö1ÿ ,=Äæşik^â½ÕNòâ´êÏÆO—’èP¨:Åb#PãkM§s.‹kÛ»6İ­Ä©'Î<âıÏcªzş©8ãé¶4ÖOšNLNLîaÒ¼ŒµE¥iNu'*•')Î\å'–È÷A¸ˆĞå33;–ö÷w6rnÚ¼©gœq”ıÌé¹©Ê<?H„}ªšÉÀF|LNZw0ë^FZÆ¢Ò¼ç:“u*NSœ¹ÊO,š5êÛÕZ3àœy<gyöƒ]1­9õÎú·İÚõJIÆW)§³]Üw9s¹BrJã­}0ÕòŞş©Û®–¥}Fš¥Jã†Ú)Á<!Zúêê…ÅUQAæ>‚Xø©’¥±#"wİmŸ$×¦mÙ¢dä¢'']8-“¢Çöúx¾g.N›ï¨ÿ ùœ³~œ»ñ¿V¯W’²æğiê~¯/¥d]ÄĞë/4-Ô²*‹¨J$„I%RI’  (    VrqVåƒYMx„g©óÙšÒš/N£M&ö75÷†bß-iËŠM›Ôxƒg8¬%¤:)Ã‚>Ò”©¼ñ3RZ}–±î²©õÏ‘eÈË[ @         óX“Ï‰0¨â÷İV8|^%aÆwéM5”R¥>%•Ì¥9áğ¾LØÇ‰tğÂƒ‹<ğB„¥¾>&ü)ôD—©:U5i`´ €     !K ¨©’QVU–eYa|Ê·‚Ï™IşÇêD¥è¿#ÈÔ~›=l½Wäy
ÛThõı?Õ/Õ½5F}¤dŒ‘œ3ëéùô¶r’QYod–í•Éõ;9IVÖ)¹$Õ4çºëÓæsÉnŠM¾pÓî^)òúV=•‡©R\O~îŸO7ù•»3¦T ªÒ~1–~gN­y;6µÅ4¥8¤£ÅË-àóšwi®h×¤jºÔd¹ÆšN/İĞùFX›Ö_~õâ`˜ÅhòäÕ4ªÚ]eµ:sõ*%³ü™:^“[S¨øZ§J½F²—±x³èêúö¨éó¡ó¼Ê”8¡„Ÿÿ Y>®‰EQÒ-ÒK3~-ÿ –÷äd¦Ú5g—NF«;¯Ÿÿ Òìæ—N<3Z¯íJxùWı˜‡vêXT“’ßº©×É˜kzÕõJVöµcJ±õœ›Iõ>Ö›w+Û
72J2’İ.YOÏ6Ï±’g´½qN.k[kŞ+O†I¦ğ=oû?¥}Šßã>¿IQÖfâ’U1</Ïùäõ­¹Y­ÓKVu·Ÿ…Ç§VJŞ7§‹±¡N¾§N…\ºr›O|‚¿gl*Q”mûÈUkĞr–V}§ÂÒŞuª_Æÿ ×²ròŞ—™ökƒLvêw„œeNr„ÓŒ¢Úiôg¬ı¥%¼+gøÒXqEj–ûF²ùKğøjOrgäM©[Vt¼^%k{Öñ½xx¹ĞnòVôbäûÇ.¯|#ïÛövÖ”bï*Î­N°¥´W³=N-'‡ôıy>qãqóÎ>MŸ^ûénÎ£²d²¶ËÇV—Vk‘šıQJÎ˜âq±Ím’ñ½{)SEÒj,Fé{c,üÏ‹ªiŸ£§ñ«N¦xZÙûÑQÖ,æ§q¸ıš´¸S÷¤*µê\U•ZÓâœ¹¿bö¸øóE¿+n~^^=©øWSû#©)ûJd”Ï~Ÿ)c¯Oı²—ñ¯™Å“³LŞòŸñ#–Ó—£‹úÕzœ’J“Gç©åúÌ¾–‘äY\‹#¼¼Ğº,T±–’‰!IT¢HD @     (©-ÑŒ©8ûQ¾VqÁbf˜‰`äÜ}¥¡K¬—¸Ó…x"FÒ+òT©Ò/ŞMIãd÷2-cİ-odF‘¸±@ eĞ         %œed!(ñG;Xx:JÎ
K=MDé›FØÓ—ƒXxeé<K%·xf½¥±œêaâ$Ô–36ÉÕ§Ú¥—<ø„xQbL÷X@ #@     dÈeDeŠ²¢¬«,ù|ŠŠ>fs{šHÊ§2_ÃXıJÉú/ÈòW?®~ãÖIú/Èòw›\I/zşê—‡êŞŠ±ÈË+‘œŸgOÏ'9>Ïe_õ³û©|Ññ2wèw‘³Õ¨Ô©%rno¦ÏEfqZ!éâÚ+š³?/QÚOÜ•¼ãÿ r<FOĞïm!gRÚ¤œcQc‰t|ÓøŸ+OìÅ*î­ÍX],b0pÛÍŸ3‹É¦,sòûŞ&Lù¢káär{İ/÷U¯İGä|¾ÒGO´Ó¸)ÙĞ…jÏq¦“Inßáï7ìíän4¨Sâ\t=	/—òù2òrNl1xwN(ãò'ÎçO?¯¼kWqÿ µ‹@Ô´ñÜÌµ>ÎÃQºúL.UI%5(ç8Û+Ü}kjvvĞ·¤Û…5„ß_ñ9çÍKá­#Ë¯“&ù-â^k´úÚw›=KæxZî7šµJ´å˜)(Åø¥ş²{bòk5Ç$áZ-—,ÇËÆéOúò’ÿ ò?“=mzª…½JÍ6©ÁÉ¥ÍádòSş½¥÷ñ=V ÿ «®¾æö²ó#y+äÏÓæcæ>e¬*R¸£ªÑ«ü%K<ïg5}¬¶–]'üÚü~'¡g“.)Å~™{ğf®l}pñ³¹¦­Rµ<9B¬U—•ğ>ı®¿c[¿î*uŒı{ùßM«©ê•áÁN$êT|¢³ó>­~ËZV|V×’Æğ«ÿ 3èçûÔ^u:|,òkÕlq¸ÛêÑÔ{è¹R¯
ÑäğÔ‘ğ»AeBœayoMRâ—HG–q”Ò÷3¶ÃE§¥Êu>’«Nk„p’üNNĞ×‚·§oŸNSãÇ‚I¯Äóqÿ ñçpör¿>,Û,j_$ä¢c;{OÌ.wiµÃø—Ìùù>†’ór³ÒQùœ9ÿ J^'ëUé²^Œò^Ÿ'æ~v_¬Ëéièª.v—šEŠ¢Äi(’$•J$„I       eÊEaU­¥¹«JKÂQáxf£¿g;n'n„ÓYD7„Û2§,<>¥ª½±âMwk«¶Ù=Ø5§nÖæætçµ©Ç…{Y`K›ÁÏË¯€         1©ë³c:«dË_,ÛÂ±¨×=Ñ²i¬£˜´&ãäjjÌ[å¬ ¥æg‡	¯™¬_É&w¦¦"XâS–pip¯i`ÚK,odF€Dd¥È’( 
      †A,€ˆ|Èd²
²Œ¹VVYÈÆ¯4nÎzÏ¿¥¬~¥&yKı®¤½ˆõ-ìymO*ò^Höı7×/Õÿ NÙ#%r2}½?9µ²U½·êFC&—o­§öšúÂš£8Ææ”vŠ›Ä¢¼3ùu;g^Pj•„!.ux—Á$yÌŒ[q0Úw0öWµé‹6¹»¯y]×¸¨çR]z%à—DM­åÅ•u^Ú£„ÖÏª’ğk©†HÉÛí×§§]¹n®½÷z(vÂ²‚U,#)uq«…ğÃ8µĞŞ_Ót£oF^´`ó)/ü•‘“x˜k;ˆzmÎÏzôÍ–RÃ^Ãïÿ ¶?³×ıoò<öFMäÁLºêXy90ï¢|º­¯­ìn•>79pç÷ŸJãµ¸·©EØ¨wqÏ{œecÀøy%øøï16•—fµÒ´g(Ê3‹á”Zi®i®Lû‘í]N¥bœ±»Up›òÁğFK“<šêƒ'.ôO—Lo®(İÎæ…GJs“msO=‰ôáÚšÊ+½³„åã
œ+àÓ>FI~6;ú¡qòócôËíÕí5ÄáŠ6°§/µ)ñ,#åN­JÕZµ%9ÉåÉ™d”ö5<~˜g7+.o\´OÀd¦IÉßO6×â>–½Ãó‰ò²}M=ô¹ó‰çå~”½\?×«ÒÒõ_™†M¨ú¯Ìüİ<¿[—ÒÙEQdv—šE‘TXBQ$"Ä•!  (      D¢¤·—
ÈŒ”–Ì'ù2p”^Py©-—C`k©•!O‡w»/Èr1œø¶\‰Öu´ªtÄÏyKwÌ‚ô–eŸzÔ1¹™l 9º€       ¬¬ Ï(¸¼2‰EIa˜Ê./sq;s˜Ñ	8¿a¹ÎmNY<	hZÏ²[ÂÉŒ¤äòËTyxOdV1âx,F»“;ìÒ’Ä|Ë€a¸ @     È%€Š²	deYvU†Tg=u²~K3”SM>L³™ÛŒá½Ó)],ò’ä×4}
”Ü,ÇÄÏ':^ø­¸wÉ™««wyk½:½«mÇj+çàrdörŠ—3æ^h´kætŸu?gªıÇ×ÁÏ­»]ğy?Kµ?,}áç›ğ#&×V•í'ÃZxK£0ÉôâbÑ¸|{DÖu)`Œ‚é6 Ñ´‚F¤¤4m €4m €4mbS*24ml“œzuÍãNá‡Û—/ó>í•BÕ)c§Û—áày³r±âó=ŞÎ?.yíŸ*ÓI¯]©O4àüy¿q÷lì©ÚCæÍ’K‘9>&~]òöö~‡ÀÇƒ¿™['MâÛ¹•:M¼ËeáâtÅ)]wz2Ş'´,‹¢¨²:9,‰!E„¢Ä"H©(       $³Œqyêƒ*ÃÏCU–-ëÂJKÚXÂÆFÒx‹d˜î±;†ue¿
3%îòmáÎs;‘&Ş¼cÂ°D`£æXÄÎÛ¬h °       Iád…R-–k+Á¬6‹¶-3ÃYæ`¤ãÈÒ5ÙìÄÆ–-‰SêŠÂ\,Ø4Ÿ46?T\™¬b¢°‰"RQBgk¤‚““y.IX€       ©bEYV\«*(Ê´]¢¬°Ë3*”T·[3vŠ´YˆŸ$L×ÃŠQp{¢¹;ZMa¬œó¡Ö?•±Ìxw®hÒÆqH8N*Q|Ó["ó³ñgg%ÿ OäÏ®ÓOaƒ¦NLSÚ\óñ1gğñµ¨Ô·¨éÖƒ„—4ÑOg^ª}İÍ58øõ^Lø—½«M:¶rïéıŸ¬¿3íàçcÉÚİ¥ùŞOÓ²âï^ğøù!§8É4×4úŸAó|$“iddhÚA6œ‚2¤¥J¥jŠ(Js|”VY÷,»9%Rş~T şoò8eÏnÒôaãäÍ:¤>Eµ¥{Êœ)¹>¯¢ógİ³Ğè[µ+†«Tğú«ó>Œ#
TÕ:P8.J+Ÿ?Ô/~ÔíĞq¾—J~Y;ÊVI,c–Ã"1rxHŞŞ[³çÄZó·Ô›V‘¨gJon^'E:J;ó~,²E‘Ö)óÛ$Ù(º*‘t3	EˆE‘•J$‚Pi$I€         +Q´–%Ä½£I¾úRpiår!Ë1HØœà»g¥œi·ì4ŒTV%e51¹•ˆˆX«šFr“—2¥Š³6ømq7·"Å`±Æe¸ @     :‘úÈĞ$ÆÜàÒTú¢˜:mÊbahMçìj`iN]™†ë>Ë˜Î\RöMâ>fX‚Óì½.¬ĞˆG†8$Ìùj#P €     @@«,AEHeˆ**Ê²íh"hÓ4i´U¢íÑYÓ)AIa¬˜N‹[Çtu4CDšÅš­í_ˆÉÅålÎ©ÒŒù¬?¥)C~kÄãjM^šä­»K»+=EbæŸN•a³÷ø{PĞnìS«ßĞÿ ‰k¡èò^iÒyƒó]êÁÍÉ‹·˜x¹?NÅ›¼v—†ÈÊ=}æ§êY”1ipú¥èIûQæõ*÷L.)5êÔñ—¼û¸9xóxKóœ\Ş7.@W#'¯OÖr}=7A½Ô’¨£ÜĞëV¦Ëİâbù+7iÓ¦<wÉnšFåó–[ÂYlûv›¯Z*µä¾Iô~»÷t>Å¥†–¿İáßVëZ‡¬êJ¤¸¤ÛgÆä}Jg¶7Şâı'ÿ v_Ù(ÛYSîí)(.²{Ê^lœç™\šÓ¡)o/E&m|“¹îûu­1WQÙD²ğ–Y´(uŸÀÖŒ"‹¤t®8.VÍ3áŠK	`²D¤N"E’	HÊ‰H`²#Ab$ŠD"H© ’  (       •£äfMÚÊÁƒC£İ²yY$Î“|‹N\>d×v¢{"sÆË™‘#â4ÄÌÊÓ^_!7Ï‘ªXXFfVµ ]         ³(¶²"6“:jVPRó*ª>»—RLº˜MÄ³pkÈO(Ø‡ù¢õ'K9K‹¡¬•Keb>@VrÂÇQM·É¥ßu€@        †T±U¢À¨£D`»EZ£EZ4h†mÑ£EpTÒhÓ`¬¹êPŒ÷^‹9§NTŞëŞwà‡¬5”bÔ‰u®Y¯—ÎÉµ;©F•HÆ­),:sYM©jğÛØsJ2ƒÄ“Lã1jËÑ­ãNKŞÌÚßf®—S¹«ÍĞ¨ö~Oı{G³º½kŸ£ı
¤$¦±ïäıÇ¤M§”ğÍewq8pJ´Ü|Ïv/¨æ¥z|ÿ «æåúV–êÎ[=NÒÒÃW·+§Ô‹ü×#ªµÍJïÓ{.Q\‘‰hS•Gˆ¬L™re¶í;{ñ`Å‚º¬i4§FU:ax³zvÑòôŸò7À®?”¶oåeNŒaÉeøš¤N	Hëáç™™ò„‰HœÙ£¤JE’#ZBE’	"¡"ÉE	¤$ @        ~«ò’³Û™XO3BøO,¡´–H“ây5qLpEt.ÓRÉE¾H¼i¥Ïrä9(ó&æWQ	OUÉ¾lhê† .H@ P      ¤ãÔ¸“bJy6çáhÍ§¿#DÓ[“8²L,Y©ád”ò²ŠT}Ãs=”o/,ÒšôLù›%…ƒRÅ|€.€       € ‚@EpA`ÑEHh±EpU¢ø#ÁÁfÑ4Ái¦x#øHÁSJ`¬©ÆkYFƒ<8ªY¼æ>ÆSèµsêÿ 3¿']c-¡ËNÑ-ê<ûĞ¢’ÂI"Ø'¢"<1kM¼«‚pN	ÁvÎ‘‚pY"R"é	‘8%"mL‘8'T`”‰À
0N€@ ’  (        D½WäHÄÒ1)%‰4L¥SŞÒĞ9K/g±˜·3¤Ê~9€oNs;B9y|ˆŒ\™ªI-‰2µ€ eĞ          +('ºØ¶Rê†Sê‚N™8µÍlUÁ?a­³5V2á~ÁSÖ÷MçbTõºŞSSà„z²à–â49IçcA¢'` (        A #ˆ(¨-‚0QVˆÁa€šS`¾ÀE0FàŒSpš`Œi¥08Kà`lÒ˜/³JàœÁ8&ÅpN	Á8"£àœ$€ Á8$‚	 €H@            gÆÔŸ§1¤‰Ú³YEÒF¥%ò¹–%&=ÉË¢(YA¾e•4]Ä3©–i7È¼a¶åÒK’¥Í“kÇ GñD‘           I®¥MLä°ÍD±hšêYMu(“rÕ4ù$Úäh¤™&‰ÚH“Â$¤ù’|(kU­ŞK,Ô °            ‚@A "¸ˆ(¨Á8(®Á`J‘‚ä`
à`¶+‚pN `°‘‚A8	Á8S                  Ÿ6^l’Ä˜‹Ä‘¯g?Ğ ÚK,ËaW4¹nUÉ²¦fÉrl`%—‚²˜Ç/ØhBXXD™™tˆĞ 
          “X`›ƒ\·Dâ™vÄÕ˜,áàÊá®…ÚjWŒ³æV^³!<=‰õ¤{LRá, fZˆĞ
ÎXÙVA´€€    ã¼¾­kUB…{„Öx©ğá{7g?é{Ÿì{Ïş?™¸¥¦6å9k©gwÚ>Æö¥¥uUN›Iµ®Yñöº~§m©Óu-œåõ”W¹¾gçÚõYWÖ®jN”èÊMfÆVËÀô½œÔkÛè”iÃM¹­åéÃ‡Ò~,úøu¦
Ş¾gO™Çç^ùíKxû>–¡Ú;-2ïè×
¯
–c¬?y¾¬ÙêJÖU%Â·n›I{ù'´÷¹Õ»ÉÛÔ û´¸*c=wØú}”¾­k§V+
÷ªß<a<-·büJ×#Êcç^Ü™Ç>Ÿô{¬ã++ <2¥ÚXê’Ô#mUT“Ş;pµöqG³³­V½­:•èJ…I/Jœšxg6·;‰{°r>ìÌMf4Øà¡­i÷•-!qZœ¸xe·öx˜vVZ^ø%ı=_Fšğñ~ãÃéÚeŞ©qÁo÷ôª>Qóg~?2cœ—CÍÊæÛXÇ7>ïÒk×£mIÕ¯R4àšNRxK;„áR
p’”_&S>³g;ÊW£Rê¥ÃN•N’åì<çg»úºµjwU¨F¯§,r‹~]>,dÅl‘o—8óW×Ëô3¾«emyJÕã
³YIò^}D×şÖ¼øÇò<& ¤µ˜Î¤ªJ5e9<·‡Ç\Ö˜êNg2ø+Óåúi>ogiÔ†BU+Î¯x—Õ^úg’ñÓi¯Ãßıt‹kÊ0	Á’„\¤ÒIe·ĞËaCUíÅÍIS´›¥EmÅ¥/nz)\İE÷ŠµU¿­ÄşgĞ§ö®í:|ÌŸR¥mªÆß¢ƒÈé=¤¯F¬i^ÍÕ¤Ş8ß­Í½5$šyO“G—6bYìÁÈ¦zî¨ÀÁ$œ^„` ù÷ÚİqjRj9XÏùg­X_UT­ë9Mı^	/Àó¯ıëOîWÍ›ö3õ÷_Ã›=óÆ§ğÿ wİñ£–yŸc¶¦uiÒIÔœ`ŸÚx&)ÔY„ã/'“ëkk«gJéG‚O	·Œ>˜öRÓî4{Î9$÷§R;e~g+›·V¥éåòïÆüºwĞAç;5­Wº¨ìîdç%(MóÛ£=%–qËŠØ­Óg§™ñÅê’¶­§Ûç¼»¤šè¥—ğGÎ¸íe…=¨Â¥gÑ¥Â¿˜®,—ñNN~«CîÒYo¹íeìö£FSäß¤ÿ ×¸ùÎ¶§«MÇµÃ[¸®KİÈõWƒ}nó“êø¢zqÄÚ^Â¾»¦ÛÕTås&ñèî—›;ã%(©E¦Ÿ&O