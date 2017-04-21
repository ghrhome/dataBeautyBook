# requireJS加载深入

define定义require加载使用，这个理解是对的。

```
作者：知乎用户
链接：https://www.zhihu.com/question/21260764/answer/20758189
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

define([require,path/myMod1,path/moMod2], function(require){  
    var mod1 = require('path/myMod1'), mod2 = require('path/myMod2');
})

其实逻辑上类似于

define([require], function(require){  
    var mod1 = require('path/myMod1'), mod2 = require('path/myMod2');
})

你只是把参数丢了而已，所以这样也行

define([require,path/myMod1,path/moMod2], function(require){  
    var mod1 = arguments[1], mod2 = arguments[2];
})

所以，可以这么用

define([require,path/myMod1,path/moMod2], function(require,mod1,mod2){  
    // 这里 mod1 和 mod2 都准备好了，还可以用require继续加载别的
})

或者，改成你看到的那些例子

define([path/myMod1,path/moMod2], function(mod1,mod2){  
})

所以

define(['jquery.validate'], function(validate){
    // 这里可以用validate，但是用不了jquery了？
})

这样本质上并没有问题，只不过你引用了这个插件，必然也是想用jquery的而不只是validate，

所以

define(['jquery', 'jquery.validate'], function(jquery, validate){
    // 这样才有jquery
})

像你一开始那样使用也行

define(['require', 'jquery.validate'], function(require, validate){
    var jquery = require('jquery');
})
```

原文问题：

# requirejs中define和require的定位以及使用区别？

“之前理解的是”，define用来定义模块\(注册为requirejs中模块\)，require是用来加载和使用模块的。但在使用的过程中和看过API后，有几个问题困扰的。

**引：**

define\(\[\], function\(\){}\)中第一个参数是依赖的名称数组，第二个参数是函数，在模块的所有依赖加载完毕后，该函数会被调用来定义该模块。依赖关系会以参数的形式注入到该函数上，参数列表与依赖名称列表一一对应。这是官方的解释。

**问：**

但为什么我自己写的模块，需要在  


```
define([require,path/myMod1,path/moMod2], function(require){  
        var mod1 = require('path/myMod1'), mod2 = require('path/myMod2');
     })
```

  


使用require之后才能使用？ 而像jQuery 以及官方给出的一些例子 只需要将依赖模块作为参数传进回调函数即可使用，不需要require呢？如

  


```
// jquery 例子
    define([jquery], function($){  
           // $ do something
    })
    // 官方API例子
    define(["./cart", "./inventory"], function(cart, inventory) {
        //返回一个定义了该"my/shirt"模块的object
        return {
            color: "blue",
            size: "large",
            addToCart: function() {
            inventory.decrement(this);
            cart.add(this);
            }
        }
        }
    );
```

有人告诉我是因为jquery挂在了

window上~但最后试了，还是不行。

**问：**

关于requireJs.config 中shim， 下面代码中已经配置好了validate依赖关系，但是在使用时还需要defind\(\['jquery', 'jquery.validate'\]\)将jquery引进来。如果按照我之前理解是已经配置了依赖关系了  


，为什么还要在使用jquery.validate时 引入jquery? 是否我之前的理解关于define这有问题？

```
shim : {
		'jquery.validate' : {
		    deps : ['jquery'],
		    exports : 'jQuery.fn.validate'
		},
	}

```

总结：因此造成我困扰的 就是define 和 require，我该怎么去理解？ 希望有熟习的朋友解答下。

