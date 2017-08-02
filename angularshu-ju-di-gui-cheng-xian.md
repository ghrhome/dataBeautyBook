# Angular数据递归呈现

# AngularJs数据递归呈现的实现的几种方式



在实践中，常常会有数据结构一致但深度不确定的数据。而通常情况下，对数据的处理会用到递归的方式。例如网站的无限级分类的展现，分类数据结构：

  
var cate = \[  
    {  
        cateId: 1,  
        cateName: '前端技术',  
        child: \[  
            {  
                cateId: 4,  
                cateName: 'JavaScript'  
            },  
            ...  
        \]  
    },  
    {  
        cate: 2,  
        cateName: '后端技术',  
        child:\[  
            {  
                cateId: 3,  
                cateName: 'PHP',  
                child: \[  
                    {  
                        cateId: 6,  
                        cateName: 'ThinkPHP'  
                    },  
                    ...  
                \]  
            }  
        \]  
    }  
\];  
用jq常常通过如下方式实现，示例代码如下：

  
/\*\*  
 \* \* 解析模板文件  
 \* @param template  模板字符串  
 \* @param scope     模板作用域  
 \* return \[string\]  解析过后的字符串  
 \*/  
function templateParse\(template, scope\) {  
    if\(typeof template != "string"\) return ;  
    return template.replace\(/\{\w+\}/gi, function\(matchs\) {  
        var returns = scope\[matchs.replace\(/\(\{\)\|\(\}\)/g, ""\)\];  
        return \(returns + ""\) == "undefined"? "": returns;  
    }\);  
}  
  
/\*\*  
 \* 解析并插入分类  
 \*/  
$\('\#category'\).append\(function\(\){  
    var template = '&lt;a href="{cateId}.html"&gt;{cateName}&lt;/a&gt;';  
    var ret = \(function\(c\){  
        if\(!c \|\| !c.length\) return '';  
        var ret = '&lt;ul&gt;';  
        for\(var i = 0, j = c.length; i &lt; j; i++\){  
            ret += '&lt;li&gt;';  
            ret += templateParse\(template, c\[i\]\);  
            ret += arguments.callee\(c\[i\].child\);  
            ret += '&lt;/li&gt;';  
        }  
        return \(ret + '&lt;/ul&gt;'\);  
    }\)\(cate\);  
  
    return ret;  
}\);  
以上页面可以点击"jq实现无限级分类展示演示DEMO"，查看对应结果。

angularJs中基于模板递归的实现

同样的原理，可以通过组合angularJs内置的ng-include、ng-init来达到递归的效果，示例模板如下：



```
<script id="recursion" type="text/ng-template">
<li ng-repeat="item in cate">
<a href="{{item.cateId}}">{{item.cateName}}</a>
<ul ng-if="item.child.length" ng-include="'recursion'"  ng-init="cate=item.child"></ul>
</li>
</script>
/*调用方式如下：*/

<div ng-app="demo">
<div ng-controller="demo">
<ul ng-include="'recursion'"></ul>
</div>
</div>

实现效果演示DEMO，"AngularJ基于模板递归实现分类展示"
```

实现效果演示DEMO，"AngularJ基于模板递归实现分类展示"

基于指令递归的实现

同样的道理，在指令中，咱们可以这么来干（内容参考自angular-recursion）：

```
angular.module('demo').directive('recursion',function($compile){

    function cpl(element, link){
        // Normalize the link parameter
        if(angular.isFunction(link)){
            link = { post: link };
        }

        // Break the recursion loop by removing the contents
        var contents = element.contents().remove();
        var compiledContents;
        return {
            pre: (link && link.pre) ? link.pre : null,
            /**
             * Compiles and re-adds the contents
             */
            post: function(scope, element){
                // Compile the contents
                if(!compiledContents){
                    compiledContents = $compile(contents);
                }
                // Re-add the compiled contents to the element
                compiledContents(scope, function(clone){
                    element.append(clone);
                });

                // Call the post-linking function, if any
                if(link && link.post){
                    link.post.apply(null, arguments);
                }
            }
        };
    }

    return {
        restrict:'A',
        scope: {recursion:'='},
        template: '<li ng-repeat="item in recursion">\
<a href="{{item.cateId}}.html">{{item.cateName}}</a>\
<ul recursion="item.child">\
</ul>\
</li>',
        compile: function(element){

            return cpl(element, function(scope, iElement, iAttrs, controller, transcludeFn){
                // Define your normal link function here.
                // Alternative: instead of passing a function,
                // you can also pass an object with
                // a 'pre'- and 'post'-link function.
            });
        }
    };
});


```

关于recursion指令

以上两种方式实现与模板耦合的过于紧密，有没有办法可以像如下的方式来使用呢？\(未完，等续\)

```
<ul recursion="cate">
<li ng-repeat="item in cate">
<a href="{{item.cateId}}">{{item.cateName}}</a>
<ul recursion-child='item.child'></ul>
</li>
</ul>

```



