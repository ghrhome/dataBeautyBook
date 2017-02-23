[AngularJS中ng-class&ng-style使用方法.md](AngularJS中ng-class&ng-style使用方法.md)有三种方法：

1、通过$scope绑定（不推荐）

2、通过对象数组绑定

3、通过key/value键值对绑定

实现方法：

1、通过$scope绑定（不推荐）：

```
function ctrl($scope) {   
    $scope.className = "selected";  
}  
```

```
<div class="{{className}}"></div>  
```

2、通过对象数组绑定：

```
function ctrl($scope) {   
    $scope.isSelected = true;  
} 
```

```
<div ng-class="{true:'selected',false:'unselected'}[isSelected]"></div>  
```

当isSelected为true时，增加selected样式；当isSelected为false时，增加unselected样式。

3、通过key/value键值对绑定：

```
function ctrl($scope) {   
    $scope.isA = true;  
    $scope.isB = false;  
    $scope.isC = false;  
}  
```

```
<div ng-class="{'A':isA,'B':isB,'C':isC}"></div> 
```

当isA为true时，增加A样式；当isB为true时，增加B样式；当isC为true时，增加C样式。

```
<ion-list>  
    <ion-item ng-repeat="project in projects" ng-click="selectProject(project, $index)" ng-class="{active: activeProject == project}">  
        {{project.title}}  
    </ion-item>  
</ion-list>  
```

根据projects循环创建ion-item，当activeProject为当前循环到的project时，增加active样式。

几点说明：

1、不推荐第一种方法，因为controller $scope应该只有数据和行为

2、ng-class是增加相关样式，可以和class同时使用



# AngularJS`ng-style`指令

使用AngularJS添加样式，使用 CSS key=&gt;value 对象格式:

## 定义和用法

**ng-style**指令为 HTML 元素添加 style 属性。

**ng-style**属性值必须是对象，表达式返回的也是对象。

对象由 CSS 属性和值注册，即 key=&gt;value 对。

> &lt;_element _ng-style="_expression_"&gt;&lt;/_element_&gt;

```
<body ng-app="myApp" ng-controller="myCtrl">

<h1 ng-style="myObj">菜鸟教程</h1>

<script>
var app = angular.module("myApp", []);
app.controller("myCtrl", function($scope) {
    $scope.myObj = {
        "color" : "white",
        "background-color" : "coral",
        "font-size" : "60px",
        "padding" : "50px"
    }
});
</script>
</body>
```



