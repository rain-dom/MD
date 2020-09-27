## 网络编程

### 网络编程的三要素

【1】IP地址：唯一标识网络上的每一台计算机两台计算机之间通信的必备要素
【2】端口号：计算机中应用的标号（代表一个应用程序,根据端口号识别发给谁）
		 		 	    0-1024系统使用或保留端口			
		  			   有效端口0-65536
【3】通信协议：通信的规则TCP，UDP



### TCP/IP

IP地址=网络ID+主机ID
-网络ID：标识计算机或网络设备所在的网段
-主机ID：标识特定主机或网络设备



InetAddress类







### 传输协议

**UDP**：相当于发短信（有字数限制），
不需要建立连接，
数据报的大小限制在64k内，
效率较高，不安全，容易丢包
**TCP**；相当于打电话，需要建立连接，
效率相对比较低，数据传输安全，
三次握手完成。
（点名>答到>确认）



### 小程序demo

传输的对象要实现序列化接口

```java
public class User implements Serializable {


    private static final long serialVersionUID = 1590020726742801370L;
```

#### LoginClient

```java
public class LoginClient {
    public static void main(String[] args) throws IOException {

        Socket client = new Socket("localhost",10000);
        OutputStream outputStream = client.getOutputStream();
        //完成登录功能，需要传输一个user对象
        User user = getUser();
        //传输对象需要ObjectOutputStream
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
        objectOutputStream.writeObject(user);
        //调用shutdown方法告诉对对方传输完成
        client.shutdownOutput();
        //接受响应
        DataInputStream dataInputStream = new DataInputStream(client.getInputStream());
        String str = dataInputStream.readUTF();
        System.out.println(str);
        //关闭流操作
        dataInputStream.close();
        objectOutputStream.close();
        outputStream.close();
        client.close();
    }

    public static User getUser(){
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入用户名:");
        String username = scanner.nextLine();
        System.out.println("请输入密码：");
        String password = scanner.nextLine();
        return new User(username,password);
    }
}
```

#### ThreadLoginClient.java

```java
public class LoginThread  implements Runnable{

    private Socket socket;

    public LoginThread(Socket socket){
        this.socket = socket;
    }

    @Override
    public void run() {
        ObjectInputStream objectInputStream = null;
        DataOutputStream dataOutputStream = null;
        try {
            objectInputStream = new ObjectInputStream(socket.getInputStream());
            User user = (User) objectInputStream.readObject();
            String str = "";
            if("msb".equals(user.getUsername()) && "msb".equals(user.getPassword())){
                System.out.println("欢迎你："+user.getUsername());
                str = "登录成功";
            }else{
                str="登录失败";
            }
            socket.shutdownInput();
            dataOutputStream = new DataOutputStream(socket.getOutputStream());
            dataOutputStream.writeUTF(str);
            socket.shutdownOutput();

        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }finally {
            try {
                dataOutputStream.close();
                objectInputStream.close();
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```

#### LoginServer

```java
public class LoginServer2 {
    public static void main(String[] args) throws Exception {

        ServerSocket serverSocket = new ServerSocket(10000);
        while(true){
            Socket socket = serverSocket.accept();
            LoginThread loginThread = new LoginThread(socket);
            new Thread(loginThread).start();
        }
//        serverSocket.close();
    }
}
```





## web

### GET和POST请求方式的区别

GET		请求获取由Request-URI所标识的资源
POST	  在Request-URI所标识的资源后附件新的数据



1、get请求参数是直接显示在地址栏的，而post在地址栏不显示’
2、get方式不安全，post安全
3、get请求参数是又长度限制的，post没有限制，

### 常见响应状态码

![image-20200923085430804](E:\deng\deng\MD\mvc\image\1600822413(1).jpg)



![image-20200923085826757](E:\deng\deng\MD\mvc\image\image-20200923085826757.png)

### Tomcat

tomcat 就是 webserver

因为服务端不确定客户端请求什么时候来，因此需要长服务



#### 自定义tomcat

![自定义tomcat的流程图](E:\deng\deng\MD\mvc\image\自定义tomcat的流程图.jpg)

MyServlet实现MyHttpServlet接口，在xml中配置调用该类的请求类型

根据传入的 request和respone 的请求类型执行不同的逻辑

server根据请求的url匹配mapping中设置的servlet类型



### Servlet

一个Servlet负责对应的一个或一组URL访问请求，并返回相应的响应内容

创建自己的Servlet类，并实现HttpServlet接口，重写其service方法

在WEB-INF下的web.xml中添加请求与servlet类的映射关系



只用doget和dopost。因为父类默认先调用service再转到get或post执行



#### 生命周期

![图片1](E:\deng\deng\MD\mvc\image\图片1.png)

#### 错误类型

404：访资源不存在1290
-请求路径跟web.xml中填写的请求不一致
-请求路径的项目虚拟名称填写错误
405：
-请求的方式跟servlet中支持的方式不一致
500：服务器内部错误
-web.xml中servlet类的名称错误
-servlet对应的处理方法中存在代码逻辑错误



#### HttpServletRequest

存放客户端请求的参数



**请求行**

​	直接get即可

**请求头**

​	k：v键值对

**请求体**

​	根据标签中的name获取



#### HttpServletResponse

**设置响应头**
-setHeader（String key，String value）添加响应信息，key重复会覆盖addHeader（String key，String value）添加响应信息，key重复不会覆盖

**设置响应状态**
-sendEror（int num，String msg）添加响应状态

**设置响应实体

-getWriter0.write（msg）响应具体的数据给浏览器



#### 乱码

1、get请求

*      1、获取字符串之后使用new String(name.getBytes("iso-8859-1"),"utf-8")
*      2、设置request的编码格式，同时在server.xml中添加useBodyEncodingForURI=true的属性
*      3、在server.xml中添加URIEncoding="utf-8"

2、post请求

*      1、request.setCharacterEncoing("utf-8")
*  3、response响应编码
*       response.setCharacterEncoding("gbk");



#### 转发

共享数据

```
request.getRequestDispatcher("hello").forward(request,response);
```

#### 重定向

```
response.sendRedirect("hello");
```

#### web.xml

匹配后通过反射生成servlet对象执行service

```xml
<servlet>
    <servlet-name>myServlet</servlet-name>
    <servlet-class>com.dzp.MyServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>myServlet</servlet-name>
    <url-pattern>/first</url-pattern>
</servlet-mapping>
```