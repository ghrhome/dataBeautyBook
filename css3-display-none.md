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

PS:另外一种 animated的封装

```
/**
 * Shortcut for adding animation class name to dom element
 * @param  {string}   cls        class name
 * @param  {Function} cb         callback
 * @param  {Function}   lastItemCb  callback execute at last elment animated 
 * @return {object}              this
 */
$.fn.classAnimoEnd = function(cls, cb, lastItemCb) {
  var el = this;

  cb && (cb = $.proxy(cb, el));

  var count = $(el).length;

  $(el)
    .removeClass('animated ' + cls)
    .one($.support.animate.end, function () {
        cb && cb();
        $(el).removeClass('animated');

        if(count > 1) {
          count --;

          if(count === 0)
            lastItemCb && lastItemCb();
        }
      })
    .addClass('animated ' + cls);

  if(!$(el).css('display') !== 'none' )
    $(el).show();

  return this;
}
```

其他思路：

```
隐藏一个元素的方法有很多种

display: none
visibility: hidden
opacity: 0
max-height: 0, overflow: hidden

这些方法中，有的是隐藏后不占据空间，有的要占据，有的是离散状态（没有transition）,有的可以有transition。
```



