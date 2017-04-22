### 如何将_angularJs_项目与_requireJs_集成

**一、简单事例的项目目录如下：**

-index.html

-scripts文件夹

--controller文件夹

```
  --- mianController.js

  --- controller1.js

  ---controller2.js
```

--directives文件夹

```
  ---mainDirective.js

  ---directive.js
```

--app.js

--router.js

--main.js

**二、首页**

首先你的index.html大概如下

```
<!doctype html>
<!-- <html xmlns:ng="//angularjs.org" id="ng-app" ng-app="webApp"> -->
<htmls>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge, chrome=1">
</head>

<body>
<!--其他html内容-->
<script type='text/javascript' src='../scripts/lib/require.js' data-main='../scripts/main.js'></script>
</body>
</html>
```

在首页index.html只需要引入requireJs库文件，并且指明入口函数main.js，采用手动启动angularjs应用，所以这里不用为首页添加ng-app='webApp'。

**三、配置main.js**

```
require.config({
    paths:{
        //一些库文件
        'jquery': '//cdn.staticfile.org/jquery/1.10.2/jquery.min',
        'angular': '//cdn.staticfile.org/angular.js/1.2.10/angular.min',
        'angular-route': '//cdn.staticfile.org/angular-ui-router/0.2.8/angular-ui-router.min','domReady': '//cdn.staticfile.org/require-domReady/2.0.1/domReady.min',
        //js文件
        'bootstrap': "../scripts/bootstrap",
        'app': "../scripts/app",
        'router': "../scripts/router"
        .....以及其他的js文件，这里省略
    },
    shim:{
        'angular':{
            exports:'angular'
        },
        'angular-route':{
            deps:['angular'],
            exports: 'angular-route'
        }
    },
    deps:['bootstrap'],
    urlArgs: "bust=" + (new Date()).getTime()  //防止读取缓存，调试用
});
```

关于require.config\(\)中的配置大概如下（依照自己的项目情况，这里我简单配置了），

所以总体上说main.js这个文件做的事情就是：由requirejs异步载入所有文件

注意到其中的deps：\['bootstrap'\]，就是告诉我们要先加载bootstrap.js文件。

**四、手动启动angularjs应用**

而bootstrap.js就是我用来手动启动angularjs应用的，代码如下

```
define(['require',
        'angular',
        'angular-route',
        'jquery',
        'app', 
        'router'
       ],function(require,angular){
            'use strict';
            require(['domReady!'],function(document){
                angular.bootstrap(document,['webapp']);
            });
        });
```

这里依赖于app.js和router.js，我们看看这两个文件分别做什么

**五、网站路由设置**

我们这里使用ui-router。所以我们的router.js应该是这样的，主要是用来定义路由的，关于ui-router的配置请自行查看相关知识，这里就简单略过

```
define(["app"],
       function(app){
            return app.run([
                            '$rootScope',
                            '$state',
                            '$stateParams',
                            function ($rootScope, $state, $stateParams) {
                                $rootScope.$state = $state;
                                $rootScope.$stateParams = $stateParams
                            }
                         ])
                     .config(function($stateProvider, $urlRouterProvider, $locationProvider, $uiViewScrollProvider){
                        //用于改变state时跳至顶部
                        $uiViewScrollProvider.useAnchorScroll();
                        // 默认进入先重定向
                        $urlRouterProvider.otherwise('/home');
                        $stateProvider
                        .state('home',{
                            //abstract: true,
                            url:'/home',
                            views: {
                                'main@': {
                                    templateUrl: '../view/home.html'
                                }
                            }
                        })                       
})
```

**六、angular.module**

这时先看看平时我们在写angularjs应用是这样写的

```
var app = angular.module("xxx",["xxx"]);

app.controller("foo",function($scope){});
app.directive(...)
```

所以我们的app.js应该是这样的

```
define(["angular",
        'mainController',
        'mainDirective'
       ],function(angular){
    return angular.module("webapp",['ui.router','ngStorage','ngSanitize','webapp.controllers','webapp.directive']);
})
```

其中我们的app.js中依赖的mainController和mainDirective主要是用来加载angular的控制器和指令

**七、控制器**

我们的mainController.js是这样的

```
define(['controller1','controller2'],function(){});
```

主要用来加载各个控制器（所有的控制器都将在这个文件中被加载）,除此之外再不用做其他，因为我们可以有很多个控制器文件，按照具体需要进行添加。

controller1.js文件\(其他控制器controller2文件类似\)

```
define(['../scripts/controller/module.js','jquery'],function(controllers,$){
    'use strict';
    controllers.controller('controller1',function($scope){
        //控制器的具体js代码
    })
})
```

其中依赖module.js

而module.js如下

```
define(['angular'], function (angular) {
    'use strict';
    return angular.module('webapp.controllers', []);
 });
```

**八、指令**

同理mianDirective.js也类似。参考控制器

**九、其他**

