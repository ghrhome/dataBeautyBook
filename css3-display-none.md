在写移动开发时经常会遇到弹出层动画。默认它是隐藏的none的。但是在这个基础上加动画就麻烦了。

可以把动画写在一个类内，另一个是none类。默认两个都引用

显示时先把none去掉，再去掉动画的那个类，动画类可以在requestAnimationFrame函数内执行

隐藏时先添加 上动画，在transitionend回调里添加上none类

```
<style>
.box{
   background: goldenrod;
   width: 200px;
   height: 200px;
   transition: all .4s linear;
   visibility: visible;
   position:absolute;right:0px;top:0px;
}
.hidden{
   display: none;
}
.visuallyhidden{
   opacity: 0;
}
</style>
<div class="box hidden visuallyhidden"></div>
<script>
var box = $('.box');
$('#but').on('click', function () {
   if (box.hasClass('hidden')) {
       box.removeClass('hidden');
       requestAnimationFrame(function(){
           box.removeClass('visuallyhidden');
       });
   } else {
       box.addClass('visuallyhidden');
       box.one('transitionend', function(e) {
           box.addClass('hidden');
       });
   }
});
</script>
```



