# requireJS——A篇

一、关于requirejs

requirejs是一个用于异步加载js模块的框架。详细介绍的请谷歌~

二、HOW TO USE

首先先去官网下载requirejs.js下来，再在自己的项目中引入

```
1 <script type="text/javascript" src="../requirejs.js" data-main="../main"></script>
```

注意到data-main这个属性，简单的理解就是一个入口函数，用来启动脚本的加载过程。

tip：为了使这个文件加载时候不失去响应，你可以选择将它放在网页底部加载，还有另外一个办法就是写成下面这样

```
<script type="text/javascript" src="../requirejs.js" data-main="../main" async="true" defer></script>
```

async这个属性表明它要异步加载，避免失去响应，另外defer是为了兼容IE浏览器（IE不支持async这个属性），所以完整的把两个都写上。

例子结构目录为

![](http://images.cnitblog.com/i/600316/201405/061451387761607.png)

三、简单的例子

假设js/a.js中的代码如下:

```
var  info = "hello world"//简单的定义了一个变量
```

然后在mian.js中用requirejs加载它，如下所示

```
1 require(["../a.js"],function(a){
2        alert(info);//弹出hello world
3  })
```

然后再查看浏览器的开发者工具下，可以看到已经加载了

![](http://images.cnitblog.com/i/600316/201405/061128179793462.png)

可以看到require接受3个参数（示例中为2个），第一个参数为标识的id（建议忽略）；第二个参数为一个字符型的数组，为要加载的js模块；第三个参数为回调函数，

当js模块注入加载完成后，此函数就会被调用，其中函数的参数，依次顺序对应第二个参数中字符串数组的值。

tip：第二个参数中字符串数组中允许不同的值，当字符串是以“.js”结尾的（例如上面中的js/a.js）或者是以“/”开头，又或者直接是一个完整的url（

包含URL协议，如"http:"、"https:"），则会被requirejs认为用户是直接加载一个js模块。

```
  否则，当字符串是类似”my/module”的时候，它会认为这是一个模块，并且会以用户配置的 baseUrl 和 paths 来加载相应的模块所在的 JavaScript 文件（后面将介绍）
```

四、配置

现在我们来换一种写法，我们使用require.config来对模块的加载行为自定义，其中参数是一个对象

```
require.config({
  paths:{
    jquery:"jquery-1.10.2.min",
    a:"a"
  }
});

require(["jquery","a"],function($,a){
  alert($().jquery);   //1.10.2
  alert(info);         //hello world
});
```

简单的理解就是参数对象中的path属性用来设置路径的。

由于requirejs是以相对于baseurl属性（示例中没有给出）地址来加载所以的代码的。其中baseUrl是为require.config的参数（参数为对象）里一个属性。

如果没有显式指定config及data-main，则默认的baseUrl为包含RequireJS的那个HTML页面的所属目录。

此时，RequireJS默认假定所有的依赖资源都是js脚本，因此无需在module ID上再加".js"后缀（即上面的jquery-1.10.2.min不用写成jquery-1.10.2.min.js

写上会报错（Uncaught Error: Script error for: jquery）。

五、用define自定义模块

```
//b.js

define(["jquery"],function($){
  return {
    dom:function(){
      $("#div1").html("123");
      alert("shabi")
    },
    abc:"8888888"
  };
})

//main.js

require.config({
  paths:{
    jquery:"jquery-1.10.2.min",
    a:"a",
    b:"b"
  }
});

require(["jquery","a","b"],function($,a,b){
  alert($().jquery);    //1.10.2　　
  alert(info);          //hello world
  b.dom();              //  改写了页面的html文字，弹出shiba
  alert(b.abc);        //8888888
  console.log(b.abc);   //控制台输出8888888
});

  页面代码为：<div id="div1">aaa</div>
```

上面演示了如何自定义模块包含了值对，函数式（存在依赖的函数式定义），可依据需要自己定义，另外，我们也可以再返回之前做一些其他的事情。

六、其他一些配置属性

baseUrl：用于设置基目录\(如上面的例子可以设置baseUrl:"../"，代码一样不变\)

config\(直接看下面的例子

```
//main.js

require.config({
  baseUrl:"../",
  paths:{
    a:"a",
  },
  config:{
    "a":{
      message:"liucunjie"
    }
  }
});

require(["a"],function(a){
  a.ms()   // 控制台下输出liucunjie
});
```

其中在mian.js配置中，指明config中是哪个模块（上面的例子是a.js模块）

然后在a.js文件代码书写

```
define(["module"],function(module){
  return{
     ms:function(){
       var mess = module.config().message;
       console.log(mess);
     }
  }
})
```

引入module，然后用module.config（）来获取。

七、加载非规范（AMD）的模块

理论上，requirejs加载的模块必须是符合AMD规范的，或者是用define（）函数定义的模块

如今，很多主流的库都符合AMD规范，但是也有很多库并不符合，这时就需要在配置里设置shim属性

例如backbone这个库，没有采用AMD规范编写

```
require.config({
  shim:{
    'underscore':{
      exports:'_'
    },
    'backbone':{
      deps:['underscore','jquery'],
      exports:'Backbone'
    }
  }
});
```

其中deps数组为表明其依赖，exports\(输出的变量名\)则为

这个模块外部调用时的名称。



[下一篇 require 简明](/requirejian-ming.md)

