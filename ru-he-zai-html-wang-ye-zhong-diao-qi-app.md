# 如何在HTML网页中调起APP

对于app打开而言最常规的打开就是通过url scheme的方式去打开你的app，如下的

```
myapp://
myapp://open
myapp://type=1&id=2sdeo223lwe
```

这些抛出都是以url的方式进行抛出，app捕捉到这些抛出去做相应的处理，本文对app的处理不做详细描述，app开发请自行谷歌百度。对于前端而言抛出的方式也有很多，而最理想的方式是通过iframe的src对其进行链抛出，来！说的在多都没有代码来的清晰，请看下面。

```
//实际上就是新建一个iframe的生成器
var  createIframe=(function(){
  var iframe;
    return function(){
        if(iframe){
            return iframe;
        }else{
            iframe = document.createElement('iframe');
            iframe.style.display = 'none';
            document.body.appendChild(iframe);
            return iframe;      
        }
    }
})()
```

之后我们还需要一个url scheme:

```
//生成一个url scheme,假设我们约定的scheme是myApp://type=1&id=iewo212j32这种形式的

var baseScheme = "myApp://"
var createScheme=function(options){
    var urlScheme=baseScheme;
    for(var item in options){
        urlScheme=urlScheme+item + '=' + encodeURIComponent(options[item]) + "&";
    }
    urlScheme = urlScheme.substring(0, urlScheme.length - 1);
    return encodeURIComponent(urlScheme);
}
```

这种scheme形式的其实不是最好的,根据我们踩过的坑，觉得约定为与http协议相近可能更好一些，具体的协议需要前端人员自己去和app端人员约定。

ok万事具备，iframe有了，urlScheme也有了，该去打开app了

```
var openApp=function(){
    var localUrl=createScheme();
    var openIframe=createIframe();
    if(isIos()){
        //判断是否是ios,具体的判断函数自行百度
        window.location.href = localUrl;
        var loadDateTime = Date.now();
        setTimeout(function () {
            var timeOutDateTime = Date.now();
            if (timeOutDateTime - loadDateTime < 1000) {
                window.location.href = "你的下载页面";
            }
        }, 25);
    }else if(isAndroid()){
        //判断是否是android，具体的判断函数自行百度
        if (isChrome()) {
            //chrome浏览器用iframe打不开得直接去打开，算一个坑
            window.location.href = localUrl;
        } else {
            //抛出你的scheme
            openIframe.src = localUrl;
        }
        setTimeout(function () {
            window.location.href = "你的下载页面";
        }, 500);
    }else{
        //主要是给winphone的用户准备的,实际都没测过，现在winphone不好找啊
        openIframe.src = localUrl;
        setTimeout(function () {
            window.location.href = "你的下载页面";
        }, 500);
    }
}
```

以上就是你要打开scheme的主要代码了，好吧，实际上不只是打开app，还要实现未打开的时候跳到下载页去。其中安卓实际上无论有没有打开都会跳到下载页去，而ios........好吧!按照网上的说法是浏览器失焦后会挂起脚本，呵呵，这是多老的ios版本的表现了，实际上现在的ios已经没有这么做，有些版本会跟安卓的表现一样，而有些则是直接跳转根本不会去打开，还有打开的时候那个恶心的系统弹窗是什么鬼。好吧，实际上至此你会发现，ios9.0以上的有些打不开直接跳，有些打得开还会有允许弹窗，而微信则是无论如何都打不开，实际上微信会在他的浏览器里拦截掉所有未经其允许的scheme包括app store。



转自：《怎么在网页中打开你的app》@AlfredMou -- segmentfault

