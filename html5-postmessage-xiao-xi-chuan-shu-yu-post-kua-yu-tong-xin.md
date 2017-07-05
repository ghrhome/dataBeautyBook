# HTML5 postMessage 消息传输与 POST 跨域通信

HTML5 的 postMessage 方法可实现不同窗体间互相通信。

postMessage 支持实现跨文档消息传输（Cross Document Messaging），并且可跨域传输信息。Internet Explorer 8, Firefox 3, Opera 9, Chrome 3和 Safari 4 以上版本浏览器都已支持 postMessage。

## 1. postMessage 功能简介 {#1. postMessage 功能简介}

postMessage 主要包含两个 API：

1\).消息监听：onmessage  
2\).消息发送：postMessage

使用步骤也很简单：

### 1.1.监听发送过来的消息 {#1.1.监听发送过来的消息}

```
function onMessage(e) {
    console.log(e, e.data);
        // 消息来源安全验证
    if(e.origin !="http://lzw.me") {
        return false;
    }
    // 消息处理...
}

window.addEventListener('message', onMessage, false);
```

注意：

e.data 即接收到的信息。  
由于该监听可接收任意发送过来的消息，故一般应对 e.origin 来源进行检测，如果不是预设的消息来源，应予以拒绝处理。

### 1.2.向其他窗体发送消息 {#1.2.向其他窗体发送消息}

首先获取要传送消息的窗体对象（如iframe），然后向该对象发送信息

```
var iframeWin = document.getElementsByTagName('iframe')[0].contentWindow;
iframeWin.postMessage('hello world!', "*");
```

注意：

postMessage 包含两个参数，第一个参数为发送的消息内容，为字符串类型。第二个为来源域限制。即限制接收窗体的 URL。

进行跨文档消息发送，首先要获取传送对象窗体。所以 postMessage 常用在从当前页创建/打开新窗体或新的 worker 线程中，实现双方通信。请与志文工作室一起来看如下使用示例。

## 2. worker 多线程 {#2. worker 多线程}

消息接收处理页面：

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<title>Test Web worker</title>
<script type="text/JavaScript">
function init(){
var worker = new Worker('compute.js');

//event 参数中有 data 属性，就是子线程中返回的结果数据
worker.onmessage= function (event) {
    // 把子线程返回的结果添加到 div 上
    document.getElementById("result").innerHTML += event.data+"<br/>";
};
}
</script>
</head>
<body onload="init()">
    <div id="result"></div>
</body>
</html>
```

消息发送 compute.js：

compute.js 中调用 postMessage 方法发送计算结果

```
var i, j, sum;

function timedCount() {
    for (j = 0, sum = 0; j < 100; j++) {
        for (i = 0; i < 100000000; i++) {
            sum += i;
        }
    }
    //  调用 postMessage 向主线程发送消息
    postMessage(sum);
}

postMessage("Before computing," + new Date());
timedCount();
postMessage("After computing," + new Date());
```

## 3. 跨域 {#3. 跨域}

同源跨域可通过修改 window.domain 方式欺骗解决。  
非同源跨域可使用 flash 控件或远程加载script文件的 jsonp 方式。

现在 postMessage 则可简单完成该需求，重要的是，它可以实现 jsonp 无法完成的跨域 POST 请求。

### 3.1 父窗体创建跨域iframe并发送信息 {#3.1 父窗体创建跨域iframe并发送信息}

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<title>跨域POST消息发送</title>
<script type="text/JavaScript">
function sendPost() {
    var iframeWin = document.getElementById("otherPage").contentWindow;
    iframeWin.postMessage( document.getElementById("message").value, "http://w.lzw.me:81");
}

window.addEventListener("message", function( event ) {
    console.log(event, event.data);
}, false);
</script>
</head>
<body>
<textarea id="message"></textarea>
<input type="button" value="发送" onclick="sendPost()">
<iframe src="http://w.lzw.me:81/other-domain.html" id="otherPage" style="display:none"></iframe>
</body>
</html>
```

### 3.2 子窗体接收信息并处理（如发起XMLHttpRequest请求） {#3.2 子窗体接收信息并处理（如发起XMLHttpRequest请求）}

other-domain.html

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<title>POST Handler</title>
<script src="//code.jquery.com/jquery-1.11.0.min.js"></script>
<script type="text/JavaScript">
window.addEventListener("message", function (event) {
    // 把父窗口发送过来的数据向服务器发送post请求
    var data = event.data;
    $.ajax({
        type: 'POST',
        data: "info=" + data,
        dataType: "json"
    }).done(function (res) {
        //可以向父窗体返回结果
        window.parent.postMessage(res, "*");
    }).fail(function (res) {
        window.parent.postMessage(res, "*");
    });
}, false);
</script>
</head>
<body>
</body>
</html>
```

## 4. postMessage 总结 {#4. postMessage 总结}

postMessage 解决了客户端不同窗体间的消息传递问题，特别是跨域消息发送，可解决跨域 POST 请求问题。

另外，解决客户端与服务器的双向实时通信，可参考 HTML5 的 webSocket API.

## 5. 相关参考 {#5. 相关参考}

[http://www.ibm.com/developerworks/cn/web/1301\_jiangjj\_html5message/](http://www.ibm.com/developerworks/cn/web/1301_jiangjj_html5message/)

--------------------补充 中间页面实现跨域

# **iframe跨域访问**

p.p1 {margin: 0.0px 0.0px 0.0px 0.0px; font: 15.0px Verdana; -webkit-text-stroke: \#000000}  
span.s1 {font-kerning: none}

js跨域是个讨论很多的话题。iframe跨域访问也被研究的很透了。

一般分两种情况：

一、 是同主域下面，不同子域之间的跨域；

同主域，不同子域跨域，设置相同的document.domian就可以解决;

```
 父页访问子页，可以document.getElementById\("myframe"\).contentWindow.document来访问iframe页面的内容；如果支持contentDocument也可以直接document.getElementById\("myframe"\).contentDocument访问子页面内容；
```

子页访问父页，可以parent.js全局属性

二、 是不同主域跨域；

前提，www.a.com下a.html，a.html内iframe调用了www.b.com下的b.html，b.html下iframe调用了www.a.com下的c.html

b.html是不无法直接访问a.html的对象，因为涉及到跨域，但可以访问parent，同样c.html的parent可以访问b.html。c.html和a.html同域，是可以访问a下的对象的。parent.parent.js对象!

看下面实例：

a.html

```
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>无标题文档</title>
</head>

<body>
<iframe src="http://www.b.com/b.html" ></iframe>
<ul id="getText"></ul>
<script>
    function dosome(text){
        document.getElementById("getText").innerHTML= decodeURI(text);    
    }
</script>
</body>
</html>
```

b.html

```
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>无标题文档</title>
</head>

<body>
<iframe id="myfarme" src="###"></iframe>
<ul id="ct">
<li>这里是内容1</li>
<li>这里是内容2</li>
<li>这里是内容3</li>
<li>这里是内容4</li>
<li>这里是内容5</li>
<li>这里是内容6</li>
</ul>
<script>
    window.onload = function(){
        var text = document.getElementById('ct').innerHTML;
        document.getElementById('myfarme').src="http://www.a.com/c.html?content="+encodeURI(text);
    }
</script>
</body>
</html>
```

c.html

```
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>无标题文档</title>
<script>
window.onload = function(){
    var text = window.location.href.split('=')[1]
    console.log(parent.parent)
    parent.parent.dosome(text);
}
</script>
</head>

<body>
ddddddddddd
</body>
</html>
```



