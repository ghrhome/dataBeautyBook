# AngularJS实例 – 装饰$log

在AngularJS中，我们可以使用Angular内置或者自定义的services，在应用的各个部分之间分享数据和方法。假设你已经定义了一个service，但是在使用了一段时间之后又想要为这个service添加一些功能怎么办？方法之一当然是修改这个service的定义，直接在源码上动刀子。但是现在很多项目都需要通过团队协作来完成，直接修改别人的代码可能需要花费不少功夫，同时还要防范“牵一发而动全身”的风险。更进一步，如果你想给一些Angular内置的service添加功能怎么办，难道要我直接阅读angular.js的源码么？

当然，我们不需要去做以上这些事情，因为我们有更好的解决办法。Angular为我们提供了一种非常简单而优雅的方式来为service添加功能，它就是Angular中的decorator。有了它我们可以非常轻松的实现我们的目的。在一面的代码中，我们将对Angular内置service $log进行装饰，让它在每次被调用时除了自身接受的参数之外，还能在参数前面输出一个当前的时间戳。具体实现如下：

```
angular.module('myapp',[])
             .config(function($provide){
                $provide.decorator('$log',function($delegate){
                    angular.foreach(['log','debug','info','warn','error'],function(o){
                        $delegate[o] = decoratorLogger($delegate[o]);
                    });
                    function decoratorLogger(orignalFn){
                        return function(){
                            var args = Array.prototype.slice(arguments);
                            args.unshift(new Date().toISPString());
                            originalFn.apply(null,args);
                        };
                    }
                });
             });  
```

上面的代码很简单，没有什么难点，关键的地方是在于decorator方法的回调函数的参数$delegate，它是decorator第一个参数代表的service的引用。现在我们就可以来验证一下刚才的进行装饰的$log service,在控制台中运行下面代码：

```
angular.element(document).injector().get('$log').log('hello')   

```

结果输出：

> 2014-03-05T15:43:29.974Z hello

成功！

