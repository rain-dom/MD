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

#### Cookie

```java
public class CookieServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //防止乱码
        req.setCharacterEncoding("UTF-8");
        resp.setCharacterEncoding("gbk");
        resp.getWriter().write("传达到了");
        Cookie cookie = new Cookie("01","this is 第一");
        //添加有效时间
        cookie.setMaxAge(7*24*3600);
        resp.addCookie(cookie);
    }
}
```

获取cookie

```java
     Cookie[] cookies = req.getCookies();
    for (Cookie cookie : cookies) {
        String key = cookie.getName();
        String value = cookie.getValue();
        System.out.println(key +"="+value);
    }
}
```



#### Seesion

![session](E:\deng\deng\MD\mvc\image\session.png)

用户使用浏览器第一次向服务器发送请求，服务器在接受到请求后，调用对应的Servlet 进行处理。在处理过程中会给用户创建一个session对象，用来存储用户请求处理相关的公共数据，并将此session 对象的JSESSIONID以sessoin的形式存储在浏览器中（临时存储，浏览器关闭即失效）。用户在发起第二次请求及后续请求时，请求信息中会附带JSESSIONID，服务器在接收到请求后，调用对应的Servlet 进行请求处理，同时根据JSESSIONID 返回其对应的 session对象

**作用：**

*      解决相同用户发送不同请求时的数据共享问题

**特点：**

*      1、服务器端存储共享数据的技术
*      2、session需要依赖cookie技术
*      3、每个用户对应一个独立的session对象
*      4、每个session对象的有效时长是30分钟
*      5、每次关闭浏览器的时候，重新请求都会开启一个新的session对象，因为返回的JSESSIONID保存在浏览器的内存中，是临时cookie，所以关闭之后自然消失

**使用：**

*      获取session对象
*      HttpSession session = request.getSession();
*      session.setAttribute("name",u.getname())
*      修改session会话的保持时间
*      session.setMaxInactiveInterval(int second);
*      设置强制失效
*      session.invalidate();

#### ServletContext

* 运行在JVM上的每一个web应用程序都有一个与之对应的Servlet上下文（Servlet运行环境）

* Servlet API提供ServletContext接口用来表示Servlet上下文，ServletContext对象可以被web应用程序中的所有servlet访问
* ServletContext对象是web服务器中的一个已知路径的根

**使用**

ServletContext context = this.getServletContext();

context .setAttribute("name",u.getname())

![contextAPI](E:\deng\deng\MD\mvc\image\contextAPI.png)

#### ServletConfig

**作用：**
ServletConfig对象是Servlet的专属配置对象，每个Servlet都单独拥有一个ServletConfig对象，用来获取web.xml中的配置信息
**使用：**
-获取ServletConfig对象
-获取web.xml中的servlet配置信息





#### 过滤器

继承Filter类,重写方法实现逻辑

在web.xml中配置过滤规则



存在U型执行链

#### 监听器

![监听器](E:\deng\deng\MD\mvc\image\监听器.png)



### jsp

java server page

因为HTML页面是固定的，而直接接在servlet中写html会导致耦合度过高，因此催生了jsp

**属性**：jsp是一种动态网页技术

**本质**： 是servlet也是jsp，通过jsp引擎转译成servlet

#### **工作原理**

![jsp](E:\deng\deng\MD\mvc\image\jsp.png)

执行过程

客户端显示的访问路径不会变 

<%--
    html注释
        也会被转译成java文件，会发送给浏览器，但是浏览器不会执行
    java注释
        也会被转译，但是不会发送给浏览器
    jsp注释
        不会被转译
--%>
<%--
    page:
        用来设置转译成servlet文件时的参数
            contentType:表示浏览器解析响应信息的时候使用的编码和解析格式
            language：表示jsp要转译成的文件类型
            import:导入需要的jar包，多个包使用逗号分隔
            pageEncoding:设置页面的编码格式
            session:用来控制sevlet中是否有session对象
            errorPage:当页面发生逻辑错误的时候，跳转的页面
            extends:需要要转译的servlet类要继承的父类（包名+类名）
    局部代码块：
        可以将java代码跟页面展示的标签写到一个jsp页面中，java代码转译成的servlet文件中，java代码跟输出是保存在service方法中的
        缺点：
            代码可读性比较差，开发比较麻烦
            一般不使用
    全局代码块：
        定义公共的方法的时候需要使用全局代码块使用<%!%>、，生成的代码在servlet类中
        调用的时候需要使用局部代码块
    脚本调用方式
        使用<%=直接调用变量和方法（必须有返回值）%>
        注意：不能再变量或方法的后面添加；
    页面导入的方式
        静态导入：
            <%@ include file="staticImport.jsp"%>  file中填写的是jsp文件的相对路径
            不会将静态导入的页面生成一个新的servlet文件，而是将两个文件合并，
            优点：运行效率高
            缺点：两个页面会耦合到一起，不利于维护，两个页面中不能存在相同名称的变量名
        动态导入：
            <jsp:include page="dynamicImport.jsp"></jsp:include>
            两个页面不会进行合并，分别生成自己的servlet文件，但是页面在最终展示的时候是合并到一起的
            优点：没有耦合，可以存在同名的变量

#### 请求转发：

​            在jsp中也可以实现servlet的请求转发功能
​            <jsp:forward page="forward.jsp"></jsp:forward>  page填写的是jsp页面的相对路径
​            注意：在标签中间不可以添加任何字符.除了<jsp:param name="" value="">
​            在转发的页面中想要获取到属性值通过request.getParameter(String key)

#### jsp九大内置对象：

​            jsp页面在转移成其对应的servlet文件的时候，会自动声明一些对象，在jsp页面中可以直接使用
​            注意：内置对象是在jsp页面中使用的，可以在局部代码块中使用，也可以在脚本段语句中使用，但是不能再全局代码块中使用

​            九大对象：
​                **pageContext**：表示页面的上下文的对象封存了其他的内置对象，封存了当前页面的运行信息
​                            注意：每一个页面都有一个对应的pagecontext对象，
​                            伴随着当前页面的结束而结束
​                **request**：封装当前请求的数据，由tomcat创建，一次请求对应一个request对象
​                **session**：用来封装同一个用户的不同请求的共享数据，一次会话对应一个session对象
​                **application**：相当于ServletContext对象，一个web项目只有一个对象，存储着所有用户的共享数据，从服务器启动到服务器结束
​                **response**：响应对象，用来响应请求数据，将处理结果返回给浏览器，可以进行重定向
​                **page**:代表当前jsp对象，跟java中的this指针类似
​                **exception**：异常对象，存储当前运行的异常信息，必须在page指令=中添加属性isErrorPage=true
​                **config**:相当于Serlverconfig对象，用来获取web.xml中配置的数据，完成servlet的初始化操作
​                **out**：响应对象，jsp内部使用，带有缓存区的响应对象，效率要高于repsonse

#### 四大作用域：

​			小 ---------------------------> 大

​                **pageContext**：表示当前页面，解决当前页面内的数据共享问题，获取其他内置对象
​                **request**：一次请求，一次请求的servlet的数据共享，通过请求转发的方式，将数据流转到下一个servlet
​                **session**：一次会话，一个用户发送的不同请求之间的数据共享，可以将数据从一个请求流转到其他请求
​                **application**：项目内，不同用的数据共享问题，将数据从一个用户流转到其他用户
​        路径问题：
​                想要获取项目中的资源，可以使用相对路径，也可以使用绝对路径
​                **相对路径**：相对于当前页面的路径
​                    问题：1、资源的位置不可以随便更改
​                          2、需要使用../的方式进行文件夹的跳出，如果目录结果比较深，可以操作起来比较麻烦
​                **绝对路径**：
​                    在请求路径的前面加/,表示当前服务器的根路径，使用的时候要添加/虚拟项目名称/资源目录
​                使用jsp中自带的全局路径声明：

​                    String path = request.getContextPath();
​                        System.out.println(path);
​                    String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
​                        System.out.println(basePath);


                    <base href="<%=basePath%>">

​                    添加资源路径的时候，从当前项目的web目录下添加即可





#### EL

EL表达式可以进行算术运算和关系运算

*          直接在表达式中写入算法操作即可，如果是关系运算，返回true或者false
*          注意：在el表达式中的+表示加法操作而不是字符串连接符
*      EL表达式可以进行逻辑运算
*          ${true&&false}<br>
*          ${true&&true}<br>
*          ${true||false}<br>
*          ${true||true}<br

EL表达式获取header信息

*          ${header}:获取所有请求头信息
*          ${header[key]}:获取指定key的数据
*          ${headerValues[key]}:获取key对应的一组数据，返回类型是数组
*          ${headervalues[key][0]}:获取key对应数组的某一个值

EL表达式获取cookie数据：

*          ${cookie}:获取cookie中的所有数据
*          ${cookie.key}：获取cookie中指定key的数据
*          ${cookie.key.name}：获取cookie指定key数据的name
*          ${cookie.key.value}：获取cookie指定key数据的value

#### 登录小程序



Seesion