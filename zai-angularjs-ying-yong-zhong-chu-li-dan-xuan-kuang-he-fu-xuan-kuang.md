AngularJS对表单的处理相当简单。在AngularJS使用双向数据绑定方式进行[表单验证](http://scotch.io/tutorials/javascript/angularjs-form-validation)的时候，实质上它在帮我们进行表单处理。

以前，我曾经写过AngularJS在表单处理方面的强大功能以及如何进行[表单处理](http://scotch.io/tutorials/javascript/submitting-ajax-forms-the-angularjs-way)的文章。今天，我们将快速浏览一下AngularJS是如何对表单中的复选框和单选按钮进行处理的。

使用复选框的的例子有很多，同时我们对它们的处理方式也有很多。这篇文章中我们将看一看把复选框和单选按钮同数据变量绑定的方法和我们对它的处理办法。

# 创建Angular表单 {#content_h1_2_5}

在这篇文章里，我们需要两个文件:index.html和app.js。app.js用来保存所有的Angular代码（它不大），而index.html是动作运行的地方。首先我们创建AngularJS文件。

```
// app.js

var formApp = angular.module('formApp', [])

    .controller('formController', function($scope) {

        // we will store our form data in this object
        $scope.formData = {};

    });
```

在这个文件里，我们所做的就是创建Angular应用。其中我们还创建了一个控制器和一个用来保存所有表单数据的对象。

下面我们看看index.html文件，在这个文件里，我们创建了表单，然后进行了数据绑定。我们使用了[Bootstrap](http://getbootstrap.com/)快速地对页面进行布局。

```
<-- index.html -->
<!DOCTYPE html>
<html>
<head>

    <!-- CSS -->
    <!-- load up bootstrap and add some spacing -->
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css">
    <style>
        body         { padding-top:50px; }
        form            { margin-bottom:50px; }
    </style>

    <!-- JS -->
    <!-- load up angular and our custom script -->
    <script src="http://code.angularjs.org/1.2.13/angular.js"></script>
    <script src="app.js"></script>
</head>

<!-- apply our angular app and controller -->
<body ng-app="formApp" ng-controller="formController">
<div class="col-xs-12 col-sm-10 col-sm-offset-1">

    <h2>Angular Checkboxes and Radio Buttons</h2>

    <form>

        <!-- NAME INPUT -->
        <div class="form-group">
            <label>Name</label>
            <input type="text" class="form-control" name="name" ng-model="formData.name">
        </div>

        <!-- =============================================== -->
        <!-- ALL OUR CHECKBOXES AND RADIO BOXES WILL GO HERE -->
        <!-- =============================================== -->

        <!-- SUBMIT BUTTON (DOESNT DO ANYTHING) -->
        <button type="submit" class="btn btn-danger btn-lg">Send Away!</button>

    </form>

    <!-- SHOW OFF OUR FORMDATA OBJECT -->
    <h2>Sample Form Object</h2>
    <pre>
        {{ formData }}
    </pre>

</div>
</body>
</html>
```

创建完成之后，我们就有了具有name输入的表单了。如果一切都按照我们设想的运行，那么如果你在name输入中键入内容，那么你应当可在下面的&lt;pre&gt;标签段看到所输入的内容了.

# 复选框 {#content_h1_3_5}

在表单里，复选框非常普遍。下面我们将看看Angular是怎样使用**ngModel**实现数据绑定的。如果有许多复选框，那么有时在把它绑定到对象的时候如何进行数据处理会让人不知所措。

在我们创建的formData对象的内部，我们还创建了另一个对象。我们把它称为favoriteColors，它请求用户选择最喜欢的颜色：

```
<!-- MULTIPLE CHECKBOXES -->
<label>Favorite Colors</label>
<div class="form-group">
    <label class="checkbox-inline">
        <input type="checkbox" name="favoriteColors" ng-model="formData.favoriteColors.red"> Red
    </label>
    <label class="checkbox-inline">
        <input type="checkbox" name="favoriteColors" ng-model="formData.favoriteColors.blue"> Blue
    </label>
    <label class="checkbox-inline">
        <input type="checkbox" name="favoriteColors" ng-model="formData.favoriteColors.green"> Green
    </label>
</div>
```

当用户点击上面复选框中的任意一个时，他们立刻看到formData对象发生了变更。我们把复选框的值存储到fromData.favoriteColors对象里。这样我们就把复选框的值传递给了服务器了。

## 复选框点击处理 {#content_h2_4_5}

有时候，当某人点击了复选框后，你需要对其进行处理。你需要做的处理可能如下：计算某个值，更改某些变量或者进行数据绑定。要实现这些，你要使用`$scope.yourFunction = function() {};`在`app.js`内创建函数。接着你就可以在的的复选框上使用`ng-click="yourFunction()"`来调用这个函数了。

处理表单复选框的方法有许多种，Angular提供了一个非常简单的方法：使用`ng-click`调用用户自定义的函数。

## 自定义复选框对应的值 {#content_h2_5_5}

默认情况下，绑定到复选框上的值是ture或者false。有时候，我们希望返回的其它值。Angular提供了一种非常好的处理方式：使用ng-ture-value和ng-false-value。

我们添加另外一组复选框，不过这时侯我们使用的不再是true或者false，而是用户自定义的值。

```
<!-- CUSTOM VALUE CHECKBOXES -->
<label>Personal Question</label>
<div class="checkbox">
    <label>
        <input type="checkbox" name="awesome" ng-model="formData.awesome" ng-true-value=" 'ofCourse' " ng-false-value="'iWish'">
        Are you awesome?
    </label>
</div>
```

> 注意ng-true/false-value值绑定 的" '' "



另外，现在我们还在formData对象里增加了一个awesome变量。如果此时设置这个值为true，那么返回的值应该是ofCourse，如果设置为false，那么返回的值为iWish。

### 复选框 {#content_h3_6_5}

依据[官方](http://docs.angularjs.org/api/ng/input/input[checkbox])说明文档, 这是和单选框不同之处:

```
<input type="radio"
   ng-model="string"
   value="string"
   [name="string"]
   [ng-change="string"]
   ng-value="string">
```

需要了解更多有关复选框的信息，请关注 [Angular 复选框](http://docs.angularjs.org/api/ng/input/input[checkbox])说明文档.

## 单选按钮 {#content_h2_6_12}

单选按钮比复选框容易些，就在于无需存储多选项数据. 单选就是一个值. 下面添加一个单选按钮看看.

```
<!-- RADIO BUTTONS -->
<label>Chicken or the Egg?</label>
<div class="form-group">
    <div class="radio">
        <label>
            <input type="radio" name="chickenEgg" value="chicken" ng-model="formData.chickenEgg">
            Chicken
        </label>
    </div>
    <div class="radio">
        <label>
            <input type="radio" name="chickenEgg" value="egg" ng-model="formData.chickenEgg">
            Egg
        </label>
    </div>
</div>
```

就像这样，单选按钮就绑定到数据对象了.

### 单选按钮用法 {#content_h3_7_5}

据[官方文档](http://docs.angularjs.org/api/ng/input/input[radio]), 这是提供的选项:

```
<input type="radio"
       ng-model="string"
       value="string"
       [name="string"]
       [ng-change="string"]
       ng-value="string">
```

更多内容，请阅读[Angular 单选](http://docs.angularjs.org/api/ng/input/input[radio])按钮文档.

## 解磷 {#content_h2_7_12}

正如你所见，使用Angular绑定复选框和单选按钮都是十分简单的. 在创建更具个性的复选框或是单选按钮时，它也有很大的灵活性.

源码 :[http://plnkr.co/edit/g0NMG4rmF4uwzoG2uZhf?p=preview](http://plnkr.co/edit/g0NMG4rmF4uwzoG2uZhf?p=preview)

