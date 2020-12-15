# IO

### 1、阻塞与非阻塞

阻塞与非阻塞是描述进程在访问某个资源时，数据是否准备就绪的的一种处理方式。当数据没有准备就绪时：

- 阻塞：线程持续等待资源中数据准备完成，直到返回响应结果。
- 非阻塞：线程直接返回结果，不会持续等待资源准备数据结束后才响应结果。

### 2、同步与异步

- 同步与异步是指访问数据的机制，同步一般指主动请求并等待IO操作完成的方式。
- 异步则指主动请求数据后便可以继续处理其它任务，随后等待IO操作完毕的通知。

1、普通水壶煮水，站在旁边，主动的看水开了没有？同步的阻塞
 2、普通水壶煮水，去干点别的事，每过一段时间去看看水开了没有，水没开就走人。 同步非阻塞
 3、响水壶煮水，站在旁边，不会每过一段时间主动看水开了没有。如果水开了，水壶自动通知他。 异步阻塞
 4、响水壶煮水，去干点别的事，如果水开了，水壶自动通知他。异步非阻塞



## Linux IO模型

Linux IO

·将I/O模型划分为以下五种类型：

>阻塞式/O模型
>非阻塞式/O模型
>I/0复用
>信号驱动式/O
>异步I/O

![LinuxIO](E:\deng\deng\MD\image\LinuxIO.png)

### 阻塞式IO模型

![阻塞IO](E:\deng\deng\MD\image\阻塞IO.png)

发起请求后，调用内核，从磁盘拷贝到内核空间，再从内核空间拷贝到应用空间

整个过程阻塞，线程调度是需要成本的，浪费了cpu

### 同步非阻塞IO模型

![非阻塞IO模型](E:\deng\deng\MD\image\非阻塞IO模型.png)

用户进程轮询调用kernel询问是否就绪,每一个都是询问kernel，kernel再返回结果。

内核再发生系统调用，引发成本问题。





### IO复用模型

![多路复用IO](E:\deng\deng\MD\image\多路复用IO.jpg)

等待可用的套接字

将1000个文件描述符fd传到kernel,让它自己监控，可用的fd返回给用户态，用户态再根据这个fd去kernel调用read(read就不会调用没有数据的kernel)，降低了用户空间的复杂度。

减少了内核态和用户态的切换

* 但是关于fd可不可用的数据拷来拷去

### 多路复用NIO

![多路复用NIO](E:\deng\deng\MD\image\redis\多路复用NIO.png)

用户态和内核态占用的不同空间是理论上的，在物理空间上，他们只是一片存储的不同物理位置。这么划分是为了安全考虑的。

因此可以拿出一块共享空间，也就是mmp

原本放在自己用户空间的fd现在放到mmp中的红黑树，那么用户态现在只需要调用系统调用而不需要传fd,kenel直接可以在红黑树中看到数据，把可用的放入链表中。

用户直接从链表中调用read



### 零拷贝又是什么

![零拷贝](E:\deng\deng\MD\image\redis\零拷贝.png)

jvm用户空间可以直接访问kernel的一块内存，不用再拷贝到jvm中再使用

以前是read到用户态，再write回内核，再发给网卡

调用sendfile后省去了拷贝的过程，kernel直接发给网卡

* kafka是两者的结合，file放在共享空间，内核通过sendfile零拷贝调用file发给消费者

### 信号驱动IO

![信号驱动IO](E:\deng\deng\MD\image\信号驱动IO.jpg)

### 异步IO

![异步IO](E:\deng\deng\MD\image\异步IO.png)

## Java IO模型

### Blocking IO

面向流

![Socket](E:\deng\deng\MD\image\Socket.png)





## NIO

三大核心部分：（面向buffer）

* Channels
* Buffers
* Selectors

![NIObuffer](E:\deng\deng\MD\image\NIObuffer.png)













### 1.Buffer

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

三大核心概念：（面向buffer）

* position
* limit
* capacity

![img](http://ifeve.com/wp-content/uploads/2013/06/buffers-modes.png)

![ByteBuffer](E:\deng\deng\MD\image\ByteBuffer.png)

heapByteBuffer不能直接从用户空间拷贝到内核空间，需要申请native内存空间过渡



#### 用法

1. 写入数据到Buffer
2. 调用`flip()`方法
3. 从Buffer中读取数据
4. 调用`clear()`方法或者`compact()`方法 

**向Buffer中写数据**

`inChannel.read(buf)`

`buf.put`

**从Buffer中读取数据**

`inChannel.write(buf)`

`buf.get();`

| 方法                | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| rewind()            | Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据,limit保持不变. |
| clear()、compact(） | 都是改变position和limit                                      |
| mark()与reset()方法 | 可以标记Buffer中的一个特定position。<br />调用Buffer.reset()方法恢复到这个position |
| equals()            | 当满足下列条件时，表示两个Buffer相等：<br />1.有相同的类型（byte、char、int等）。<br />2.Buffer中剩余的byte、char等的个数相等。<br />3.Buffer中所有剩余的byte、char等都相同。 |
| compareTo()         | compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：<br />1.第一个不相等的元素小于另一个Buffer中对应的元素<br />2.所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少) |

**mark()与reset()方法**



### 2.Selector

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。

![overview-selectors](E:\deng\deng\MD\image\overview-selectors.png)

要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。

#### Selector的创建

通过调用Selector.open()方法创建一个Selector，如下：

`1` `Selector selector = Selector.open();`

**向Selector注册通道**

为了将Channel和Selector配合使用，必须将channel注册到selector上。通过SelectableChannel.register()方法来实现

~~~java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,Selectionkey.OP_READ);

~~~

与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。

注意**register()**方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

1. Connect
2. Accept
3. Read
4. Write

通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

这四种事件用SelectionKey的四个常量来表示：

1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE

如果对多个事件感兴趣，可以这么写。

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

#### 通过Selector选择通道

一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。

- int select()
  - 阻塞到至少一个通道在注册的时间上就绪了
- int select(long timeout)
  - 最长会阻塞 timeout
- int selectNow()
  - 不阻塞，直接返回，无就返回0

select()方法返回的int值表示有多少通道已经就绪。

如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。



channel要注册到select上，会返回一个SelectionKey

这样可以通过SelectionKey的selectedKeySet()方法访问这些对象。

~~~java
//返回SelectionKey集合
Set selectedKeys = selector.selectedKeys();

Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
}

~~~





### 3.Channel

![img](http://ifeve.com/wp-content/uploads/2013/06/overview-channels-buffers.png)



#### Channel的实现

#### FileChannel

FileChannel 从文件中读写数据。

**案例**-使用FileChannel读取数据到Buffer中

```java
public static void main(String[] args) throws Exception {

    RandomAccessFile aFile = new RandomAccessFile("1.txt", "rw");
    //读取文件获取channel
    FileChannel channel = aFile.getChannel();
    //给ByteBuffer分配大小
    ByteBuffer buf = ByteBuffer.allocate(10);
    //读取channel到buff（最多读上上限停止）
    int bytesRead = channel.read(buf);
    while (bytesRead != -1){
        System.out.println("read" + bytesRead);
        System.out.println("1ci ");
        //翻转为读状态
        buf.flip();
        while (buf.hasRemaining()){
            System.out.println((char)buf.get());
        }
        buf.clear();
        bytesRead = channel.read(buf);
    }
    aFile.close();

}
```

- DatagramChannel
  - DatagramChannel 能通过UDP读写网络中的数据。

#### SocketChannel

SocketChannel 能通过TCP读写网络中的数据。	

**从 SocketChannel 读取数据**

```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://www.baidu.com",8080));
```

**写入 SocketChannel**

```java
public static void main(String[] args) throws Exception {
    SocketChannel socketChannel = SocketChannel.open();
    String newData = "New String to write to file..." + System.currentTimeMillis();
    ByteBuffer buf =  ByteBuffer.allocate(48);
    buf.clear();
    buf.put(newData.getBytes());
    
    buf.flip();
    while (buf.hasRemaining()){
        socketChannel.write(buf);
    }
}
```

##### 非阻塞模式

设置之后，就可以在异步模式下调用connect(), read() 和write()了。

~~~java
socketChannel.configureBlocking(false);

//在非阻塞模式下，此时调用connect()，该方法可能在连接建立之前就返回了。
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

//为了确定连接是否建立，可以调用finishConnect()
while(! socketChannel.finishConnect() ){
    //wait, or do something else...
}

~~~



#### ServerSocketChannel

- ServerSocketChannel可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。
- 

```java
public static void main(String[] args) throws IOException {
    //创建
    ServerSocketChannel ssc = ServerSocketChannel.open();
    //监听端口
    ssc.socket().bind(new InetSocketAddress(9999));
    //设置非阻塞模式
	serverSocketChannel.configureBlocking(false);
    
    //注册到Selector,并关注accept事件
    ssc.register(selector, SelectionKey.OP_ACCEPT);

    
    while (true){
        SocketChannel socketChannel = ssc.accept();
        //do something with socketChannel...
    }
    
    ssc.close();

}
```







* 可以从流中获得通道
  * FileInputStream fin = new FileInputStream (infile)
  * FileChannel fcin = fin.getChannel()





### scatter/gather

**Scattering Reads**

指数据从一个channel读取到多个buffer中。如下图描述：

![Java NIO: Scattering Read](http://ifeve.com/wp-content/uploads/2013/06/scatter.png)

~~~java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
~~~

Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息(消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。



**Gathering Writes**

![Java NIO: Gathering Write](http://ifeve.com/wp-content/uploads/2013/06/gather.png)

~~~java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);
//write data into buffers
ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);

~~~

buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入。

因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。

### 通道间的数据传输

**transferFrom()**

~~~java
01RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

toChannel.transferFrom(position, count, fromChannel);

~~~

从toChannel的position处开始向目标文件写入数据

count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。

SocketChannel只会传输此刻准备好的数据（可能不足count字节）

**transferTo()**

两级反转，其它一样





### 编程流程

#### Dispatcher

```java
class Dispatcher{
    private int port;
    private EventHandler handler;
    public Dispatcher(final int port, final EventHandler handler){

    }
    public void run() throws IOException {
        //创建selector
        Selector selector = Selector.open();
		//创建ServerSocketChannel
        ServerSocketChannel listenChannel = ServerSocketChannel.open();
        //绑定监听端口
        listenChannel.socket().bind(new InetSocketAddress(this.port));
        //设置为非阻塞
        listenChannel.configureBlocking(false);
		//注册到selector并设置关注ACCEPT事件
        listenChannel.register(selector,SelectionKey.OP_ACCEPT);

        while (!Thread.interrupted()){
            if (selector.select(3000) == 0) continue;
            Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
            while (keyIterator.hasNext()){
                SelectionKey key = keyIterator.next();
                if (key.isAcceptable()){
                    //处理时间
//                    handler.handleAccept(key);
                }
                if (key.isReadable()){
//                    handler.handleReade(key);
                }
                if (key.isWritable()){
//                    handler.handleWrite(key);
                }
                keyIterator.remove();
            }
        }
    }
}
```

#### EventHandler

~~~java

~~~



# Netty

### 架构

![img](https://upload-images.jianshu.io/upload_images/1089449-78814cbb3acc30bd.png?imageMogr2/auto-orient/strip|imageView2/2/w/525/format/webp)



![netty解析](E:\deng\deng\MD\image\netty解析.png)







**服务端**

循环的部分抽象出EventLoopGroup

创建出2个group

bossGroup只关注accept类的事件

ServerBootstrap配置server的相关参数，再启动

业务逻辑写在ChildChannelHandler

具体处理在EchoServerHandler

ctx.write 写到缓冲区

ctx.writeAndFlush写到网络上

**客户端**

客户端只需要创建一个group,客户端不存在接收，只有连接

具体处理在EchoClientHandler





**Netty主要组件**：

**Transport Channel**----对应NIO中的channel
**EventLoop**----对应于NIO中的while循环
**EventLoopGroup**：多个EventLoop
**ChannelHandler**和**ChannelPipeline**---对应于NIO中的客户逻
辑实现handleRead/handleWrite
**ByteBuf**----对应于NIO中的ByteBuffer
**Bootstrap**和**ServerBootstrap**---对应NIO中的Selector、
ServerSocketChannel等的创建、配置、启动等

![netty主要组件](E:\deng\deng\MD\image\netty主要组件.png)

![netty运行流程](E:\deng\deng\MD\image\netty运行流程.png)

EventLoop

![EventLoop](E:\deng\deng\MD\image\EventLoop.png)

### ByteBuf

——Netty Buffer

![ByteBuf](E:\deng\deng\MD\image\ByteBuf.png)

![ByteBuf比较](E:\deng\deng\MD\image\ByteBuf比较.png)





### Unpooled工具类





### ChannelHandler

业务处理核心逻辑，自定义

channelHandlerAdapter 只需要重写自己要的方法，其他自动帮你重载

Channel生命周期



















