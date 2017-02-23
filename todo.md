有三种方法：

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



