PostMessage应用Demo

PageA:

```
<!DOCTYPE html >
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>跨文档通信之窗体通信 </title>
    <style>
        #boy, #girl{
            display:inline-block;
            padding:0.2em 1em;
            outline:0;
            border-radius:3px;
            box-shadow:0 1px rgba(0, 0, 0, .2), inset 0 0 1px rgba(255,255,255,1), inset 0 .9em rgba(255,255,255,0.15);
            font-weight:bold;
            text-decoration:none!important;
            letter-spacing:.1em;
        }
        #boy{background-color:#2DA200; border:1px solid #1F7F00; color:#fff; text-shadow:0 -1px #137900;}
        #girl{background:#EF3830; border:1px solid #980205; color:#fff; text-shadow:0 -1px #D42521;}
        #boy:active,#girl:active{
            box-shadow:0 1px rgba(0, 0, 0, .2), inset 0 0 1px rgba(0,0,0,.5), inset 0 .9em rgba(255,255,255,0.1);
        }
    </style>
</head>
<body>
<div id="header">
    <h3>效果：</h3>
    <div class="show">
        <div class="demo">
            <a target="_blank" href="P2.html" id="boy">男生点这个</a>
            <a target="_blank" href="P2.html" id="girl">女生点这个</a>
            <p style="padding-top:160px;"></p>
        </div>
    </div>
</div>
</div>
</div>
<script>


    var message = '';

    document.querySelector("#boy").onclick = function() {
        message = '我是男生，帅气的男生！';
    };
    document.querySelector("#girl").onclick = function() {
        message = '我是女生，漂亮的女生！';
    };


    window.addEventListener('message', function(e) {
        var interval;
        // 检测来源
        console.log(e);
        console.log(arguments);

        if( e.origin == 'http://192.168.120.241:8000')
            switch (e.data) {
                case 'ready':
                    // e.source 为发送的 window 对象
                    message=message+"---"
                    interval = setInterval(function(win) {
                        console.log("e....source")
                        console.log(e.source);
                        console.log(win);

                        win.postMessage(message,'http://192.168.120.241:8000/');
                    }, 2000, e.source);
                    break;

                case 'closed':
                    alert('close');
                    clearInterval(interval);
                    break;
            }
    }, false);
</script>



</body>
</html>
```

Page B

```
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>测试窗体</title>
    <link rel="stylesheet" href="http://s1.xiaomishu.com/b/css/min/base.css">
    <style>
        body{min-width:0;}
    </style>
</head>
<body>
<div id="message" class="p20"></div>
<script>
    console.log(window.opener);
    window.addEventListener('message', function(e) {
        document.querySelector("#message").innerHTML = "接受到的信息是：" + e.data;
        console.log((new Date()).getTime());
        console.log(e.data)
    }, false);

    // 当文档加载完毕, 给父级来源发送信息。
    window.addEventListener('load', function(e){
        console.dir(e);
        console.log(arguments);
        console.dir(e.currentTarget.opener);
        e.currentTarget.opener.postMessage('ready', 'http://192.168.120.241:8000');
    }, false);

    // 当窗体关闭，告诉父级窗体已经关闭了。
    window.addEventListener('unload', function(e){
        e.currentTarget.opener.postMessage('closed', 'http://192.168.120.241:8000');
    }, false);
</script>


</body>

```



