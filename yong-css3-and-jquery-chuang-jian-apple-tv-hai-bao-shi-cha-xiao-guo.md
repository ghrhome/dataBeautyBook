# 用CSS3 & jQuery创建apple TV海报视差效果

你见过新版的**Apple TV视差效**果么?非常棒，我们用CSS和jQuery来实现它，尽量看起来和原效果一样。

本教程里，我将使用CSS,HTML和jQuery来创建一个近似Apple TV视差效果，如果你正在阅读，我假设你对上述三种技术都有基本的了解。

废话不多说，开始第一部分。

## HTML页面

我们的页面结构像下面这样：

```
<div class="poster">
  <div class="shine"></div>
  <div class="layer-1"></div>
  <div class="layer-2"></div>
  <div class="layer-3"></div>
  <div class="layer-4"></div>
  <div class="layer-5"></div>
</div>
```

首先，需要一个样式类为`.poster`的`div`，在这个`div`里包含5个其他样式的层`div`。在这五个层`div`上有一个`shine`div来添加一些闪光效果。

## CSS部分

首先，添加以下代码确保网页`body`部分的高度是整个页面高度：

```
body, html { height: 100%; min-height: 100%; }
```

再给`body`部分一些背景渐变颜色：

```
body { background: linear-gradient(to bottom, #f6f7fc 0%,#d5e1e8 40%); }
```

为了让`.poster`有3D旋转的效果，父容器需要设置透视和变换效果。如我们所见，`div`的父容器就是`body`本身，所以添加以下CSS代码：

```
body {
    background: linear-gradient(to bottom, #f6f7fc 0%,#d5e1e8 40%);
    transform-style: preserve-3d;
    transform: perspective(800px);
}
```

现在给卡片设置样式跟大小，让它在页面居中，添加一些圆角跟阴影效果：

```
.poster {
    width: 320px;
    height: 500px;
    position: absolute;
    top: 50%; left: 50%;
    margin: -250px 0 0 -160px;
    border-radius: 5px;
    box-shadow: 0 45px 100px rgba(0, 0, 0, 0.4);
    overflow:hidden;
}
```

为了让海报居中，需要设置`position`的值为`absolute`，`top:50%`, 'left:50%', 上部的`margin`值是`div`高度的一半的负数，左边的`margin`值是`div`宽度的一半的负数。需要记住的是`.poster`的中心也是整个页面的中心。

![](/assets/1621354-20ac0fc235101fd7.jpg)

阴影效果

我们可以用以下的CSS选择器来选择所有的层：

```
div[class *= 'layer-']
```

`.poster` 已经设计好了，来看看效果。

所以，CSS选择了所有class类名里含有“layer-”的 `div`。

现在，设置所有的层的`position`值是`absolute`,`background-repeat`值为`no-repeat`,`background-position`为`top left`

, 层背景的大小为100%宽度和自动高度。

```
div[class*="layer-"] {
    position: absolute;
    top: -10px; left: -10px;
    right: -10px; bottom: -10px;
    background-size: 100% auto;
    background-repeat: no-repeat;
    background-position: 0 0;
    transition:0.1s;
}
```

注意到`top`,`left`,`right`,`bottom`的值都是-10px,目的是让层的大小比`poster`的大20px，这样在各个层进行视察效果的时候就不会看到层的边缘部分了。

以下是给每个层添加背景：

```
.layer-1 {
    background-image: url('http://designmodo.com/demo/apple-tv-parallax/images/1.png');
}

.layer-2 {
    background-image: url('http://designmodo.com/demo/apple-tv-parallax/images/2.png');
}

.layer-3 {
    top: 0; bottom: 0;
    left: 0; right: 0;
    background-image: url('http://designmodo.com/demo/apple-tv-parallax/images/3.png');
}

.layer-4 {
    background-image: url('http://designmodo.com/demo/apple-tv-parallax/images/4.png');
}

.layer-5 {
    background-image: url('http://designmodo.com/demo/apple-tv-parallax/images/5.png');
}
```

在`layer-3`部分， 层不会移动，所以尺寸就不用太大了。

![](/assets/1621354-baaed885ccda9cd7.jpg)

## JavaScript部分

在开始之前，请确保已经引入了jQuery库，否则会报错的。

视差效果的逻辑是这样的，每当鼠标移动的时候，根据光标的位置，`.poster`的`transforms：translateY`,`rotate`,`rotateY`

属性将会改变。光标距离页面左上角越远，动画的效果越明显。

公式就类似于这样的：offsetX=0.5-光标距离页面顶端的位置/宽度。

为了每个元素的值都不一样，将给每一个光标公式返回的值乘以一个自定义的值，返回HTML的代码给每个会有动画的层元素添加

`data-offset=数字`的属性。

```
<div data-offset="15" class="poster">
    <div class="shine"></div>
    <div data-offset="-2" class="layer-1"></div>
    <div class="layer-2"></div>
    <div data-offset="1" class="layer-3"></div>
    <div data-offset="3" class="layer-4"></div>
    <div data-offset="10" class="layer-5"></div>
</div>
```

每一个`.layers`的规则都相同，但是我们给他们应用到`translateY`和`translateX`属性上。

`data-offset`属性的值越大，动画的效果越明显，可以改变这些值体验下。

为了代码可读性，我们在JavaScript里给`.poster`赋值给`$poster`变量，`.shine`给`$shine`变量，`$layer`变量代表所有层，`w`,`h`代表页面的宽度和高度。

```
var $poster = $('.poster'),
$shine = $('.shine'),
$layer = $('div[class*="layer-"]’);
```

现在，需要考虑下当光标移动的时候获取到光标位置的问题。我们可以用`$(window)`的`mousemove`事件来实现，这个事件会返回一个JavaScript对象，含有我们需要的位置信息和其他一些我们暂时还用不到的变量。

```
$(window).on('mousemove', function(e) {
    var w=e.currentTarget.innerWidth,h=e.currentTarget.innerHeight;
    var offsetX = 0.5 - e.pageX / w, /* where e.pageX is our cursor X coordinate */
    offsetY = 0.5 - e.pageY / h,
    offsetPoster = $poster.data('offset'), /* custom value for animation depth */
    transformPoster = 'translateY(' + -offsetX * offsetPoster + 'px) rotateX(' + (-offsetY * offsetPoster) + 'deg) rotateY(' + (offsetX * (offsetPoster * 2)) + 'deg)';

    /* apply transform to $poster */
    $poster.css('transform', transformPoster);

    /* parallax foreach layer */
    /* loop thought each layer */
    /* get custom parallax value */
    /* apply transform */
    $layer.each(function() {
        var $this = $(this);
        var offsetLayer = $this.data('offset') || 0; /* get custom parallax value, if element docent have data-offset, then its 0 */
        var transformLayer = 'translateX(' + offsetX * offsetLayer + 'px) translateY(' + offsetY * offsetLayer + 'px)';

        $this.css('transform', transformLayer);
    });
});

```

下一步，就是用上面解释的公式来计算`offsetY`和`offsetX`的值，然后就是把视差效果应用到`.posert`和每一个海报层。

非常酷啊，现在我们就有了一个有视差效果的小部件了。

**但是还没完,海报上的光泽部分还没设置**

现在回到CSS部分，给`.shine`div 绝对定位，添加一个渐变颜色效果，设置`z-index`属性值为100，让它在所有层的上面。

```
.shine {
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    background: linear-gradient(90deg, rgba(255,255,255,.5) 0%,rgba(255,255,255,0) 60%);
    z-index: 100;
}
```

已经有了一个漂亮的闪光层在海报上，但是为了达到更逼真的效果，光照应该随着光标的移动而变化。

