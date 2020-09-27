# AJAX

在页面没有重新加载的情况下实现页面的局部更新

![网页响应](E:\deng\image\网页响应.png)



基础使用

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="./js/jquery-3.4.1.js"></script>
</head>
<body>
    <h1>ajax小课堂开课啦</h1>
    <button>单击我向服务器端悄悄发起上行请求(get)</button>
</body>
</html>
<script>
    //给按钮绑定事件
    //第一个参数向服务器请求path
    //第二个参数额外传递一些数据（本处没写）
    //第三个参数，当服务器响应时会立刻执行一次
    
    //get
    $("button:eq(0)").click(function(){
        $.get("./data.txt",function(data){
            $("h1").html(data);
        })
    });
    //post
    $("button:eq(1)").click(function(){
        $.post("./data.txt",function(data){
            $("h1").html(data);
        })
    });


</script>
~~~



# JSON跨域

概述：跨域（前端当中术语）：当用户发起多次请求的时候，如果协议|域名|端口号不同情况下，

称之为跨域。



JSONP数据格式跨域so easy：实现原理是如下：

* 利用script标签src属性（本身就可以跨域请求数据）
* 利用的是将一个函数执行部分放置另外一个服务器上面

~~~html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    <select></select>
</body>

</html>
<script>
    //获取下拉框
    var select = document.querySelector("select");
    //声明一个同名函数
    function jQuery4494404(arr) {
        //遍历数组
        for (var i = 0; i < arr.length; i++) {
            //创建节点
            var op = document.createElement("option");
            op.innerHTML = arr[i].name;
            select.appendChild(op);
        }
    }
</script>
<!-- 将京东服务器调用部分引入 -->
<script src="https://fts.jd.com/area/get?fid=72&callback=jQuery4494404&_=1578732186910"></script>

~~~



# bootstrap

https://www.runoob.com/bootstrap/bootstrap-tutorial.html

~~~html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="./css/bootstrap.min.css">
</head>

<body>
    <div class="container">
        <div class="row">
            <div class="col-xs-6">么么哒</div>
            <div class="col-xs-6">哈哈哈</div>
        </div>
        <div class="row">
            <div class="col-xs-2">小明</div>
            <div class="col-xs-10">小123131</div>
        </div>
        <span class="glyphicon glyphicon-fullscreen"></span>
        <ul class="pagination">
            <li><a href="#">&laquo;</a></li>
            <li><a href="#">1</a></li>
            <li><a href="#">2</a></li>
            <li><a href="#">3</a></li>
            <li><a href="#">4</a></li>
            <li><a href="#">5</a></li>
            <li><a href="#">&raquo;</a></li>
        </ul>
    </div>
</body>

</html>

~~~

