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

































