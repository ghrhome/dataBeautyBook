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

