# AngularJS Module类的方法

AngularJS中的Module类负责定义应用如何启动，它还可以通过声明的方式定义应用中的各个片段。我们来看看它是如何实现这些功能的。

一.Main方法在哪里

    如果你是从Java或者Python编程语言转过来的，那么你可能很想知道AngularJS里面的main方法在哪里？这个把所有东西启动起来，并且第一个被执行的方法在哪里？JavaScript代码里面负责实例化并且把所有东西组合到一起，然后命令应用开始运行的那个方法在哪里？



    事实上，AngularJS并没有main方法，AngularJS使用模块的概念来代替main方法。模块允许我们通过声明的方式来描述应用中的依赖关系，以及如何进行组装和启动。使用这种方式的原因如下：



    1.模块是声明式的。这就意味着它编写起来更加容易，同时理解起来也很容易，阅读它就像读普通的英文一样！



    2.它是模块化的。这就迫使你去思考如何定义你的组件和依赖关系，让它们变得更加清晰。



    3.它让测试更加容易。在单元测试吕，你可以有选择地加入模块，并且可以避免代码中存在无法进行单元测试的内容。同时，在场景测试中，你可以加载其他额外的模块，这样就可以更好地和其他组件配合使用。



    例如，在我们的应用中有一个叫做"MyAwesomeApp"的模块。在HTML里面，只要把以下内容添加到&lt;html&gt;标签中（或者从技术上说，可以添加到任何标签中）：

```
<html ng-app="MyAwesomeApp">
```

    ng-app指令就会告诉AngularJS使用MyAwesomeApp模块来启动你的应用。那么，应该如何定义模块呢？举例来说，我们建议你为服务、指令和过滤器分别定义不同的模块。然后你的主模块可以声明依赖这些模块。



    这样可以使得模块管理更加容易，因为它们都是良好的、完备的代码块，每个模块有且只有一种职能。同时，单元测试可以只加载它们所关注的模块，这样就可以减少初始化的次数，单元测试也会变得更精致、更专注。

二.加载和依赖

    模块加载动作发生在两个不同的阶段，这一点从函数名上面就可以$$x = y$$反映出来，它们分别是Config代码块和Run代码块（或者叫做阶段）。

1.Config代码块

    在这一阶段里面，AngularJS会连接并注册好所有数据源。因此，只有数据源和常量可以注入到Config代码块中。那些不确定是否已经初始化好的服务不能注入进来。

2.Run代码块

    Run代码块用来启动你的应用，并且在注射器创建完成之后开始执行。为了避免在这一点开始之后再对系统进行配置操作，只有实例和常量可以被注入到Run代码块中。你会发现，在AngularJS中，Run代码块是与main方法最类似的东西。

三.快捷方法

    利用模块可以做什么呢？我们可以用它来实例化控制器、指令、过滤器以及服务，但是利用模块类还可以做更多事情。如下模块配置的API方法：

1.config\(configFn\)

    利用此方法可以做一些注册工作，这些工作需要在模块加载时完成。

2.constant\(name, object\)

    此方法会首先运行，所以你可以用它来声明整个应用范围内的常量，并且让它们在所有配置（config方法）和实例（后面的所有方法，例如controller、service等）方法中可用。

3.controller\(name,constructor\)

    它的基本作用是配置好控制器方便后面使用。

4.directive\(name,directiveFactory\)

    可以使用此方法在应用中创建指令。

5.filter\(name,filterFactory\)

    允许你创建命名的AngularJS过滤器，就像前面章节所讨论的那样。

6.run\(initializationFn\)

    如果你想要在注射器启动之后执行某些操作，而这些操作需要在页面对用户可用之前执行，就可以使用此方法。

7.value\(name,object\)

    允许在整个应用中注射值。

8.factory\(name,factoryFn\)

    如果你有一个类或者对象，需要首先为它提供一些逻辑或者参数，然后才能对它初始化，那么你就可以使用这里的factory接口。factory是一个函数，它负责创建一些特定的值（或者对象）。我们来看一个greeter\\(打招呼\\)函数的实例，这个函数需要一条问候语来初始化：

```
function Greeter(salutation) {
 this.greet = function(name) {
 return salutation + ' ' + name;
};
}
```

```
    greeter函数示例如下：
```

```
myApp.factory('greeter', function(salut) {
 return new Greeter(salut);
});
```

```
    然后可以这样来调用它：
```

```
var myGreeter = greeter('Halo');
```

9.service\(name,object\)

```
    factory和service之间的不同点在于，factory会直接调用传递给它的函数，然后返回执行的结果；而service将会使用"new"关键字来调用传递给它的构造方法，然后再返回结果。所以，前面的greeter Factory可以替换成下面这个greeter Service：
```

```
myApp.service('greeter', Greeter);
```

```
    每当我们需要一个greeter实例的时候，AngularJS就会调用新的Greeter\(\)来返回一个实例。
```

10.provider\(name,providerFn\)

```
    provider是这几个方法中最复杂的部分（显然，也是可配置性最好的部分）。provider中既绑定了factory也绑定了service，并且在注入系统准备完毕之前，还可以享受到配置provider函数的好处（也就是config块）。

    我们来看看使用provider改造之后的greeter Service是什么样子：
```

```
myApp.provider('greeter', function() {
 var salutation = 'Hello';
 this.setSalutation = function(s) {
 salutation = s;
}

 function Greeter(a) {
 this.greet = function() {
 return salutation + ' ' + a;
}
}

 this.$get = function(a) {
 return new Greeter(a);
};
});
```

```
    这样我们就可以在运行时动态设置问候语了（例如，可以根据用户使用的不同语言进行设置）。
```

```
var myApp = angular.module(myApp, []).config(function(greeterProvider) {
greeterProvider.setSalutation('Namaste');
});
```

```
    每当有人需要一个greeter实例时，AngularJS就会在内部调用$get方法。
```

附：angular.module\('MyApp',\[...\]\)和angular.module\('MyApp'\)之间有一个很小但是却很重要的不同点

```
    angular.module\('MyApp',\[...\]\)会创建一个新的Angular模块，然后把方括号（\[...\]）中的依赖列表加载进来；而angular.module\('MyApp'\)会使用由第一个调用定义的现有的模块。

    所以，对于以下代码，你需要保证在整个应用中只会使用一次：
```

```
angular.module('MyApp', [...]) //如果你的应用是模块化的，这里可能是MyModule
```

```
    如果你不打算把模块的引用存到一个变量中，然后在整个应用中通过这个变量来引用模块，那么，在其他文件中使用angular.module\(MyApp\)的方式可以保证得到正确的AngularJS模块引用。模块上的所有东西都必须通过访问这个模块引用来定义，或者在模块定义的地方把那些必备的内容添加上去。
```



