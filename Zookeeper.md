它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。

简单来说zookeeper=文件系统+监听通知机制。

现在把这些配置全部放到zookeeper上去，保存在 zookeeper 的某个目录节点中，然后所有相关应用程序对这个目录节点进行监听，一旦配置信息发生变化，每个应用程序就会收到 zookeeper 的通知，然后从 zookeeper 获取新的配置信息应用到系统中。

![img](https://img-blog.csdn.net/20180712143454552?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2phdmFfNjY2NjY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





复制集群，写操作在主，读在从

![ZooKeeper Service](https://zookeeper.apache.org/doc/current/images/zkservice.jpg)

为了保证zookeeper的速度，尽量减少它存储的数据。

每个node不超过1M

![ZooKeeper's Hierarchical Namespace](https://zookeeper.apache.org/doc/current/images/zknamespace.jpg)



### 持久节点与临时节点



顺序一致性：客户端的更新将按发送顺序应用

原子性：要么更新到整个集群，要么失败

单系统映像：无论服务器连接到哪个服务器，客户端都看到同样的服务试图

可靠性：通过日志实现

及时性：系统的客户视图保证在特定时间范围内是最新的



session也会统一视图，所以

### watcher

Watcher（事件监听器），是Zookeeper中的一个很重要的特性。Zookeeper允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通知到感兴趣的客户端上去，该机制是Zookeeper实现分布式协调服务的重要特性。



**两类watch**

* new zk() 时候，传入的watch，这个watch是session级别的，跟path 、node没有关系。

### API

| 配置                    |                                  |
| ----------------------- | -------------------------------- |
| create /zkPro myData    | 创建znode                        |
| create -e /zkPro myData | 创建临时znode                    |
| create -s /zkPro myData | 序列化创建，规避分布式下覆盖问题 |
| ls /                    | 看/下子节点                      |
| get /ooxx               | 康康数据                         |
| set /ooxx "hello"       | 放数据，最多放1M（二进制安全）   |

### 配置

小岛（Island）——ZK Server Cluster
议员（Senator）—ZK Server
提议（Proposal）——ZNode Change（Create/Delete/SetData.…）
提议编号（PID）——Zxid（ZooKeeper Transaction ld）
正式法令——所有ZNode及其数据



| 配置                                                     |                                                       |
| -------------------------------------------------------- | ----------------------------------------------------- |
| tickTime=2000                                            | 心跳时间                                              |
| initLimt =10                                             | 10*2 的启动实现限制                                   |
| syncLimit=5                                              | 5*2的连接时间限制                                     |
| dataDir=/var/dzp/zk                                      | 放数据                                                |
|                                                          |                                                       |
| server.1=node01:2888:3888<br />server.2=node02:2888:3888 | 人为配置个数，行数/2 +1<br />3888端口号用于主挂时通信 |
|                                                          |                                                       |
|                                                          |                                                       |

### 搭建集群

要想随时随地执行命令，需要设置环境变量/etc/profile



| 命令               |              |
| ------------------ | ------------ |
| zkServer.sh start  | 开始         |
| zkServer.sh status |              |
|                    |              |
| zkCli.sh           | 命令行客户端 |
|                    |              |



* mv zoo_sample.cfg zoo.cfg
  * conf/zoo_sample.cfg 改名
  * 加设置
  * server.1=192.168.0.75:2888:3888
    server.2=192.168.0.75:2888:3888
    server.3=192.168.0.75:2888:3888
  * 3188用于集群间通信
* sh zkServer.sh start
* sh zkServer.sh status
  * 查看状态
* sh zkCli.sh



* scp -r zookeeper-3.4.6/ root@192.168.75.112:/usr/local
  * 拷贝到其他服务器
* echo "1">/var/dzp/zk/myid
  * 告诉服务器我是哪一个服务器，改成123
* scp -r ./zookeeper/ root@192.168.75.112:`pwd`
  * scp -r zookeeper-3.4.6 root@192.168.75.113:/root/packages/
  * 拷贝
* vi /etc/profile
  * 配置环境变量
  * :$ZOOKEEPER_HOME/bin
* . /etc/profile
* scp /etc/profile node02:/etc



* sh zkServer.sh start
  * 把三台都启动了
* sh zkCli.sh -server 192.168.75.111:2181,192.168.75.112:2181,192.168.75.113:2181



### Paxos/ZAB



Watch





### React

**反应式编程**

减少计算机空转。

原本是人为规划执行流程，阻塞住

现在是让计算机自行处理，减少cpu空转机会



### HA

zookeeper的高可用通过它的快速恢复leader来实现

任意两个node都可用直接连接



对外要保证



### 分布式锁

通过session实现

### IDEA使用

```java
public class App {
    public static void main( String[] args ) throws Exception {
        System.out.println( "Hello World!" );

        //zk是有session概念的，没有连接池的概念
        //watch:观察，回调
        //watch的注册值发生在 读类型调用，get，exites。。。
        //第一类：new zk 时候，传入的watch，这个watch，session级别的，跟path 、node没有关系。
        final CountDownLatch cd = new CountDownLatch(1);
        final ZooKeeper zk = new ZooKeeper("192.168.150.11:2181,192.168.150.12:2181,192.168.150.13:2181,192.168.150.14:2181",
                //设置了3秒过期时间
                3000, new Watcher() {
            //Watch 的回调方法！
            @Override
            //这里的event是不停刷新，初始化会触发多个event
            public void process(WatchedEvent event) {
                Event.KeeperState state = event.getState();
                Event.EventType type = event.getType();
                String path = event.getPath();
                System.out.println("new zk watch: "+ event.toString());

                switch (state) {
                    case Unknown:
                        break;
                    case Disconnected:
                        break;
                    case NoSyncConnected:
                        break;
                    case SyncConnected:
                        System.out.println("connected");
                        cd.countDown();
                        break;
                    case AuthFailed:
                        break;
                    case ConnectedReadOnly:
                        break;
                    case SaslAuthenticated:
                        break;
                    case Expired:
                        break;
                }

                switch (type) {
                    case None:
                        break;
                    case NodeCreated:
                        break;
                    case NodeDeleted:
                        break;
                    case NodeDataChanged:
                        break;
                    case NodeChildrenChanged:
                        break;
                }


            }
        });

        cd.await();
        ZooKeeper.States state = zk.getState();
        switch (state) {
            case CONNECTING:
                System.out.println("ing......");
                break;
            case ASSOCIATING:
                break;
            case CONNECTED:
                System.out.println("ed........");
                break;
            case CONNECTEDREADONLY:
                break;
            case CLOSED:
                break;
            case AUTH_FAILED:
                break;
            case NOT_CONNECTED:
                break;
        }

        //ACL权限控制、临时带序列
        String pathName = zk.create("/ooxx", "olddata".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        //准备存放拿到的源数据
        final Stat  stat=new Stat();
        byte[] node = zk.getData("/ooxx", new Watcher() {

            //这个节点有事件发生时触发process
            @Override
            public void process(WatchedEvent event) {
                System.out.println("getData watch: "+event.toString());
                try {
                    //true时是default Watch  被重新注册。是new zk注册的的那个watch
                    zk.getData("/ooxx",this  ,stat);
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //stat放的是zxid等这些源数据
        }, stat);

        System.out.println(new String(node));

        //触发回调,第三个参数version是什么
        Stat stat1 = zk.setData("/ooxx", "newdata".getBytes(), 0);
        //还会触发吗？不会，watch是一次性的
        Stat stat2 = zk.setData("/ooxx", "newdata01".getBytes(), stat1.getVersion());


        //callback是异步的，从
        System.out.println("-------async start----------");
        zk.getData("/ooxx", false, new AsyncCallback.DataCallback() {
            @Override
            //rc-状态码   ctx-"abc"   data-数据  stat-源数据
            public void processResult(int rc, String path, Object ctx, byte[] data, Stat stat) {
                System.out.println("-------async call back----------");
                System.out.println(ctx.toString());
                System.out.println(new String(data));

            }
        //传一个context
        },"abc");
        System.out.println("-------async over----------");



        Thread.sleep(2222222);
    }
}
```





