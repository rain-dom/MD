## 基础使用

主要用于处理静态请求，所以可以单个处理

tomcat多处理动态请求

### 安装Nginx

uname -a

> 查看内核版本，要2.6以上

yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel

> 安装环境-C++编译

tar -zxvf nginx-1.18.0.tar.gz

> 解压，cd 进文件夹

./configure

> 安装，生成一个objs文件夹

make  再 make install

> 构建可执行文件

cd /usr/local/nginx

> 跳到安装目录

./sbin/nginx

> 配置端口

curl 127.0.0.1

> 启动

./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-debug

> 在nginx安装目录下（有configure）,基于参数构建



/usr/local/nginx/sbin/nginx -s stop

> 停止运行，准备替换

cp nginx /usr/local/nginx/sbin/

> 进入objs目录，替换文件



**工作目录**

/usr/local/nginx/sbin/nginx

**查看进程**

ps -ef|grep nginx



### 控制命令

控制命令：
默认方式**启动**：
./sbin/nginx
指定配置文件启动
./sbing/nginx-c/tmp/nginx.conf
指定nginx程序目录启动
/sbin/nginx-p/usr/Local/nginx/
快速停止
./sbin/nginx-s stop
优雅停止
./sbin/nginx-s quit
·热装载配置文件
./sbin/nginx-s reload

重新打开日志文件
./sbin/nginx-s reopen

> 备份日志后，如果没有reopen，写入的还是原来的日志



### 使用流程

cp nginx.conf nginx.conf.backup

> 备份配置文件

找个位置创建自己的代理文件夹

#### 配置文件nginx.conf


```java
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;


    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;	//端口
        server_name  localhost;
		
        //请求地址
        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   00 502 503 504  /50x.html;
```
#### 修改配置文件

* usr/nginx/conf/nginx.conf

* 添加配置

  * ~~~
    server{
    	listen 80;
    	server_name www.dzp.com *.dzp.com;
    	root /usr/com/dzp/;
    	location{
    		index dzp.html;
    	}
    	
    }
    ~~~

* 在nginx目录下检查配置正确性
  
  * ./sbin/nginx -t
* 重载配置
  
  * ./sbin/nginx -s reload

#### 动静分离

提高访问效率

![配置动静分离](E:\deng\deng\MD\mvc\image\配置动静分离.png)

* 在com文件夹中创建静态文件夹static

* conf配置

  * ~~~java
    server{
    	listen 80;
    	server_name www.dzp.com *.dzp.com;
    	root /usr/com/dzp/;
    	
        //配置访问静态路径
        location /static {
            alias /usr/com/dzp/static/;
        }
        
        location {
    		index dzp.html;
    	}
        //配置正向代理，在自己的地址后加/baidu.html
        location = /baidu.html {
    		proxy_pass http://www.baidu.com
    	}
        
        
    	
    }
    ~~~

  * alias配置别名，直接找/usr/com/dzp/static/
    不然会去找/usr/com/dzp/static/static/

#### 黑白名单

在conf目录下创建

#### 下载限速

location /dowload { alias  }



## 性能优化实战

### 1.反向代理

代理对于客户端是透明的，

代理接受后再分发给不同的服务端，否则用户直接访问不同的端口服务过于麻烦

![反向代理](E:\deng\deng\MD\mvc\image\反向代理.png)

* 备份配置文件

  * mv nginx.conf nginx.conf.backup.2

* 恢复默认的配置文件

  * mv nginx.conf.backup nginx.conf

* 配置反向代理

  * location /dzp/ {

    ​	proxy_pass http://127.0.0.1:8010;}

  * 访问dzp会反向代理到8010服务

### 2.负载均衡

在server同级目录下配置 upstream，其中加入服务端口

在location 中加入proxy_pass 定位到upstream的别名

curl 127.0.0.1

> 访问地址  

默认分发权重是1:1，可调

* upstream backend{

  ​	server 127.0.0.1:8010 weight = 1;

  ​	server 127.0.0.1:8080 weight = 1;

  ​	server 127.0.0.1:8888 backup;	//备用，替补选手

  }
  
  * 其它的都不可用时，backup才会使用

#### upstream 相关参数

==

#### upstream 负载均衡算法

保证会话一致性

* 分发到不同tomcat的数据缓存交由redis
  * ![数据共享问题](E:\deng\deng\MD\mvc\image\数据共享问题.png)

* ll+weight: 轮询加权重（默认）
* 或通过hash(ip)%2=0,1算法，每次分发到统一tomcat
  * 容易分发不均匀，一个tomcat崩溃，数据就无了

* has(url)，可以用于处理静态资源（如图片）
  * ![虚拟节点](E:\deng\deng\MD\mvc\image\虚拟节点.png)
  * 中间会分布许多虚拟节点，一个节点挂掉只影响它到另一个节点的资源，不会影响全部资源



## Nginx高速缓存

1.案例分析

用图中架构的话，内网的IO较高

2.缓存配置

3。缓存清楚

yum -y install wget

> 安装 wget命令

wget http://labs.frickle.com/files/ngx_cache_purge-2.3.tar.gz

到nginx1.18目录下

./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-debug --add-module=/root/packages/ngx_cache_purge-2.3.tar.gz



## 性能调优

配置多个worker



* 正常来说，worker数量等于cpu核数，不然会出现争用。

  但是当需要读取大量硬盘文件时，worker数量可以多余cpu核数

worker_connections

* 最大连接数包括连接tomcat，不止与客户端的连接



## 架构 

![Nginx架构](E:\deng\deng\MD\mvc\image\Nginx架构.png)

* 基于epoll实现非阻塞

**早期**

* 每个连接都要通过内核去硬盘寻址，效率低下

* 加入一个select队列，判断是否有读写需求，再走内核。

  > 队列变长以后依然麻烦



