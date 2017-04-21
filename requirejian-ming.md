# requirejs简明

一、前言

[requireJS\(一\)](http://www.cnblogs.com/cunjieliu/p/3711139.html)

本篇主要整理requirejs的一些用法，相对比较零散。

实例目录

![](http://images.cnitblog.com/i/600316/201405/071103064174423.png)

二、优化

requirejs建议我们给每一个模块书写一个js文件。但是这样会增加网站的http请求，这时可利用[工具](http://requirejs.org/docs/optimization.html)打包，详情求戳链接查看。

三、关于define自定义模块

之前说到自定义模块define（）可接收三个参数，其中第一个参数为模块的名字。即我们可以显式的定义模块的名字

```
1 define('js/a',["b"],function(b){
2     do something...
3 })
```

但是不建议这样做，因为这样的硬编码对代码的可移植性比较差，就是说若你将文件移动到其他目录下，你就得重命名。



四、在一个模块中的define（）内部使用require（）调用其他模块

这样就可以用相对于此文件的位置来调用模块了。下面我们演示在d模块中调用c模块，

假设我们在d.js中编写如下代码

```
//d.js

define(["require"],function(require){
  var mod = require("./c");
  console.log(mod.ccc);   //此时可以在控制台看到输出123456789
  //alert(mod.ccc)
})
```

```
//而在c.js

define(function($){
  return {
    ccc:"123456789"
  };
})
```

这样，你可以访问模块的相邻模块，无需知道该目录的名称。

或者直接用更简洁的代码（commonjs模块语法）ps：没研究过commonjs。

```
define(function(require) {
    var mod = require("./c"); console.log(mod.ccc);
});
//该形式利用了Function.prototype.toString()去查找require()调用，然后将其与"require"一起加入到依赖数组中，这样代码可以正确地解析相对路径了。
```

效果与上面的相同。



五、require.toUrl\(\)生成相对于模块的URL地址

```
//d.js
define(["require"],function(require){
  var url = require.toUrl("./c");
  console.log(url);  // 输出../c
})

```

六、控制台调试

我们可以使用require\("module/name"\).callsomefunction\(\)来调试模块，简单实例请看下面截图

![](http://images.cnitblog.com/i/600316/201405/071004442457190.jpg)

七、循环依赖

所谓的循环依赖，即系a依赖b，b同时依赖a。（一般情况下最好要避免循环依赖）

这时候，如果代码这样的写得话

```
//e.js

define(["f"],function(f){
  return{
    eee:"eeeeee"
  }
})

//f.js

define(["e"],function(e){
  return {
    ale:function(){
      var E  = e.eee;
      console.log(E);
    }
  }
})

//mian.js
代码省略，主要就是注入模块依赖
```

会出现 uncaught TypeError

![](http://images.cnitblog.com/i/600316/201405/071024010261094.png)

这时，我们只要将代码更改为

```
define(["require","e"],function(require,e){
  return {
    ale:function(){
      var E  = require("e").eee;
      console.log(E);
    }
  }
})
```

这样就可以看到在控制台下输出eeeeee啦

